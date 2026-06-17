# 50 · 企业大规模开发落地方案

> 本文是 GHO-75 的新增综合交付，回答用户关于“核心理念、瀑布式团队改变、敏捷团队改变、企业系统大规模开发如何落地”的问题。
>
> 证据基础：本文复用 [10-multica.md](10-multica.md)、[20-omnigent.md](20-omnigent.md)、[30-comparison.md](30-comparison.md)、[90-evidence-index.md](90-evidence-index.md) 的源码调查结论，并在本次任务中重新只读复核两个研究对象的当前 checkout：`multica` 为 `241a3582cf3d5cb7c9ce9c290877b33758cff8f5`，`omnigent` 为 `7a199c94515bb1de7cfffcb1a764bf23bc534880`。两个研究对象均未修改。

## 0. 一句话结论

`multica` 的核心理念是：**把 AI coding agent 变成组织里可分配、可触发、可审计、可治理的队友**。它以 issue / comment / squad / autopilot / runtime 为中心，解决“团队如何把多个 agent 纳入正式研发流程”的问题。

`omnigent` 的核心理念是：**把多种 agent harness 收敛到统一会话、工具、子 agent、policy 和沙箱层**。它以 session / AgentSpec / sub-agent / policy / runner 为中心，解决“个人或团队如何在一个可共享、可 fork、可治理的会话里统一驱动多种 agent”的问题。

对企业级开发而言，两者的最佳心智模型不是“替代人”，而是把研发流程改造成 **人负责目标、边界、取舍、验收；agent 负责可拆分、可验证、可回放的执行单元**。管理重点从“盯人做事”转为“设计清晰工作包、治理运行环境、审查证据链、控制风险和成本”。

## 1. 两个工具的核心理念

### 1.1 multica：任务流里的 Managed Agents 平台

`multica` 不是新的模型或单个 coding agent。它的架构把外部 CLI provider（Claude Code、Codex、Cursor、Gemini 等）包成平台中的 agent 身份，再让这些 agent 进入团队的 issue 工作流。

核心理念可以拆成五点：

| 理念 | 含义 | 源码 / 文档依据 |
| --- | --- | --- |
| **任务是组织协作原子** | 工作以 issue、comment、子 issue、状态、assignee 表达。agent 通过这些原语接任务、报告进度、说明阻塞。 | `server/internal/service/task.go` 的 issue/comment/task 入队链；[10-multica.md](10-multica.md) §2-3 |
| **agent 是可管理队友** | agent 有 profile、assignee 身份、runtime 绑定、capacity、instructions、skills、活动历史。 | `docs/product-overview.md`、`server/migrations/001_init.up.sql`、[10-multica.md](10-multica.md) §2 |
| **运行在本地，治理在平台** | server 管数据、权限、队列、事件；daemon 在用户机器上 claim task、准备工作目录、启动外部 CLI。 | `apps/docs/content/docs/how-multica-works.mdx`、`server/internal/daemon/daemon.go` |
| **编排靠组织结构而非 prompt 串联** | squad 是 leader 路由层，autopilot 是定时 / webhook / 手动触发自动化，project resources 提供长期上下文。 | `server/internal/handler/squad.go`、`server/internal/service/autopilot.go`、[10-multica.md](10-multica.md) §5-6 |
| **可追溯优先** | 任务、评论、状态、PR、metadata、runtime 状态形成审计链。适合团队管理和跨角色协作。 | `agent_task_queue`、issue/comment schema、[30-comparison.md](30-comparison.md) |

因此，`multica` 适合放在企业研发流程的 **任务管理与执行治理层**：把一个个明确任务分派给人或 agent，把结果回收到同一看板和同一评论链，便于管理吞吐、阻塞、质量和责任。

### 1.2 omnigent：会话流里的 meta-harness

`omnigent` 的自我定位是 meta-harness：它不只包装一个 CLI，而是在多种 harness / SDK 之上提供统一会话层。它把 Claude Code、Codex、Cursor、Pi、OpenAI Agents、自定义 YAML agent 等收敛成统一的 session、runner、tool、policy 和 UI 协作模型。

核心理念可以拆成五点：

