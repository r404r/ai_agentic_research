# 90 · 证据索引

本研究的事实性结论来自 Phase 1 的两份只读源码调查。本索引汇总来源，便于交叉 review 追溯。

## Phase 1 来源（任务 issue 评论）

研究任务 issue：**GHO-53**（`研究 multica 与 omnigent 的 AI Agentic 实现、差异和 omnigent 用法`）。

| 调查 | 调查方 | 评论 ID | 锚定提交 |
| --- | --- | --- | --- |
| omnigent 源码调查 | Frontend Architecture Expert - Codex@Ghost | `29ac468d-9b22-4921-913c-4febdb82cdca` | `7a199c94515bb1de7cfffcb1a764bf23bc534880` |
| multica 源码调查 | Backend Engineer - Codex@Ghost | `509f3de1-2ddc-4ca3-b83b-1e8638bf7fb0` | `241a3582cf3d5cb7c9ce9c290877b33758cff8f5`（Codex 于 Phase 3 在该提交上复核行号引用成立） |

> 两份调查均声明只读：`git status --short` 为空，未修改 / 提交 / 推送。
>
> [10-multica.md](10-multica.md) 的文件路径与**行号锚定于** multica 提交 `241a3582`；[20-omnigent.md](20-omnigent.md) 锚定于 omnigent 提交 `7a199c94`。行号随源码演进会变，核验时请 checkout 对应提交。

## multica 关键证据文件（路径速查）

每条附 1–2 个代表性 `file:line`（锚定提交 `241a3582`）以降低二次核验成本；完整行号见 [10-multica.md](10-multica.md) 各节。

- 定位 / 用户：代表性 `docs/product-overview.md:46-88`、`README.zh-CN.md:29-35`；另见 `apps/docs/content/docs/how-multica-works.mdx:8-14`、`agents.zh.mdx:8-38`
- provider 适配：代表性 `server/pkg/agent/agent.go:15-60`；另见 `server/pkg/agent/codex.go:78-85`、`server/pkg/agent/claude.go:147-195`
- 数据模型：代表性 `server/migrations/004_agent_runtime_loop.up.sql:1-15`、`042_autopilot.up.sql:14-18`；另见 `001_init.up.sql:35-49`、`090_task_is_leader.up.sql:1-9`、`096_autopilot_squad_assignee.up.sql:10-19`
- 调度 / 执行：代表性 `server/internal/service/task.go:990-1102`、`server/internal/daemon/daemon.go:1995-2090`；另见 `server/pkg/db/generated/agent.sql.go:523-562`
- squad：代表性 `server/internal/handler/squad_briefing.go:21-89`、`squad.go:982-1014`；另见 `squad_comment_trigger_test.go:116-199`
- autopilot：代表性 `server/internal/service/autopilot.go:284-348`；另见 `server/cmd/server/autopilot_scheduler.go:71-107`、`server/internal/handler/autopilot_webhook.go:96-168`、`server/cmd/multica/cmd_autopilot.go:18-21`
- 配置：代表性 `server/internal/daemon/config.go:175-240`、`CLI_AND_DAEMON.md:163-224`；另见 `server/cmd/multica/cmd_agent.go:158-174`、`cmd_daemon.go:74-87`

## omnigent 关键证据文件（路径速查）

每条附代表性 `file:line`（锚定提交 `7a199c94`）；完整行号见 [20-omnigent.md](20-omnigent.md) 各节。

- 定位 / 用法：代表性 `README.md:5-7`（meta-harness 定位）、`README.md:138-177`（用法）；另见 `pyproject.toml`（`project.scripts`）、`docs/AGENT_YAML_SPEC.md:1-42`
- CLI：代表性 `omnigent/cli.py:4949-5121`（`run` flags）；另见 `cli.py:4540-4802`（`_dispatch_run`）、`cli.py:4867-4949`（`attach`）
- spec：代表性 `omnigent/spec/types.py:480`（`ExecutorSpec` 起，定义跨多行）；同文件 `:784/:1248/:1320/:1344` 为各 spec 定义起始行
- harness / runtime：代表性 `omnigent/runtime/harnesses/__init__.py`（`_HARNESS_MODULES` 注册表）；另见 `process_manager.py`、`_executor_adapter.py`、`omnigent/runner/app.py:1-4`
- inner executors：代表性 `omnigent/inner/claude_sdk_executor.py`；另见 `codex_executor.py`、`openai_agents_sdk_executor.py`、`cursor_executor.py`、`antigravity_executor.py`、`pi_executor.py`、`databricks_supervisor_executor.py`
- tools / policy：代表性 `omnigent/tools/builtins/spawn.py`（`SysSessionSendTool`，sub-agent enum）；另见 `omnigent/runtime/policies/builder.py`、`omnigent/policies/base.py`
- server：代表性 `omnigent/server/routes/sessions.py`（`parent_conversation_id`/`sub_agent_name`/child spend）
- 示例 agent：代表性 `omnigent/resources/examples/polly/config.yaml`、`debby/config.yaml`
- 前端：`ap-web/package.json`（React/TanStack/xterm/Monaco/TipTap）

## 复现说明

- 两个研究对象用 `multica repo checkout <url>` 拉取（只读调查）。
- 校验证据时请锚定上表提交（multica `241a3582`、omnigent `7a199c94`），因行号随源码演进会变。
