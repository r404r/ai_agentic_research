# 10 · multica 的 Agentic 实现

> 来源：Phase 1 `multica` 源码调查（只读，`git status --short` 为空）。证据为文件路径 + 行号；汇总见 [证据索引](90-evidence-index.md)。

## 1. 定位

`multica` 是一个“人 + AI coding agent 共用 issue 工作流”的 **Managed Agents / AI-native 任务管理平台**。它本身**不实现模型推理**，而是把 Claude Code、Codex、Copilot、OpenCode、OpenClaw、Gemini、Cursor Agent、Kimi、Kiro 等外部 CLI 包装成可被分配 issue、评论触发、自动化触发的“团队成员”。

证据：

- `README.zh-CN.md:29-35` —— “像分配给同事一样分配给 Agent”，支持多种 coding agent CLI，并有 Squads。
- `docs/product-overview.md:46-88` —— 明确 Multica 是“人 + AI 协作的任务管理平台”，不自己训模型，是调度器；daemon 探测本地 CLI。
- `server/pkg/agent/agent.go:1-4,15-21,135-171` —— `agent.Backend` 是统一 CLI 执行接口，`New()` 按 provider 创建各 backend。

## 2. Agentic 实现模型

核心模型：**服务端持久化工作流 + 本地 daemon 执行 + 外部 AI CLI provider**。

- 服务端：负责 workspace / issue / comment / agent / runtime / task / autopilot / squad 的数据与调度。
- 本地 daemon：注册 runtime、claim task、准备隔离工作目录、注入上下文，启动外部 agent CLI。
- agent：通过 task-scoped token 与 `multica` CLI 回写评论、状态、issue。

关键实体证据：

- `server/migrations/001_init.up.sql:35-49` —— `agent` 含 `runtime_mode`、`runtime_config`、`visibility`、`status`、`max_concurrent_tasks`。
- `server/migrations/004_agent_runtime_loop.up.sql:1-15,17-92` —— 引入 `agent_runtime`，agent / task 绑定 `runtime_id`，并为 runtime claim 建索引。
- `server/migrations/001_init.up.sql:51-72,96-107,126-140` —— `issue`、`comment`、`agent_task_queue` 是 agent 工作流基础。
- `server/migrations/042_autopilot.up.sql:3-24,29-47,49-77` —— `autopilot`、`autopilot_trigger`、`autopilot_run` 与 task / issue origin 关联。

## 3. 任务调度与执行链

典型链路：

1. issue 被分配给 agent / 评论 @agent / quick create / chat / autopilot 创建 task。
2. `TaskService` 写入 `agent_task_queue(status='queued')`，广播 queued 事件，并通过 wakeup 通知 runtime。
3. daemon poller 先拿本地并发槽，再向 server claim runtime 的 task。
4. server 通过 `ClaimTaskForRuntime` 找 runtime 下 queued candidate，按 agent capacity + 同 issue/session 串行规则原子 claim。
5. daemon 准备工作目录、注入 runtime config / brief，调用 provider backend 执行外部 CLI。
6. daemon 回调 `StartTask` / `CompleteTask` / `FailTask`；服务端广播、更新 task，必要时补发 agent comment、同步 chat / autopilot 状态。

证据：

- 入队：`server/internal/service/task.go:430-494`（issue task）、`500-548`（mention / squad leader）、`606-671`（quick create）、`711-740`（chat）。
- wakeup：`server/internal/service/task.go:1929-1962`，入队后 bump empty-claim cache 并 `NotifyTaskAvailable`。
- claim：`server/internal/service/task.go:924-988`（按 agent capacity）、`990-1102`（按 runtime，含 stale dispatched recovery 与 empty-claim fast path）。
- 原子 claim SQL：`server/pkg/db/generated/agent.sql.go:523-562`，`FOR UPDATE SKIP LOCKED`，避免同一 agent 对同 issue/chat/quick-create 并行。
- daemon 主循环：`server/internal/daemon/daemon.go:601-685`。
- daemon poller：`server/internal/daemon/daemon.go:1995-2090`，先拿 `MaxConcurrentTasks` slot 再 `ClaimTask`。
- 执行：`server/internal/daemon/daemon.go:2208-2301`（处理 task）、`2625-2865`（准备环境、注入上下文、设置 `MULTICA_TOKEN`/workspace/agent/task env）。
- 结果回传：`server/internal/daemon/daemon.go:2497-2557`（`CompleteTask`/`FailTask`）；服务端完成处理 `server/internal/service/task.go:1179-1343`，失败处理 `1362-1448`。