| 理念 | 含义 | 源码 / 文档依据 |
| --- | --- | --- |
| **会话是执行和协作原子** | session/conversation 可以 attach、fork、share；消息、文件、终端、子 agent 同步到同一会话树。 | `README.md`、`omnigent/server/routes/sessions.py`、[20-omnigent.md](20-omnigent.md) §2-4 |
| **harness 可替换、可组合** | `executor.harness` 选择 claude-sdk、codex、cursor、pi、openai-agents 等；CLI 可覆盖 harness/model。 | `omnigent/runtime/harnesses/__init__.py`、`docs/AGENT_YAML_SPEC.md` |
| **sub-agent 是独立 session** | 子 agent 不是普通函数调用，而是可并行、可续接、有独立 conversation 的会话节点。 | `omnigent/tools/builtins/spawn.py`、`omnigent/server/routes/sessions.py` |
| **policy / sandbox 是内建治理层** | policy 可在 request、response、tool_call、tool_result 等阶段 ALLOW / ASK / DENY；沙箱控制 OS 访问。 | `docs/POLICIES.md`、`omnigent/runtime/policies/builder.py`、`omnigent/runtime/policies/engine.py` |
| **AgentSpec 是可版本化的作业说明书** | YAML 声明 prompt、harness、model、tools、sub-agents、policies、os_env、terminals。 | `docs/AGENT_YAML_SPEC.md` |

因此，`omnigent` 适合放在企业研发流程的 **会话执行与多 harness 实验层**：一个资深开发者、tech lead 或 agent orchestrator 可以在一个 live session 里组织多个子 agent 并行探索、实现、评审，同时用 policy 和 sandbox 控制风险。

## 2. 传统瀑布式开发团队需要做出的改变

传统瀑布式团队通常按“需求 → 设计 → 开发 → 测试 → 发布”大阶段推进。引入 `multica` / `omnigent` 后，不建议取消阶段门；真正需要改变的是：**每个阶段输出都必须变成 agent 可执行、可验证、可审查的工作包**。

### 2.1 共同改变

| 传统做法 | 需要改变为 | 原因 |
| --- | --- | --- |
| 大文档一次性交付给下游 | 阶段文档 + 可执行 issue / session spec + 验收标准 | agent 不能可靠处理含糊的大包任务，必须拆成边界清晰的执行单元 |
| 项目经理按人力排期 | 同时管理人力、agent capacity、runtime 在线率、review capacity | agent 执行也消耗机器、模型、工具权限和 reviewer 时间 |
| 设计评审只看文档 | 设计评审看文档、原型、代码 spike、agent 输出证据 | agent 可以提前生成验证性 spike，阶段门应检查可运行证据 |
| QA 在末期集中介入 | QA 规则、测试数据、验收脚本前置 | agent 产出快，若质量门后置，会把缺陷快速放大 |
| 安全 / 合规发布前审查 | policy、sandbox、权限、secret 边界前置 | agent 能调用工具和改文件，安全边界必须在执行前声明 |

### 2.2 使用 multica 时，瀑布团队的流程改造

`multica` 更适合把瀑布阶段转成 **issue 树 + 阶段门 + agent 队列**。

推荐做法：

1. **立项阶段**：项目经理创建 project，建立 Epic / parent issue。每个阶段目标用 issue 描述清楚，避免把整套系统一次性丢给 agent。
2. **需求阶段**：业务分析师把需求拆成“业务规则、接口、数据、验收样例、非功能指标”。每条需求成为可引用的 issue 或文档链接。
3. **架构阶段**：架构师创建架构评审 issue，要求 agent 做只读源码研究、边界分析、ADR 草案、风险列表。架构师本人保留最终决策权。
4. **详细设计阶段**：技术负责人把模块拆成子 issue，写明输入、输出、文件范围、验收命令、禁止事项。严格串行的任务创建为 backlog，等前置完成后再转 todo。
5. **开发阶段**：按模块或能力分配给不同 agent / squad。squad leader 负责读取上下文、再委派具体实现或研究任务。
6. **集成阶段**：建立集成验证 issue，要求 agent 汇总变更、跑测试、修复冲突、更新迁移说明。
7. **测试阶段**：QA agent 负责测试补齐、回归脚本、边界 case；人类 QA 负责抽查和业务验收。
8. **发布阶段**：Release manager 使用 issue 状态、PR 链接、测试结果、阻塞记录做发布决策；autopilot 可做定期依赖检查、变更摘要、安全巡检。

