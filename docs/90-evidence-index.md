# 90 · 证据索引

本研究的事实性结论来自 Phase 1 的两份只读源码调查。本索引汇总来源，便于交叉 review 追溯。

## Phase 1 来源（任务 issue 评论）

研究任务 issue：**GHO-53**（`研究 multica 与 omnigent 的 AI Agentic 实现、差异和 omnigent 用法`）。

| 调查 | 调查方 | 评论 ID | 锚定提交 |
| --- | --- | --- | --- |
| omnigent 源码调查 | Frontend Architecture Expert - Codex@Ghost | `29ac468d-9b22-4921-913c-4febdb82cdca` | `7a199c94515bb1de7cfffcb1a764bf23bc534880` |
| multica 源码调查 | Backend Engineer - Codex@Ghost | `509f3de1-2ddc-4ca3-b83b-1e8638bf7fb0` | 未在 Phase 1 记录（建议补记） |

> 两份调查均声明只读：`git status --short` 为空，未修改 / 提交 / 推送。

## multica 关键证据文件（路径速查）

- 定位 / 用户：`README.zh-CN.md`、`docs/product-overview.md`、`apps/docs/content/docs/agents.zh.mdx`、`apps/docs/content/docs/how-multica-works.mdx`
- provider 适配：`server/pkg/agent/agent.go`、`server/pkg/agent/codex.go`、`server/pkg/agent/claude.go`
- 数据模型：`server/migrations/001_init.up.sql`、`004_agent_runtime_loop.up.sql`、`042_autopilot.up.sql`、`090_task_is_leader.up.sql`、`096_autopilot_squad_assignee.up.sql`
- 调度 / 执行：`server/internal/service/task.go`、`server/internal/daemon/daemon.go`、`server/pkg/db/generated/agent.sql.go`
- squad：`server/internal/handler/squad.go`、`squad_briefing.go`、`squad_comment_trigger_test.go`
- autopilot：`server/internal/service/autopilot.go`、`server/internal/handler/autopilot.go`、`autopilot_webhook.go`、`webhook_delivery.go`、`server/cmd/server/autopilot_scheduler.go`、`server/pkg/db/generated/autopilot.sql.go`、`server/cmd/multica/cmd_autopilot.go`
- 配置：`server/internal/daemon/config.go`、`server/cmd/multica/cmd_daemon.go`、`server/cmd/multica/cmd_agent.go`、`CLI_AND_DAEMON.md`

（具体行号见 [10-multica.md](10-multica.md) 各节。）

## omnigent 关键证据文件（路径速查）

- 定位 / 用法：`README.md`、`pyproject.toml`、`docs/AGENT_YAML_SPEC.md`、`docs/POLICIES.md`
- CLI：`omnigent/cli.py`
- spec：`omnigent/spec/types.py`
- harness / runtime：`omnigent/runtime/harnesses/__init__.py`、`process_manager.py`、`_runner.py`、`_executor_adapter.py`、`omnigent/runtime/README.md`、`omnigent/runtime/workflow.py`、`omnigent/runner/app.py`
- inner executors：`omnigent/inner/claude_sdk_executor.py`、`codex_executor.py`、`openai_agents_sdk_executor.py`、`cursor_executor.py`、`antigravity_executor.py`、`pi_executor.py`、`databricks_supervisor_executor.py`
- tools / policy：`omnigent/tools/builtins/spawn.py`、`omnigent/tools/builtins/`、`omnigent/policies/base.py`、`omnigent/runtime/policies/builder.py`
- server：`omnigent/server/routes/sessions.py`
- 示例 agent：`omnigent/resources/examples/polly/config.yaml`、`omnigent/resources/examples/debby/config.yaml`
- 前端：`ap-web/package.json`

（具体行号见 [20-omnigent.md](20-omnigent.md) 各节。）

## 复现说明

- 两个研究对象用 `multica repo checkout <url>` 拉取（只读调查）。
- 校验证据时请锚定上表提交（multica 提交待补），因行号随源码演进会变。
