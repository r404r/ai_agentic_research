# 20 · omnigent 的实现、用法、特点与现实课题

> 来源：Phase 1 `omnigent` 源码调查（只读，提交 `7a199c94515bb1de7cfffcb1a764bf23bc534880`，`git status --short` 为空）。证据为文件路径 + 行号；汇总见 [证据索引](90-evidence-index.md)。

## 1. 定位与解决的现实课题

`omnigent` 是一个 **meta-harness**：目标不是自研单一模型 agent，而是**统一** Claude Code、Codex、Cursor、Pi、OpenAI Agents、Antigravity 和自定义 YAML agents 的运行、协作、治理、沙箱与会话层。

它解决的现实课题（**(b) 的核心答案**）：**多种 agent / harness 已经存在，但缺少统一的运行、协作、治理和可观测会话层。** 用户不必在 Claude Code、Codex、Cursor、Pi、OpenAI Agents 和自写 agent 之间重写工具 / 会话 / 权限 / 沙箱；团队也能在同一 live session 中共享、接管、fork、查看文件 / 终端 / subagents，并用 policy 控制风险与成本。

证据：

- `README.md:5-7` —— “A meta-harness for all your AI agents”，提供 Claude Code、Codex、Cursor、Pi、自写 agent 的 common layer。
- `README.md:28-50` —— 列出核心问题：跨设备继续会话、多 agent 监督、任意模型 / 订阅 / API key / gateway、团队协作、云沙箱、policy 治理。
- `pyproject.toml` —— `project.scripts` 定义 `omnigent = "omnigent.cli:main"`、`omni = "omnigent.cli:main"`；依赖含 `fastapi/starlette/uvicorn/sqlalchemy/pexpect/mcp/openai/claude-agent-sdk/openai-agents/cursor-sdk`（同时覆盖服务端、终端控制、MCP、LLM SDK/harness）。

## 2. 主要用法 / 入口命令（(b)）

典型工作流：

- **安装**：`curl ... scripts/install_oss.sh | sh` 或 `uv tool install omnigent`（`README.md:61-84`）。
- **本地会话**：`omnigent` 启动终端会话 + 本地 web UI `http://localhost:6767`（`README.md:138-146`）。
- **指定 harness**：`omnigent claude`、`omnigent codex`、`omnigent run path/to/agent.yaml`（`README.md:159-163`）。
- **示例 agent**：`omnigent run examples/polly/`、`omnigent run examples/debby/`；可用 `--harness pi/openai-agents/cursor` 覆盖 orchestrator harness（`README.md:168-177`）。
- **服务 / 远程**：`omnigent server start` + `omnigent host`；远端 `omnigent login <server>` + `omnigent host <server>`（`README.md:181-257`）。
- **继续 / 协作**：`omnigent attach <session_id>`、`omnigent run --fork <session_id>`（`README.md:293-307`）。

CLI 证据：

- `omnigent/cli.py:1110`（click root）；`omnigent/cli.py:3828-4129`（`claude`/`codex`/`pi` wrapper 命令）。
- `omnigent/cli.py:4540-4802`（`_dispatch_run()` 把 `omnigent run` 路由到 server-backed REPL / one-shot / resume / fork）；`omnigent/cli.py:4949-5121`（`run()` 定义 `--harness --model -p --resume --continue --fork --no-session --server --host`）。
- `omnigent/cli.py:4867-4949`（`attach()` 仅 attach，不启动 server/runner/harness）。

## 3. 核心架构

分层：

- **Spec 层**：`AgentSpec` / YAML 描述 agent、executor、tools、sub_agents、guardrails、os_env、terminals。
- **Server / AP 层**：FastAPI 会话、权限、持久化、runner 路由、SSE/REST、UI 后端。
- **Runner 层**：在 host 上为每个 session 启动 harness 子进程，执行 tools/MCP/terminal/filesystem 资源。
- **Harness / inner executor 层**：适配 Claude SDK、Codex CLI、Pi、OpenAI Agents SDK、Cursor、Antigravity、Databricks Supervisor。
- **Web UI 层**：`ap-web` 前端（React / TanStack Query / xterm / Monaco / TipTap）。