管理重点：

- 用 issue 状态表达真实阶段，不用口头同步替代状态。
- 用 comment 要求 agent 报告“做了什么、证据是什么、阻塞在哪里”。
- 用 metadata 只记录高信号事实，如 PR、deploy、blocked_reason，不把运行日志塞进 metadata。
- 用 squad 处理专家路由，不让项目经理手工猜哪个 agent 最适合。
- 用 autopilot 处理周期性工作，不把例行检查压给人。

### 2.3 使用 omnigent 时，瀑布团队的流程改造

`omnigent` 更适合把瀑布阶段转成 **可共享会话 + agent spec + policy gate + 可 fork 证据链**。

推荐做法：

1. **立项阶段**：定义项目级 agent specs，例如需求分析 agent、架构评审 agent、实现 orchestrator、测试 agent、迁移评审 agent。
2. **需求阶段**：业务分析师用 shared session 与 agent 迭代需求澄清，保留会话证据；复杂问题可 fork 出多个探索会话。
3. **架构阶段**：架构师运行 orchestrator session，让不同 harness 的子 agent 分别做方案、风险、替代路线，再由人类收敛为 ADR。
4. **设计阶段**：把确认后的模式固化到 YAML agent spec、policy、工具权限中，保证后续执行使用同一套作业说明。
5. **开发阶段**：orchestrator 派生 coding sub-agents，各自使用独立 workspace / worktree，避免互相覆盖。
6. **测试阶段**：review / QA 子 agent 从会话树读取上下文，独立验证实现，不与实现 agent 共享同一盲点。
7. **发布阶段**：发布负责人检查 session artifacts、policy 记录、测试输出、人工批准点，再决定是否合入。

管理重点：

- 不把 omnigent session 当临时聊天窗口，而要当“可回放的执行记录”。
- policy 必须在执行前配置，尤其是 shell、网络、凭据、生产数据访问。
- 子 agent 的 role、harness、model、工具权限应写进 AgentSpec，不靠临场口头说明。
- fork 用于探索不同方案，最终交付仍需收敛到版本库文档 / 代码 / issue 记录，避免知识只留在会话里。

## 3. 敏捷开发团队需要做出的改变

敏捷团队已有 backlog、迭代、review、retro 等机制，和 agentic 工具更接近。但敏捷团队常见问题是：任务粒度太粗、验收标准口头化、review 被速度压垮。引入 `multica` / `omnigent` 后，敏捷团队要把“人类敏捷”升级为 **人机混合敏捷**。

### 3.1 共同改变

| 敏捷实践 | 改造要求 |
| --- | --- |
| Backlog refinement | 每个 story 必须有 agent-readable context：目标、非目标、文件范围、验收命令、风险、依赖 |
| Sprint planning | 估算时区分 human capacity、agent runtime capacity、review capacity、CI capacity |
| Daily standup | 不只问人，也看 agent issue 状态、blocked comments、失败任务、runtime 离线 |
| Definition of Ready | ready 前必须有测试入口、验收标准、权限边界、可拆分性 |
| Definition of Done | done 前必须有代码 / 文档变更、测试证据、review 结论、回滚或迁移说明 |
| Sprint review | 展示人和 agent 协作完成的可运行结果，不展示 prompt 过程本身 |
| Retrospective | 分析 agent 失败类型：需求不清、上下文不足、权限不足、测试缺失、review 瓶颈 |

### 3.2 使用 multica 时，敏捷团队的工作方式

`multica` 可以直接承接敏捷团队的 backlog 和看板。推荐把 agent 当成 sprint team 的可分配成员，但不要把 agent 视为无限容量。

Sprint 内推荐流程：

