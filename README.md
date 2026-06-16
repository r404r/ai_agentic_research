# AI Agentic 工具研究：multica 与 omnigent

本仓库是 **AI Agentic 工具对比研究** 的产出库（研究专用）。研究对象为两个内部仓库：

- [`multica`](https://github.com/r404r/multica) —— 只读研究对象
- [`omnigent`](https://github.com/r404r/omnigent) —— 只读研究对象

> 两个研究对象仅作只读调查，研究过程中**未做任何修改、提交或推送**。本仓库是唯一的写入目标。

## TL;DR（一句话结论）

- **multica** 是“人 + AI coding agent 共用 issue 工作流”的 **任务管理 / Managed-Agents 平台**：把外部 coding-agent CLI 包装成可被分配 issue、评论触发、自动化触发的“团队成员”，核心价值在**协作编排与运行时治理**。
- **omnigent** 是面向多种 AI agent 的 **meta-harness（统一运行/会话层）**：在 Claude Code、Codex、Cursor、Pi、OpenAI Agents 等之上提供统一的运行、协作、治理、沙箱与会话能力，核心价值在**跨 harness 的统一会话与多 agent 编排**。
- 两者**主要定位不同、但存在重叠**：multica 偏“团队任务流 + 调度治理”，omnigent 偏“单机/团队会话运行时 + 多 harness 抽象”；在**单机/单用户的本地多 agent 编排**场景下，两者功能确有交叠、存在竞争关系。本研究不把“互补”当作确定结论（置信度见 [docs/40](docs/40-synthesis-and-judgments.md) J1）。详见对比文档。

## 文档导航

| 文档 | 内容 | 对应用户诉求 |
| --- | --- | --- |
| [docs/00-research-scope.md](docs/00-research-scope.md) | 研究背景、范围、方法、只读约束、术语表 | 全局上下文 |
| [docs/10-multica.md](docs/10-multica.md) | multica 的 Agentic 实现、架构、调度执行链、目标用户 | (a) 实现 |
| [docs/20-omnigent.md](docs/20-omnigent.md) | omnigent 的实现、**主要用法**、**特点**、**解决的现实课题**、目标用户 | (a)(b) |
| [docs/30-comparison.md](docs/30-comparison.md) | **并排对比表** + 差异分析 + 各自面向的用户群体 | (a)(c) |
| [docs/40-synthesis-and-judgments.md](docs/40-synthesis-and-judgments.md) | 综合判断（与原始证据区分标注）+ 待确认项 | 综合 |
| [docs/90-evidence-index.md](docs/90-evidence-index.md) | Phase 1 证据来源索引（评论 + 关键文件路径清单） | 可追溯 |

## 证据原则

本研究的事实性结论均来自 Phase 1 的源码调查（见 [证据索引](docs/90-evidence-index.md)），结论附带文件路径 / 模块 / 命令等证据。文档中**“综合判断”一律单独标注**，与原始证据区分，避免脱离证据的推断。

## 研究状态

- Phase 1：multica / omnigent 两份源码调查 —— **已完成**（见证据索引）。
- Phase 2：综合 + 初始化研究仓库 —— **本次产出**（草稿）。
- Phase 3：交叉 review —— 由发起方统一安排，尚未开始。
