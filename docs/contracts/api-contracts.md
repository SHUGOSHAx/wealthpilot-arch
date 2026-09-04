# API 契约

M00/M01 Walking Skeleton 的 Import、Run、SSE 与最小 FinancialSnapshot API 由 [Walking Skeleton Contract Addendum 1](walking-skeleton-contract-addendum-1.md) 冻结；其机器可读 Schema 是该切片的规范源。

## 通用规则

- 基础路径：`/api/v1`；
- 请求和响应使用 JSON，文件上传使用 multipart；
- 金额使用十进制字符串；
- 时间使用 UTC ISO 8601；
- 写请求支持 `Idempotency-Key`；
- 长任务返回 `202 Accepted`、`run_id` 和事件 URL；
- 分页使用稳定 Cursor；
- 错误遵循 [错误契约](error-contracts.md)。

## Imports

```text
POST /imports
GET  /imports/{import_id}
POST /imports/{import_id}/commit
POST /imports/{import_id}/cancel
```

上传返回 ImportBatch。Commit 必须引用用户确认的预览版本。

## Wealth

```text
GET  /wealth/snapshot?as_of=
GET  /wealth/cashflow?from=&to=
POST /wealth/scenarios
GET  /wealth/goals
POST /wealth/goals
PUT  /wealth/goals/{goal_id}
GET  /wealth/ips
POST /wealth/ips
```

IPS 更新创建新版本，不覆盖历史版本。

## Documents、Coverage 与 Evidence

```text
POST /documents
POST /documents/discover
GET  /documents/{document_id}
GET  /documents/{document_id}/blocks
GET  /coverage/{symbol}?as_of=
GET  /evidence/{fact_id}
```

Discover 接收 symbol、as_of、document requirements 和来源范围，返回异步 Run。

## Research

```text
POST /research/runs
GET  /research/runs/{run_id}
GET  /research/runs/{run_id}/events
GET  /research/runs/{run_id}/memo
GET  /research/runs/{run_id}/trace
POST /research/runs/{run_id}/challenge
POST /research/runs/{run_id}/feedback
POST /research/runs/{run_id}/cancel
```

Challenge 创建补充研究子任务，不修改已有 Decision Memo。

## Screening 与 Comparison

```text
POST /screeners/interpret
POST /screeners/run
GET  /screeners/{screener_id}
POST /comparisons
```

Interpret 返回 ScreeningSpec 和自然语言解释；Run 只接受通过 Validator 的 Spec。

## Portfolio 与 Backtest

```text
GET  /portfolio?as_of=
GET  /portfolio/risk?as_of=
POST /portfolio/rebalance
POST /backtests
GET  /backtests/{backtest_id}
```

## Orders

```text
POST /orders/proposals
GET  /orders/{proposal_id}
POST /orders/{proposal_id}/approve
POST /orders/{proposal_id}/reject
POST /orders/{proposal_id}/cancel
```

Approve 必须提交 Proposal 内容哈希；Live 额外提交 WebAuthn assertion。

## Evolution

```text
GET  /evolution/candidates
GET  /evolution/candidates/{candidate_id}
POST /evolution/{candidate_id}/approve
POST /evolution/{candidate_id}/reject
POST /evolution/{candidate_id}/rollback
```

## Health 与 Notification

```text
GET  /model-gateway/health
GET  /broker/qmt/health
GET  /notifications
POST /notifications/{notification_id}/read
```
