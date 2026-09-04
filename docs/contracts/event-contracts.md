# 事件契约

M00/M01 Walking Skeleton 的 Event envelope、七类事件与 Last-Event-ID replay 由 [Walking Skeleton Contract Addendum 1](walking-skeleton-contract-addendum-1.md) 冻结为可执行契约。

## Envelope

```json
{
  "event_id": "uuid",
  "run_id": "uuid",
  "event_type": "task.started",
  "occurred_at": "2026-09-04T02:00:00Z",
  "sequence": 7,
  "payload_version": "1.0.0",
  "payload": {}
}
```

- `sequence` 在单个 Run 内单调递增；
- 客户端按 `event_id` 去重；
- 断线重连通过 Last-Event-ID 续传；
- Payload 不包含 P2/P3 原文或隐藏思维链。

## Run 事件

| 事件 | 最小 Payload |
|---|---|
| `run.started` | objective、intent、budget |
| `plan.created` | task_count、plan_artifact_id |
| `run.completed` | result_artifact_ids、cost |
| `run.failed` | error_code、stage、retryable |

## 情报与证据事件

| 事件 | 最小 Payload |
|---|---|
| `coverage.checked` | symbol、score、missing_count |
| `source.discovered` | source_id、document_type |
| `document.downloaded` | document_id、version、hash |
| `document.parsed` | document_id、block_count、quality |
| `evidence.added` | fact_id、entity、claim_type |

## Agent 事件

| 事件 | 最小 Payload |
|---|---|
| `task.started` | task_id、agent_id、input_artifact_ids |
| `tool.called` | task_id、tool_id、safe_parameter_summary |
| `task.completed` | task_id、output_artifact_ids、usage |
| `debate.started` | bull_case_id、bear_case_id |

## 风险与审批事件

| 事件 | 最小 Payload |
|---|---|
| `risk.blocked` | risk_assessment_id、blocking_rule_ids |
| `approval.required` | proposal_id、expires_at、environment |

事件 Payload 增加字段保持向后兼容；删除、重命名或改变语义需要提升主版本。