1. **Refinement**：Product Owner 与 Tech Lead 把 story 拆成可独立验收的 issue。若一个 issue 需要多个 agent 同时改同一文件，应继续拆分或先做设计 issue。
2. **Planning**：Scrum Master / Project Lead 确认本迭代 agent 数量、runtime 在线、provider 可用、review 人力、CI 队列。
3. **Execution**：普通实现 issue 分给具体 agent；跨领域 issue 分给 squad，由 leader 判断后再委派。
4. **Daily**：查看 board 中 agent 的 blocked、failed、in_review；及时补充上下文或改派，不让失败 task 静默堆积。
5. **Review**：人类 reviewer 不只看 diff，还要看 agent 的证据：测试命令、设计理由、影响范围、未覆盖风险。
6. **Retro**：统计哪些 issue 适合 agent、哪些不适合；把成功模式固化为 skills、模板、checklist 或 autopilot。

### 3.3 使用 omnigent 时，敏捷团队的工作方式

`omnigent` 适合敏捷团队在 story 级或 spike 级做快速并行探索和实现。它更像一个会话级“临时作战室”。

Sprint 内推荐流程：

1. **Story kick-off**：开发负责人开启 session，加载相关 spec、工具、policy，并明确本次目标。
2. **Spike / 方案对比**：对高不确定 story，fork 多个探索 session，或让 Debby/Polly 类 orchestrator 调度不同 sub-agent 产生方案。
3. **Implementation**：orchestrator 把独立任务派给 coding sub-agents，每个子 agent 使用隔离 worktree。
4. **Cross-review**：不同 harness / model 的 reviewer 检查实现，降低单一模型盲点。
5. **Merge handoff**：把最终结论、代码、测试结果、风险项落回正式仓库和 issue，不把交付只留在 session 中。
6. **Retro**：复盘 session tree、policy ASK/DENY、成本、失败工具调用，优化 AgentSpec 和默认 policy。

## 4. 企业系统基于 multica 的大规模开发流程

### 4.1 适用定位

当企业目标是“让多个团队、多个 agent、多个代码库围绕正式 backlog 协作”时，优先用 `multica` 做主控平面。

推荐定位：

- `multica` 是 **研发任务系统和 agent 调度系统**。
- issue 是需求、任务、缺陷、技术债、发布事项的统一载体。
- squad 是专家路由层。
- autopilot 是例行自动化和外部事件触发层。
- runtime / daemon 是执行资源池。

### 4.2 角色与职责

| 角色 | 职责 |
| --- | --- |
| 业务负责人 / Product Owner | 定义业务目标、优先级、验收标准，确认最终交付是否满足业务价值 |
| 项目经理 / Delivery Manager | 维护 project、issue 层级、里程碑、风险、跨团队依赖和节奏 |
| 系统架构师 | 定义模块边界、技术路线、数据流、集成方式、非功能要求、ADR 和回滚策略 |
| Squad Leader Agent | 读取复杂 issue，判断任务类型，拆分或委派给合适 agent / 人 |
| 专项 Agent | 执行具体任务，如后端实现、前端实现、测试补齐、文档、迁移检查、安全审查 |
| Runtime Owner | 维护 daemon、provider CLI、认证、机器容量、在线率和本地工作目录策略 |
| Reviewer / Maintainer | 审查 agent 输出，合并代码，保护主干质量 |
| QA / 测试负责人 | 定义测试矩阵、回归策略、验收数据和质量门 |
| Security / Compliance | 定义哪些数据、命令、仓库、凭据可被 agent 使用，审查风险记录 |
| Release Manager | 管理发布节奏、变更冻结、回滚、上线验证和发布后巡检 |

### 4.3 Step by step 作业流

