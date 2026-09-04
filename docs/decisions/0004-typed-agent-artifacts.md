# 0004：Agent 使用类型化 Artifact 协作

## 状态

Accepted

## 适用范围

Agent Harness、任务 Handoff、持久化、Replay、UI 和评测。

## 最终决定

Agent 输入和输出使用版本化 Pydantic Artifact。自由文本只作为 Artifact 中受约束的解释字段，不作为跨 Agent 的唯一协议。

## Non-obvious rationale

自由文本 Handoff 无法稳定验证字段完整性、证据引用、单位、风险结果和版本，且难以安全恢复或重放。类型化 Artifact 让工作流在模型输出之后执行确定性校验。

## 实现约束

- Agent 注册输入与输出 Schema；
- 已发布 Artifact 不可变；
- Schema 使用语义版本；
- 不合法输出经过受限修复后仍失败则终止任务；
- Replay 使用持久化 Artifact，而不是重新解析旧文本。

## 验收方式

- 非法 Schema 不进入下游 Agent；
- Checkpoint 可从 Artifact 恢复；
- 契约测试覆盖所有生产 Agent。