证据：

- `omnigent/runtime/README.md` —— runtime 是 execution engine，给 spec + 用户输入后驱动 LLM、tools、skills、responses；是 library，server 是 primary host。
- `omnigent/spec/types.py`：`ExecutorSpec`（class 定义自 `:480` 起，字段 harness/model/auth/context_window/supervisor_tools 跨多行）、`ToolsConfig`（自 `:784` 起：agents/builtins/timeout/retry）、`PolicySpec`（`:1248` 起）、`GuardrailsSpec`（`:1320` 起）、`AgentSpec`（`:1344` 起）。上述行号为各 class/spec 定义的**起始行**，非单行包含全部字段。
- `omnigent/runtime/harnesses/__init__.py` —— 注册 `_HARNESS_MODULES`：`claude-sdk`、`claude-native`、`codex-native`、`codex`、`pi`、`pi-native`、`openai-agents`、`cursor`、`antigravity`、`databricks_supervisor`。
- `omnigent/runtime/harnesses/process_manager.py` —— 一会话一个 harness subprocess，经 Unix socket 暴露 HTTP client，负责 idle reaper、crash/orphan cleanup。
- `omnigent/runtime/harnesses/_runner.py` —— `python -m` harness 子进程入口（`--harness`、`--module`、`--socket`、`--conversation-id`、`--parent-pid`）。
- `omnigent/runner/app.py:1-4` —— runner FastAPI app 负责 spawn harness subprocess 并调度。

## 4. Agentic 实现方式 / 数据流

典型数据流：

1. 用户通过 CLI / web 创建或恢复 session。
2. Server 持久化 conversation/session/agent bundle，绑定 runner/host。
3. Runner 按 spec 的 `executor.harness` 查 `_HARNESS_MODULES`，为 conversation 懒启动 harness 子进程。
4. Harness 子进程把统一 REST/SSE 协议转为具体 SDK/CLI 调用。
5. LLM 输出 tool call 时，经 adapter/runner/server policy gate 后执行 MCP、本地工具、terminal、filesystem 或 `sys_session_send` 子 agent。
6. 子 agent 是独立 conversation/session，完成后通过 inbox / `async_work_complete` 唤醒父 agent。
7. Web UI 与 CLI 通过同一 session 流展示 messages、subagents、terminals、files、permissions。

证据：

- `omnigent/runtime/harnesses/_executor_adapter.py` —— 适配职责：Omnigent request → inner `Message`/`ExecutorConfig`，inner events → Omnigent SSE，`request.tools`/`instructions` 转发，`ctx.dispatch_tool` 桥接工具，cancel 传播。
- `omnigent/inner/*_executor.py` / `*_harness.py` —— 具体 harness executor：`claude_sdk_executor.py`、`codex_executor.py`、`openai_agents_sdk_executor.py`、`cursor_executor.py`、`antigravity_executor.py`、`pi_executor.py`、`databricks_supervisor_executor.py`。
- `omnigent/tools/builtins/spawn.py` —— `SysSessionSendTool`：子 agent 是 separate Omnigent agent sessions / own conversation / visible in session tree；支持 `(agent,title)` 自动 create-or-continue 或 `session_id` 续接；返回 `{task_id, kind: "sub_agent", conversation_id, status...}`；schema 动态把 declared sub-agent names 作为 `agent` enum（LLM 只能调度声明过的子 agent）。
- `omnigent/runtime/workflow.py:90-98`（async work 含 tools / sub-agents / client tools，经 `async_work_complete` drain）、`2195`（`_find_spec_by_name()` 递归查 sub-agent spec）。
- `omnigent/server/routes/sessions.py` —— 保存 `parent_conversation_id`、`sub_agent_name`、runner liveness、child spend、pending prompts（子 agent 纳入 session 树和 UI/权限/成本模型）。

## 5. 配置与扩展点

扩展点主要在 **YAML + Python 插件式模块**：