| Step | 作业内容 | 主要责任人 | 产出 |
| --- | --- | --- | --- |
| 1 | 建立 workspace / project，绑定相关 repo，定义 issue prefix 和项目资源 | Delivery Manager、Runtime Owner | project、resources、访问范围 |
| 2 | 建立 agent 角色矩阵：架构、后端、前端、测试、安全、文档、发布 | 架构师、Runtime Owner | agent profiles、instructions、skills、runtime 绑定 |
| 3 | 建立 squads：按域或能力组织，如 “支付域小队”“前端平台小队”“测试小队” | 架构师、Delivery Manager | squad leader、成员、职责说明 |
| 4 | 把企业系统拆成领域 / 模块 / 里程碑 issue 树 | Product Owner、架构师 | parent issues、子 issue、依赖关系 |
| 5 | 为每个模块定义 Definition of Ready：上下文、边界、验收命令、风险、禁止事项 | Tech Lead、QA、安全 | issue 模板、checklist |
| 6 | 先派发研究 / 设计 issue，要求 agent 输出 ADR、模块边界、数据流、迁移影响 | 架构师、专项 agent | ADR、设计文档、风险列表 |
| 7 | 架构师评审并冻结关键边界，再批量创建实现子 issue | 架构师、Delivery Manager | 可并行实现 issue |
| 8 | 将独立实现 issue 分给 agent；跨域 issue 分给 squad leader；串行任务用 backlog 停住后续步骤 | Delivery Manager、Squad Leader | agent task queue、执行评论 |
| 9 | agent 执行后在 comment 中报告变更、测试、阻塞；失败任务及时补上下文或改派 | 专项 agent、Tech Lead | 代码 / 文档变更、测试证据 |
| 10 | Reviewer 做人工 review；QA agent 或测试小队补回归和边界测试 | Reviewer、QA | review 结论、测试结果 |
| 11 | Release Manager 汇总 PR、迁移、配置、回滚、风险，组织集成发布 | Release Manager、架构师、QA | release issue、发布说明、回滚方案 |
| 12 | Autopilot 执行周期性巡检：依赖、测试摘要、bug triage、发布后健康检查 | Release Manager、Runtime Owner | 定期 issue / run 记录 |

### 4.4 管理细节

- **容量管理**：agent 数量不是吞吐上限，review 人力、runtime 并发、CI 并发才是实际瓶颈。
- **上下文管理**：重复使用的规范写成 skills 或项目资源，不要每次在 issue 里重写。
- **风险管理**：高风险变更先做设计 issue 和 spike，不直接给实现 agent。
- **冲突管理**：并行 issue 要避免改同一文件；必须改同一文件时，先串行或指定集成 agent。
- **审计管理**：关键决策写入 issue comment / ADR，PR 只承载代码，不承载全部决策。
- **自动化管理**：适合 autopilot 的是周期性、可模板化、低歧义任务；不适合的是需要临场权衡的架构决策。

## 5. 企业系统基于 omnigent 的大规模开发流程

### 5.1 适用定位

当企业目标是“在一个复杂任务中统一调度多个 harness / sub-agent，并让团队共同观察、接管、fork 会话”时，优先用 `omnigent` 做会话执行层。

推荐定位：

- `omnigent` 是 **多 agent 会话运行时和实验 / 实现工作台**。
- AgentSpec 是可版本化的作业说明书。
- sub-agent 是可独立执行、可并行、可续接的 worker session。
- policy / sandbox 是执行前置治理。
- shared session / fork 是协作和方案分歧处理机制。

### 5.2 角色与职责

| 角色 | 职责 |
| --- | --- |
| Session Owner | 启动和管理关键 session，决定 fork、attach、share、结束和归档 |
| Orchestrator Agent / Tech Lead | 分解任务，调度 sub-agent，综合结果，控制工作树隔离 |
| Coding Sub-agent | 在指定 harness / model / worktree 中实现具体任务 |
| Review Sub-agent | 独立检查实现、测试、边界条件、可维护性和安全问题 |
| AgentSpec Maintainer | 维护 YAML agent、sub-agent、tools、model、harness、instructions |
| Policy Admin | 维护 server / session / agent policy，处理 ASK/DENY 规则、成本和权限边界 |
| Host / Runner Owner | 维护 host、runner、CLI/SDK 依赖、沙箱、凭据和资源隔离 |
| Human Reviewer | 对最终代码、架构取舍、业务影响和发布风险负责 |
| QA / Compliance | 验收测试、生产数据边界、审计要求和合规材料 |

### 5.3 Step by step 作业流

