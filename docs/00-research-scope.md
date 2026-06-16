# 00 · 研究背景、范围与方法

## 背景与目标

用户提出研究 AI Agentic 相关课题，聚焦两个内部工具 `multica` 与 `omnigent`，需要回答三个问题：

- (a) 两个工具的**具体实现内容**、二者**差异**，以及各自**面向的用户群体**。
- (b) `omnigent` 的**主要用法**、**特点**，以及它**解决的现实课题**。
- (c) 一份**并排对比表**（定位 / 架构模型 / agent 执行方式 / 多 agent 编排 / 治理与沙箱 / 协作模式 / 部署形态 / 目标用户）。

研究结果沉淀进本仓库 `ai_agentic_research`（研究专用，初始为空，本次初始化）。

## 范围与约束

- **只读对象**：`multica`、`omnigent` 仅作只读调查，全程未修改 / 提交 / 推送。
- **可写对象**：仅本仓库 `ai_agentic_research`，由发起方明确授权写入 / 提交 / 推送。
- 本次（Phase 2）交付为**草稿**，推送到工作分支，**不合并默认分支**；交叉 review 由发起方统一安排。

## 方法

研究采用两阶段流程：

1. **Phase 1（源码调查，并行、互不重叠）**：两名工程 agent 分别对 `multica` 与 `omnigent` 做只读源码调查，结论必须附证据（文件路径、模块 / 函数、配置、命令观察），结果以结构化评论贴回任务 issue。
2. **Phase 2（综合 + 初始化仓库）**：将两份 Phase 1 结论综合为结构化研究交付，保留证据引用，区分原始证据与综合判断，并初始化本研究仓库。

> 本文档及 `10`/`20`/`30` 的事实性结论均**源自 Phase 1 调查**（见 [证据索引](90-evidence-index.md)）。综合性推断集中在 [40-synthesis-and-judgments.md](40-synthesis-and-judgments.md)，并在正文中以 **“综合判断”** 显式标注。

## 术语表

| 术语 | 含义 |
| --- | --- |
| **Agentic 工具** | 以 LLM agent 为核心、能自主执行多步任务的系统。 |
| **Harness / meta-harness** | 包装一个或多个 agent 运行时的执行外壳；meta-harness 指统一多种 harness 的上层抽象（omnigent 自我定位）。 |
| **Provider / backend** | 被包装的外部 AI coding CLI（如 Claude Code、Codex、Cursor、Gemini）。 |
| **Runtime（multica）** | daemon × provider × workspace 的执行单元；agent 绑定到 runtime。 |
| **Daemon（multica）** | 运行在用户机器上的本地进程，探测本地 CLI、claim 任务、准备工作目录、启动外部 agent CLI。 |
| **Squad（multica）** | 可被分配任务的对象 + leader agent 路由层。 |
| **Autopilot（multica）** | 可定时 / 手动 / webhook 触发的自动化。 |
| **Session / conversation（omnigent）** | omnigent 中一次 agent 运行的会话单元，可被 attach / fork / share。 |
| **Sub-agent（omnigent）** | 独立 conversation / session 的子 agent，纳入 session 树。 |
| **Policy / Guardrails** | 对 request / response / tool_call / tool_result 的治理规则（ALLOW / ASK / DENY）。 |
