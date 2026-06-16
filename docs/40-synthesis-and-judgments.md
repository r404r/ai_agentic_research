# 40 · 综合判断与待确认项

> 本文档集中放置**综合判断**（在汇总两份 Phase 1 证据时形成的推断），与 `10`/`20` 的原始证据明确区分。每条判断标注其依据与不确定度，便于交叉 review 时核验。

## 1. 综合判断

### J1 · 主要定位不同、分层可叠加，但本地单用户场景存在重叠
- **判断**：omnigent 解决“单机/会话级的多 harness 统一运行与编排”，multica 解决“团队级的任务编排与运行时治理”，二者主要处于 agentic 栈的不同层次、可叠加；但**并非完全互补**——在单机/单用户的本地多 agent 编排场景下，两者功能确有交叠、存在竞争关系。
- **依据**：multica 定位为不做推理的调度器 + 任务管理平台（`docs/product-overview.md:46-88`）；omnigent 定位为 meta-harness 会话运行时（`README.md:5-7,28-50`）。
- **不确定度**：中。两库均能独立完整运行，是否设计为协同栈未在代码中直接验证；“互补 vs 竞争”取决于具体使用场景，不作为确定结论。

### J2 · multica 的差异化在“协作编排与运行时治理”，而非模型智能
- **判断**：multica 价值核心是 issue/comment/squad/autopilot/runtime 这套**管理与治理层**。
- **依据**：见 [10-multica.md](10-multica.md) 第 2、5、6、9 节证据。
- **不确定度**：低（Phase 1 调查方亦持此结论）。

### J3 · omnigent 的差异化在“跨 harness 统一会话 + sub-agent 一等编排”
- **判断**：omnigent 价值核心是把多种 harness 收敛到统一会话/SSE 协议，并把 sub-agent 做成独立 session。
- **依据**：见 [20-omnigent.md](20-omnigent.md) 第 1、4、6 节证据（`_HARNESS_MODULES`、`_executor_adapter.py`、`spawn.py`）。
- **不确定度**：低。

### J4 · 用户群体重叠但侧重不同
- **判断**：两者都服务“使用多种 AI coding agent 的开发者/团队”，但 multica 重团队异步任务流，omnigent 重个人/会话实时编排与受控环境运行。
- **依据**：multica 用户群证据（`README.zh-CN.md`、`how-multica-works.mdx`）；omnigent 用户群证据（`README.md:138-177,241-257,293-307`）。
- **不确定度**：中（用户群体为基于文档定位的归纳，非用户调研数据）。

## 2. 待确认项（建议交叉 review 时核验）

1. **协同关系**（J1）：omnigent 是否可作为 multica 的某个 provider/runtime 执行外壳？需在 multica `provider` 列表与 omnigent host 接口层比对，目前**无直接证据**。
2. **omnigent 沙箱实现细节**：bwrap/seatbelt 的具体启用条件与默认策略仅见于 README 与 YAML spec 描述，未深入 runner 实现代码核验。
3. **multica 与 omnigent 的会话恢复语义差异**：multica 的 session 串行/恢复（task 层）与 omnigent 的 attach/fork（conversation 层）是否在语义上对应，尚未逐行比对。
4. **版本与时效**：omnigent 调查锚定提交 `7a199c94515bb1de7cfffcb1a764bf23bc534880`；multica 未在 Phase 1 记录锚定提交，建议补记，便于复现。

## 3. 维护建议

- 本仓库文档与源码强相关，源码演进后应更新对应文件路径/行号证据。
- 新增结论务必沿用本仓库约定：**事实附证据，推断单独标注**。
- 交叉 review 的修订建议与结论变更，建议以新文档或本文件追加节记录，保留可追溯历史。
