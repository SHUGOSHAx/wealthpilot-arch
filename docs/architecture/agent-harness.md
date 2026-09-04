# Agent Harness

## 职责

LangGraph 描述状态流转，Harness 提供生产运行语义：

- Agent、Tool、Model 和 Artifact Registry；
- 类型化任务和 Handoff；
- 依赖图和并行执行；
- Token、人民币成本、步骤和时间预算；
- Retry、Timeout、Circuit Breaker；
- Checkpoint、Resume、Replay 和 Cancel；
- Human-in-the-loop；
- Tool 权限、副作用、幂等和补偿；
- Context Compression；
- Trace、审计和 Evaluation Harness。

## Agent 声明

每个 Agent 注册：

- `agent_id` 和版本；
- 输入与输出 Artifact Schema；
- 允许 Tool；
- 模型任务等级；
- 最大步骤、Token、成本和期限；
- 隐私等级上限；
- 是否允许并行；
- 重试与失败策略。

## Tool 权限

| 等级 | 语义 | 示例 |
|---|---|---|
| READ | 无状态读取 | 查询行情、读取文档 |
| COMPUTE | 确定性计算 | XIRR、回测、估值 |
| WRITE_REVERSIBLE | 可补偿写入 | 创建草稿、通知 |
| WRITE_SENSITIVE | 敏感写入 | 提交已批准订单 |

LLM 只能提出 ToolCall。Harness 校验 Schema、Agent 权限、隐私、预算和审批后执行。

## Run 状态

```python
class WealthRunState(BaseModel):
    run_id: UUID
    objective: str
    intent: str
    as_of: datetime
    privacy_level: str
    research_snapshot_id: UUID | None
    financial_snapshot_id: UUID | None
    plan: list[TaskNode]
    artifact_ids: dict[str, UUID]
    evidence_ids: list[UUID]
    warnings: list[str]
    risk_flags: list[str]
    token_budget: int
    cost_budget_cny: Decimal
    step_budget: int
    deadline: datetime
    checkpoint_version: int
    status: str
```

## 执行语义

- 无依赖的分析任务并行；
- Data Steward 完成 ResearchSnapshot 后才能启动研究 Agent；
- Evidence Auditor 完成后才能启动 Portfolio Manager；
- Risk Manager 未通过时不得创建 OrderProposal；
- 预算不足时保存 Checkpoint 并返回部分 Artifact；
- 同一幂等键只允许一次副作用；
- 不可重试错误立即终止受影响分支。

## Context Compression

上下文按 Document Card、Section Summary、Page Summary、Raw Block 逐级展开。Agent 共享 Evidence Ledger，不重复输入全文。压缩结果绑定来源和版本，不能删除影响条件、否定、单位、期间或例外的内容。

## Trace

Trace 保存 Task、Tool 参数安全摘要、Artifact、Evidence、模型、Token、成本、延迟、重试和状态。Trace 不保存 P2/P3 原文或隐藏思维链。