## 4. Agent / Runtime / Provider 适配

Runtime 是 **daemon × provider × workspace** 的执行单元。daemon 启动时探测本机 CLI 并向 server 注册 runtime；agent 绑定 runtime。Provider backend 把统一 prompt / options 翻译成不同 CLI 协议。

证据：

- daemon 注册 runtime：`server/internal/daemon/daemon.go:753-801`（逐个探测 provider 版本并上报）。
- server 注册 API：`server/internal/handler/daemon.go:168-184`（register request），`253+`（处理注册）。
- provider 统一接口：`server/pkg/agent/agent.go:15-60`，`Backend.Execute` 与 `ExecOptions`（cwd / model / system prompt / session / custom args / MCP / thinking level）。
- Codex backend：`server/pkg/agent/codex.go:78-85`（`codex app-server --listen stdio://`），`21-29`（屏蔽硬编码 transport 被 custom args 覆盖）。
- Claude backend：`server/pkg/agent/claude.go:17-23,59-67,147-195`（`claude` stream-json 执行并解析事件）。

## 5. Squad（专家路由层）

Squad 是“可分配对象 + leader agent 路由层”。任务分配给 squad 时，不是整队同时执行，而是先触发 **leader agent**；leader 读 issue 后用 mention / child issue 委派给成员。leader task 用 `is_leader_task` 区分角色，避免自触发循环。

证据：

- `server/migrations/090_task_is_leader.up.sql:1-9` —— task 加 `is_leader_task`。
- `server/internal/handler/squad.go:23-38`（response 含 `leader_id`、members）、`196-260`（创建 squad 并 auto-add leader）。
- leader briefing：`server/internal/handler/squad_briefing.go:11-19,21-89`（leader 只协调、不默认执行、用 mention 委派、记录 activity）、`91-117`（claim 时拼 briefing）、`119-218`（构造 roster 与可触发 mention markdown）。
- 分配触发：`server/internal/handler/squad.go:941-980`（非 backlog 且 leader ready 才触发），实际入队 `982-1014`（`EnqueueTaskForSquadLeader`）。
- 评论路由规则：`server/internal/handler/squad_comment_trigger_test.go:116-199`（成员无 @ 评论触发 leader；显式 @ 时不再重复触发 leader；agent 评论仍可触发 leader 协调）。

## 6. Autopilot（自动化）

Autopilot 支持定时 / 手动 / webhook 触发，两种 execution mode：

- `create_issue`：创建 issue，分配给 agent 或 squad，再走常规 issue 入队链。
- `run_only`：不创建 issue，直接创建 `agent_task_queue` 任务，适合一次性自动执行。

证据：

- schema：`server/migrations/042_autopilot.up.sql:14-18`（`execution_mode`/`concurrency_policy`）、`29-47`（trigger 支持 `schedule/webhook/api`）、`49-65`（run 状态与 issue/task link）。
- squad autopilot：`server/migrations/096_autopilot_squad_assignee.up.sql:10-19,24-39`（`assignee_type='squad'`，执行解析到 leader）。
- 核心 dispatch：`server/internal/service/autopilot.go:46-68`（admission gate，offline runtime skip）、`89-113`（按 mode 分发）、`145-248`（create_issue）、`284-348`（run_only 直接建 task 并 wakeup）。
- 手动触发：`server/internal/handler/autopilot.go:1322-1341`（`DispatchAutopilot(..., "manual", nil)`）。
- cron：`server/cmd/server/autopilot_scheduler.go:14-31`（30s ticker）、`71-107`（claim due triggers 后 dispatch）、`109-139`（算下次运行）。
- cron claim SQL：`server/pkg/db/generated/autopilot.sql.go:32-43,65-70`（原子 claim，`next_run_at` 置 NULL 防并发）。
- webhook：`server/internal/handler/autopilot_webhook.go:80-89,96-168`（规范化 envelope）、`212-232,234-257`（dedupe/signature）；`server/internal/handler/webhook_delivery.go:299-305`（replay/dispatch 调 `DispatchAutopilot(..., "webhook", ...)`）。
- CLI 入口：`server/cmd/multica/cmd_autopilot.go:18-21,56-89,119-167`（list/get/create/update/delete/trigger/runs/trigger-add/update/delete/rotate-url）。

