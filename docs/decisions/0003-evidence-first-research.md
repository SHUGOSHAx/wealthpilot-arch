# 0003：投资研究证据优先

## 状态

Accepted

## 适用范围

文档 RAG、市场数据、Agent 报告、Bull/Bear、Decision Memo 和用户界面。

## 最终决定

所有重要研究事实先进入 Evidence Ledger。Agent 报告和 Decision Memo 只能引用 Evidence ID；数值推导使用金融计算 DSL；用户可以从结论跳转到来源、页码、表格和计算链。

## Non-obvious rationale

在多 Agent 链路中把引用附加在最终文本末尾无法证明中间 Agent 使用了同一事实，也无法识别单位、期间、版本和来源冲突。独立 Evidence Ledger 让所有 Agent 共享同一事实对象并接受统一审计。

## 实现约束

- 数值 Evidence 必须包含单位和期间；
- 文档 Evidence 必须可定位页码和 Block；
- 置信度由可观察质量信号计算；
- 无有效 Evidence 的主张不能进入高置信度结论。

## 验收方式

- 重要主张证据覆盖率 ≥95%；
- 引用页码准确率 ≥95%；
- 无来源数字进入 Decision Memo 的数量为 0。