| Step | 作业内容 | 主要责任人 | 产出 |
| --- | --- | --- | --- |
| 1 | 定义项目 agent specs：orchestrator、coder、reviewer、tester、安全、文档 | AgentSpec Maintainer、架构师 | YAML specs、工具与权限清单 |
| 2 | 配置 harness / model 组合，按任务类型选择 Claude、Codex、Cursor、Pi、OpenAI Agents 等 | Tech Lead、Host Owner | harness 矩阵 |
| 3 | 配置 policy 和 sandbox：shell、网络、文件、凭据、成本、敏感数据 | Policy Admin、Security | 默认 policy、session policy |
| 4 | 启动项目 session 或 story session，加载目标、仓库、上下文和验收标准 | Session Owner | live session |
| 5 | Orchestrator 做任务分解，确定哪些任务可并行、哪些必须串行 | Orchestrator Agent、Tech Lead | 子任务计划 |
| 6 | 通过 sub-agent 派发实现 / 研究 / 测试任务，每个子 agent 使用独立 worktree 或资源边界 | Orchestrator Agent | child sessions、worktrees |
| 7 | 父 session 接收 async inbox / completion，综合输出，要求必要的补充验证 | Orchestrator Agent | 综合结论、待修复项 |
| 8 | Review sub-agent 和人类 reviewer 交叉审查，必要时 fork 新 session 复验 | Review Sub-agent、Human Reviewer | review 记录 |
| 9 | 将最终代码、文档、测试证据、风险项落盘到正式仓库 / issue / ADR | Tech Lead、Session Owner | 可审计交付物 |
| 10 | 归档或分享 session，沉淀可复用 AgentSpec、policy、prompt、checklist | AgentSpec Maintainer | 复用资产 |

### 5.4 管理细节

- **会话不是最终归档系统**：session 适合执行和协作，但最终结论要进入仓库、issue、ADR 或发布文档。
- **子 agent 必须隔离**：并行实现必须使用独立 worktree / workspace，避免互相覆盖。
- **policy 必须前置**：不要等 agent 触碰敏感工具后再补权限边界。
- **harness 组合要有意图**：不同 harness 适合不同任务。实现、评审、推理、UI、测试可以使用不同模型 / harness，降低同质化失败。
- **成本是管理对象**：把模型选择、最大工具调用、session spend 变成 policy 和 review 关注点。
- **fork 是探索工具，不是分裂事实源**：多个 fork 最后必须由人类或 orchestrator 收敛为单一决策。

## 6. 两者在企业中的组合方式

从现有代码证据看，`multica` 和 `omnigent` 尚无明确的“一方直接作为另一方 provider/runtime”集成证据。因此组合使用应先作为架构方案和试点，不应假设已原生打通。

推荐组合：

| 层级 | 推荐工具 | 用法 |
| --- | --- | --- |
| 企业级 backlog、责任、状态、审计 | multica | 官方任务源、agent 分配、squad 路由、autopilot、PR / 发布追踪 |
| story / spike / complex session 执行 | omnigent | 多 harness 会话、sub-agent 并行、policy、sandbox、share/fork |
| 最终事实源 | 正式代码仓库 + multica issue | 代码、文档、ADR、测试证据、发布记录必须回写正式仓库和 issue |

落地策略：

1. 用 `multica` 管理正式需求、缺陷、任务和发布。
2. 对高不确定、高并行、需要多 harness 比较的任务，创建 `multica` issue 后，由负责人启动 `omnigent` session 执行探索或实现。
3. `omnigent` session 的结果必须回写 `multica` issue：结论、代码位置、测试、风险、是否需要后续子 issue。
4. 若未来要让 `omnigent` 成为 `multica` 的 provider/runtime，应先做小范围 spike：认证、工作目录、task token、结果回写、取消、超时、日志、policy 冲突都要逐项验证。

## 7. 90 天采用路线图

### 第 0-30 天：试点

- 选择一个非核心但真实的企业系统模块。
- 用 `multica` 建立项目、agent、runtime、2-3 个 squad。
- 用 `omnigent` 建立 2-3 个 AgentSpec：orchestrator、coder、reviewer。
- 只允许 agent 处理低风险任务：文档、测试、重构建议、小缺陷、只读研究。
- 建立最小治理：禁止生产凭据、要求测试证据、人工 review 必须通过。

