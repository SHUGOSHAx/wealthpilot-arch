# 错误契约

M00/M01 Walking Skeleton 的错误码、HTTP 映射与 `ErrorResponse 1.0.0` 由 [Walking Skeleton Contract Addendum 1](walking-skeleton-contract-addendum-1.md) 冻结。

## ErrorResponse

```json
{
  "error_code": "DOCUMENT_PARSE_LOW_QUALITY",
  "message": "文档解析质量不足，无法形成可靠证据。",
  "trace_id": "uuid",
  "stage": "document.quality_check",
  "retryable": false,
  "partial_artifacts": ["uuid"],
  "checkpoint_status": "SAVED",
  "recovery_action": "上传清晰版本或人工确认页面。"
}
```

## 错误类别

| 前缀 | 语义 |
|---|---|
| `VALIDATION_` | 输入或 Schema 不合法 |
| `PRIVACY_` | 隐私等级或最小化检查失败 |
| `SOURCE_` | 来源、授权、下载或限流错误 |
| `DOCUMENT_` | 文件、解析、OCR 或质量错误 |
| `EVIDENCE_` | 证据缺失、冲突或引用错误 |
| `MODEL_` | Provider、超时、Schema 或预算错误 |
| `HARNESS_` | 计划、Checkpoint、步骤或状态错误 |
| `RISK_` | 硬风控阻断 |
| `BROKER_` | 订单、连接、回报或未知状态错误 |
| `EVOLUTION_` | 评测、审批、灰度或回滚错误 |

## 规则

- `message` 不包含 Secret、账户原文或 Provider 原始响应；
- `retryable=true` 只表示技术上可安全重试；
- 存在外部写入未知状态时必须 `retryable=false`；
- 产生部分 Artifact 时必须列出引用；
- 长任务错误保留 Checkpoint；
- 风控阻断使用业务错误，不映射为服务故障；
- API HTTP 状态与错误语义一致：400 校验、403 权限/隐私、409 状态冲突、422 业务不可执行、429 限流、502/503 外部依赖。
