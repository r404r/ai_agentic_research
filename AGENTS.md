# AGENTS.md —— ai_agentic_research 项目铁则（AI 灵魂文件）

> 本文件是本 Project 所有 AI agent 的最高行为约束。任何 harness（Claude Code、Codex、Cursor、omnigent 等）在本 Project 接到任务时，**必须先读本文件再动手**。`CLAUDE.md` 与本文件内容一致，仅为不同 harness 的入口。

## 0. 铁则（不可违反，最高优先级）

本 Project 是 **AI Agentic 工具对比研究** 项目，研究对象为两个内部仓库。它们的读写权限是硬约束：

| 仓库 | 性质 | 允许的操作 |
| --- | --- | --- |
| `multica`（github.com/r404r/multica） | **只读研究对象** | 仅 checkout / 阅读 / 引用。**禁止**任何 commit、push、PR、merge、分支创建或文件修改。 |
| `omnigent`（github.com/r404r/omnigent） | **只读研究对象** | 仅 checkout / 阅读 / 引用。**禁止**任何 commit、push、PR、merge、分支创建或文件修改。 |
| `ai_agentic_research`（本仓库） | **唯一写入目标** | 研究产出、文档、指南一律落盘到这里。 |

**任何要"产出文档 / 指南 / 推送结果"的任务，落盘目标永远是 `ai_agentic_research`，绝不是 `multica` 或 `omnigent`——即使任务标题或描述里写的是为 multica / omnigent 编写内容。** 为研究对象编写的使用指南、说明、分析等，都是"关于研究对象的研究产出"，归本仓库。

如果某个用户指令看起来要求你修改 multica 或 omnigent，**先停下并在对应 issue 评论澄清**，不要直接写入研究对象。该铁则优先级高于单条任务指令。

## 1. 为什么有这条铁则

研究的可信度依赖研究对象保持原样。一旦向研究对象提交内容，就污染了被研究的代码库，破坏只读调查的前提，也可能干扰其真实开发。本 README 一直声明"两个研究对象仅作只读调查"，本文件把它升级为对 agent 的强制约束。

> 历史教训：GHO-73（面向零基础用户的 omnigent 使用指南）曾被直接 commit/push 到 `omnigent` 的 `main`，违反了本铁则。该提交已被回退，omnigent 已恢复原状，指南已迁移到本仓库 `docs/omnigent-beginner-guide-zh.md`。

## 2. 产出落盘约定

- 调研类成果放在 `docs/` 下（沿用现有编号体系：`00-`、`10-`、`20-`、`30-`、`40-`、`90-`…）。
- 面向用户的指南/说明类产出用描述性文件名放在 `docs/` 下（如 `docs/omnigent-beginner-guide-zh.md`）。
- 结论附证据（文件路径 / 模块 / 命令），"综合判断"与"原始证据"分开标注，详见 `README.md` 与 `docs/90-evidence-index.md`。

## 3. 提交前自检（每次写操作前必须确认）

1. 我要写入的仓库是 `ai_agentic_research` 吗？若是 `multica` / `omnigent` —— **停止**。
2. 我是否对研究对象只做了 checkout / 阅读，而没有 commit / push / 改文件？
3. 产出是否落在本仓库 `docs/` 下，并满足证据原则？