成功标准：

- 至少 10 个 agent issue 完成且可审计。
- 至少 3 个 omnigent session 产出被落盘。
- 没有绕过 review 合入主干的 agent 变更。

### 第 31-60 天：扩大到团队流程

- 把 backlog refinement 模板改为 agent-readable 模板。
- 建立 squad：后端、前端、测试、安全、文档。
- 把重复任务做成 skills / AgentSpec / policy。
- 建立 autopilot：依赖巡检、bug triage、测试摘要、文档更新提醒。
- 统计 agent 失败原因并改进模板。

成功标准：

- agent 承担 20%-40% 的可拆分任务执行。
- review 周期未明显拉长。
- 缺陷逃逸率不高于试点前基线。
- runtime 在线率、CI 失败率、policy ASK/DENY 有统计。

### 第 61-90 天：制度化

- 把 agent 工作纳入正式 Definition of Ready / Done。
- 建立 agent 变更准入：高风险代码必须由不同 harness 或人类做二次 review。
- 把安全、成本、权限、生产数据访问纳入 policy。
- 将成功模式沉淀为组织级 templates、skills、AgentSpecs、runbooks。
- 决定是否试点更深集成，如 omnigent 作为 multica 下游执行外壳。

成功标准：

- 关键流程可审计：从需求 issue 到 PR / release 都能追踪。
- agent 失败有分类和处理 SLA。
- 组织明确哪些任务适合 agent，哪些必须由人主导。

## 8. 成功指标与主要风险

### 8.1 成功指标

| 指标 | 说明 |
| --- | --- |
| Cycle time | 从 issue ready 到 review ready 的时间是否下降 |
| Review load | 人类 reviewer 是否从“补救 agent 错误”转向“审查关键取舍” |
| Rework rate | agent 产出被大幅返工的比例 |
| Test evidence rate | 完成 issue 中包含明确测试证据的比例 |
| Runtime availability | daemon / host / provider 可用率 |
| Policy signal | ASK / DENY 的数量、原因、处理时长 |
| Defect escape | 上线后缺陷是否增加 |
| Knowledge reuse | skills、AgentSpec、模板复用次数 |

### 8.2 主要风险

| 风险 | 表现 | 缓解 |
| --- | --- | --- |
| 任务含糊 | agent 做偏、反复返工 | 强化 DoR：目标、非目标、文件范围、验收命令 |
| 并行冲突 | 多 agent 改同一文件或设计互相打架 | 先做架构拆分，独立 worktree，必要任务串行 |
| 审查瓶颈 | agent 产出堆积，人类 review 跟不上 | 限制 agent 并发，增加 reviewer agent 做预审 |
| 权限风险 | agent 访问不该访问的数据或命令 | policy / sandbox / secret 边界前置 |
| 成本失控 | 多 session / 多 agent 产生不可见模型成本 | policy 限额、session 预算、日报统计 |
| 事实源分裂 | 结论留在 session，issue / repo 没同步 | 规定最终交付必须回写仓库和 issue |
| 过度自动化 | autopilot 生成大量低价值 issue | 设定触发条件、频率、owner、定期清理 |

## 9. 最终建议

对于企业系统大规模开发，推荐先采用下面的分层落地：

1. **用 multica 作为正式研发协作平面**：任务、状态、责任、审计、agent 分派、squad 路由、autopilot。
2. **用 omnigent 作为复杂任务会话执行平面**：多 harness 对比、sub-agent 并行、会话协作、policy / sandbox 控制。
3. **把人类职责上移**：业务价值、架构边界、安全策略、最终 review、发布决策仍由人负责。
4. **把 agent 任务下沉为可验证单元**：每个 agent 任务必须小、清晰、有验收、有上下文、有权限边界。
5. **把治理做在执行前**：runtime、policy、sandbox、review、CI、成本和回滚都要在批量使用前建立。

这样落地时，企业不会只是“多了几个会写代码的机器人”，而是获得一套更高吞吐的研发操作系统：`multica` 管组织级工作流，`omnigent` 管会话级执行力，人类团队管方向和质量。