## 7. 配置与扩展点

主要扩展点：provider CLI、agent 配置、skills / MCP、daemon 配置、project resources / local directories、autopilot triggers、squad roster / instructions。

证据：

- daemon config：`server/internal/daemon/config.go:20-63`（默认 server/poll/heartbeat/watchdog/GC/并发）、`73-105`（Config 字段）。
- provider 探测：`server/internal/daemon/config.go:175-240`（通过 env / custom path / default command 探测 `claude/codex/opencode/openclaw...`）。
- daemon CLI flags：`server/cmd/multica/cmd_daemon.go:74-87`（daemon-id/device/runtime-name/poll/heartbeat/agent-timeout/codex inactivity/max-concurrent/auto-update）。
- agent 配置 CLI：`server/cmd/multica/cmd_agent.go:158-174`（create 支持 runtime-id/runtime-config/model/custom-args/custom-env/mcp-config/visibility/max-concurrent-tasks）、`176-198`（update）。
- 配置矩阵文档：`CLI_AND_DAEMON.md:163-224`（daemon env/flags、GC、provider path/model/args env、参数优先级）。
- task 环境注入：`server/internal/daemon/daemon.go:2796-2829`（写入/清理 runtime-specific config）、`2832-2865`（task-scoped env）。

## 8. 面向用户群体

主要面向**软件团队**、工程 / 产品 / 设计 / 运营协作团队，以及希望把多个 AI coding agents 纳入正式任务管理、自动化和审计流程的组织。尤其适合：

- 想用多个 agents 放大吞吐、而非手工复制 prompt 到本地 CLI 的小团队。
- 已使用 Claude Code/Codex/Cursor/Gemini 等 CLI、希望接入 issue board、评论、runtime、skills、autopilot 的开发者。
- 需要自托管、workspace 隔离、权限、可回溯活动历史、通知、可复用 skill 的团队。
- 需要用 squad 做专家路由的大团队（按主题分派，而非每次手挑具体 agent）。

证据：

- `README.zh-CN.md:31-35`（把 coding agent 变成队友，面向“人类 + AI 团队”）、`47-51`（小团队 + agents 放大）、`55-63`（agent-as-teammate / squads / autonomous execution / autopilots / skills / 统一 runtime / 多 workspace）。
- `docs/product-overview.md:54-73`（传统 AI coding agent 痛点：复制 prompt、盯终端、无跨任务记忆、多 agent 无全局看板）。
- `apps/docs/content/docs/agents.zh.mdx:8-38`（agent 是 workspace 一等成员，可被分配 issue、评论、被 @、做 project lead）。
- `apps/docs/content/docs/how-multica-works.mdx:8-14,46`（平台分布式：server 管数据、daemon 在用户机器执行、runtime 是 daemon × AI coding tool；区别于 Linear/Jira 在于 agents 不在 server 上执行）。

## 9. 关键判断（Phase 1 调查方给出）

> 标注：以下为 Phase 1 multica 调查方基于上述证据给出的判断。

`multica` 的 Agentic 核心不在“模型智能”本身，而在把外部 coding-agent CLI 变成具有**身份、权限、任务队列、工作目录、会话恢复、评论/issue 交互、自动化触发和团队路由**能力的可管理工作者。差异化重点是**协作编排与运行时治理**（issue board、comment trigger、squad leader routing、autopilot、runtime daemon、skills/MCP/project resources），而非单个 agent 的推理算法。
