# 30 · multica vs omnigent：并排对比与差异

> 本文档汇总两份 Phase 1 调查，给出用户要求的对比维度。表格为事实性归纳（依据 [10-multica.md](10-multica.md) / [20-omnigent.md](20-omnigent.md) 的证据）；表后的“差异解读”含**综合判断**，已单独标注。

## 1. 并排对比表（(c)）

| 维度 | multica | omnigent |
| --- | --- | --- |
| **定位** | 人 + AI coding agent 共用 issue 工作流的**任务管理 / Managed-Agents 平台**；不做模型推理，是调度器 | 面向多种 AI agent 的 **meta-harness**：统一运行 / 会话 / 治理 / 沙箱层 |
| **架构模型** | **服务端持久化工作流 + 本地 daemon 执行 + 外部 AI CLI provider**（server 管数据、daemon 在用户机执行） | **分层 harness**：Spec(YAML) → Server/AP(FastAPI) → Runner → Harness/inner executor → Web UI |
| **agent 执行方式** | daemon claim task → 准备隔离工作目录 → 注入上下文 → 启动外部 **coding-agent CLI**（Claude/Codex/Copilot/Gemini…），通过 task-scoped token 回写 | runner 为每个 session 懒启动 **harness 子进程**（一会话一进程，Unix socket），把统一 REST/SSE 转为具体 SDK/CLI 调用 |
| **被包装对象** | 外部 **CLI**：Claude Code、Codex、Copilot、OpenCode、OpenClaw、Gemini、Cursor Agent、Kimi、Kiro 等 | 多 **harness/SDK**：claude-sdk、claude-native、codex(-native)、pi(-native)、openai-agents、cursor、antigravity、databricks_supervisor |
| **多 agent 编排** | **Squad + leader 路由**：任务给 squad 先触发 leader，leader 用 mention / child issue 委派成员；以 **issue / 评论 / 子 issue** 为编排载体 | **Sub-agent as session**：子 agent 是独立 conversation/session，纳入 session 树，可并行、续接、带独立 harness/model/tools（`sys_session_send`） |
| **治理与沙箱** | **运行时治理**：runtime online/offline、agent capacity、并发槽、串行规则、权限/可见性、审计；隔离工作目录由 daemon 准备 | **Policy/Guardrails 拦截** request/response/tool_call/tool_result（ALLOW/ASK/DENY），成本/工具/安全策略；**OS 沙箱**：Linux bwrap、macOS seatbelt、云 sandbox/managed host |
| **协作模式** | **异步、任务流式**：issue board、评论触发、@mention、autopilot（定时/手动/webhook）、squad 专家路由；以“看板 + 评论”协作 | **同步、会话式**：live session 共享、co-drive、attach、fork、share；以“实时会话 + web UI”协作 |
| **部署形态** | **分布式自托管**：中心 server（数据/调度）+ 用户机器上的 daemon（runtime × provider × workspace）；agents 不在 server 上执行 | **本地优先 + 可选远程**：本地 `omnigent server start` + web UI(`:6767`)，或 `omnigent host`/`login` 接远端 server；移动端/云沙箱可选 |
| **目标用户** | 软件团队 / 多角色协作团队 / 想把多 AI agent 纳入正式任务管理、自动化、审计的组织 | 个人开发者、远程协作团队、agent/平台开发者、企业/受控环境用户 |

## 2. 差异解读

### 2.1 编排载体不同：任务流 vs 会话流

- **multica** 把 agent 编排建立在**任务管理原语**（issue / comment / squad / autopilot）之上，天然异步、可审计、可回溯，适合“把 agent 当同事排进看板”。
- **omnigent** 把编排建立在**会话原语**（session / sub-agent / inbox）之上，强调实时、可接管、可 fork，适合“在一个活会话里指挥多个 agent 协同”。

### 2.2 抽象层次不同：CLI 调度 vs harness 抽象

- **multica** 包装的是外部 **CLI 进程**，统一接口是 `agent.Backend.Execute`（prompt/options → 具体 CLI 参数）。
- **omnigent** 包装的是多种 **harness/SDK**，统一接口是 inner executor adapter（Omnigent request/SSE ↔ inner SDK 事件），抽象层更贴近 SDK 协议。

### 2.3 治理重心不同：运行时/组织治理 vs 调用级 policy

- **multica** 的治理偏**运行时与组织层**：并发、容量、串行、可见性、审计、runtime 在线状态。
- **omnigent** 的治理偏**单次调用层**：policy/guardrails 在 request/tool_call/tool_result 粒度拦截，叠加 OS 级沙箱。

### 2.4 协作半径不同：团队看板 vs 实时会话

- **multica** 面向**团队级异步协作**（多人多 agent 共享看板、评论、自动化）。
- **omnigent** 面向**会话级实时协作**（share/co-drive/fork 一个活 session）。

## 3. 综合判断：互补而非竞争

> **综合判断（本研究在综合两份证据时给出，非 Phase 1 原文结论）。**

两者解决的是 agentic 工作流的**不同层次**，可叠加而非替代：

- **omnigent** 偏“**单机/会话级的多 harness 运行时**”——让一个人或一个会话能统一驱动并编排多种 agent。
- **multica** 偏“**团队级的任务编排与运行时治理平台**”——让一个组织能把多个 agent 当成可被分配、触发、审计的成员。

一个合理的协同设想是：omnigent 作为某个 multica runtime/provider 之下的执行外壳，或 multica 在团队层调度、omnigent 在会话层运行。**此协同关系为推断，需在代码层进一步验证**，详见 [40-synthesis-and-judgments.md](40-synthesis-and-judgments.md)。
