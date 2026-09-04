# WealthPilot Arch 文档规则

## 仓库职责

本仓库只描述 WealthPilot 已采用的产品行为和实现架构。不要加入生产代码、实验实现、真实数据、凭据、讨论记录或未采用设计。

## 写作要求

- 使用中文解释行为，代码标识符、协议字段和枚举使用英文。
- 只写最终状态；禁止保留 intermediate attempts、草稿、TODO 方案和未合入状态。
- 只解释会影响正确性、安全性、可复现性或实施方式的 non-obvious rationale。
- 不重复显而易见的理由，不撰写方案选择过程。
- 使用稳定、一致的 Agent、Artifact、API、事件和领域术语。
- Trace 只描述结构化行为，不要求或展示模型隐藏思维链。

## 变更一致性

- `PLAN.md` 是总纲。领域或架构行为变化时必须同步更新它。
- 新增或修改公开接口时同步更新 `docs/contracts/api-contracts.md`。
- 修改 Artifact 时同步更新 `docs/contracts/domain-artifacts.md`。
- 修改事件或错误语义时同步更新对应契约。
- 修改用户可见行为时同步更新 `docs/product/acceptance-scenarios.md` 和测试标准。
- 修改已采用架构决策时同步更新 `docs/decisions/` 中对应记录。
- Mermaid 图中的名称必须与正文契约一致。

## 不可擅自改变的边界

- 单用户、本地数据优先、A 股优先。
- 所有生成式 Agent 使用云模型 API，不依赖 Ollama 或本地生成式模型。
- P2 原始个人数据不得发送云模型；P3 数据永不进入模型上下文。
- 金额、仓位、回测和风险由确定性代码计算。
- 投资结论必须引用 Evidence Ledger，并绑定 Research Snapshot。
- Agent 不得修改硬风控、审批、Broker 权限或实盘开关。
- 实盘订单必须逐单人工批准。
- 自进化候选必须通过评测并由用户批准。

## 提交前检查

- 检查 Markdown 内部链接。
- 检查 Artifact、API 和事件命名一致性。
- 检查是否包含真实数据、Secret、本机私有路径或账户标识。
- 检查是否出现讨论过程、中间尝试或未采用状态。
- 检查所有约束是否可以通过验收场景验证。