- Agent YAML：`name/prompt/instructions/executor/tools/policies/params/os_env/terminals/async/cancellable/timers`（`docs/AGENT_YAML_SPEC.md:1-42`）。
- Executor/harness：`executor.harness` 可选 `claude-sdk/openai-agents/codex/cursor/pi/antigravity/...`，CLI `--harness --model` 可覆盖（`docs/AGENT_YAML_SPEC.md:44-82`）。
- Tools：MCP server、本地 Python function、sub-agent、client runtime tool（`docs/AGENT_YAML_SPEC.md:94-177`）。
- OS/terminal：`os_env` 声明文件 / shell 权限和沙箱；`terminals` 声明 named interactive terminal（`docs/AGENT_YAML_SPEC.md:84-93,220-244`）。
- Policies：YAML `policies`/`guardrails` 配 function policy、phase、ASK timeout；`omnigent/runtime/policies/builder.py` —— policy 顺序为 session policies → agent spec policies → admin/default policies，子 agent 继承 root session policies。
- Builtin tools：`omnigent/tools/builtins/`（spawn、agents、web_search、file_tools、timer、policy、async_inbox、sys_terminal 等）。

## 6. 产品特点（(b)）

- **多 harness 统一**：同一 spec / 会话层可跑 Claude SDK、Codex、Pi、OpenAI Agents、Cursor、Antigravity、Databricks Supervisor（`omnigent/runtime/harnesses/__init__.py`）。
- **多 agent orchestration 是一等能力**：sub-agent 是独立 session（非简单函数调用），可并行、可继续、可被 UI 展示、可带独立 harness/model/tools。
- **协作 / 跨设备**：本地 server + web UI + mobile + attach/fork/share（`README.md:138-146,293-307`）。
- **治理**：policy 可拦截 request/response/tool_call/tool_result，ASK/DENY/ALLOW，支持成本 / 工具调用 / 安全策略（`README.md:317-353`；`omnigent/policies/base.py`；`omnigent/runtime/policies/builder.py`）。
- **沙箱与 host**：Linux bwrap、macOS seatbelt、云 sandbox / managed host，terminal/filesystem 资源由 runner 管理（`README.md:95-103,241-257`；`docs/AGENT_YAML_SPEC.md:84-93`）。
- **Agent 可由 agent 编写**：README “Write your own agent” 说明 agent 即 YAML，且 “agents can build agents”（`README.md:359-392`）。

## 7. 示例 agent 佐证

**Polly**（multi-agent coding orchestrator）：

- `omnigent/resources/examples/polly/config.yaml` —— `executor.config.harness: claude-sdk`；通过 `tools.agents`/`sys_session_send` 调度 `claude_code`、`codex`、`pi`；要求每个 worker 独立 worktree、并行、跨 vendor review。
- `README.md:179-190` —— Polly 自己不写代码，作为 tech lead 计划、分派到 coding sub-agents，再路由不同 vendor reviewer。

**Debby**（two-headed brainstorming partner）：

- `omnigent/resources/examples/debby/config.yaml` —— 主 brain `claude-sdk`，子 agent 为 `claude` 和 `gpt`，每个问题并行发给两者，inbox 唤醒后并列展示。
- `README.md:192-197` —— Debby 用 Claude 和 GPT 两个 heads，`/debate` 让二者互评再收敛。

## 8. 面向用户群体

- **个人开发者 / AI coding 用户**：用 `omnigent`、`omnigent claude`、`omnigent codex`、`omnigent run path/to/agent.yaml` 启动本地或浏览器会话（`README.md:138-177`）。
- **团队 / 远程协作用户**：分享 live session、co-drive、fork、权限（`README.md:293-307`；`omnigent/server/routes/sessions.py` 的 session permission/owner/share 通知）。
- **Agent / 平台开发者**：用 YAML 声明 agent、sub-agent、tools、policies、os_env、terminals（`docs/AGENT_YAML_SPEC.md:1-42`）。
- **企业 / 受控环境用户**：policy、成本限制、沙箱、Databricks/gateway/managed hosts（`README.md:241-257,317-353`；`docs/POLICIES.md`）。
