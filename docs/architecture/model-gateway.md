# Model Gateway

## Provider

- `OpenAICompatibleAdapter`：默认国内兼容 API；
- `OpenAIAdapter`：OpenAI 原生接口；
- `AnthropicAdapter`：Anthropic 原生接口；
- `FakeModelAdapter`：测试和离线演示。

业务 Agent 只依赖统一 `ModelGateway`，不得导入厂商 SDK。

## 路由等级

| 等级 | 任务 |
|---|---|
| FAST | 意图、分类、实体、查询改写 |
| STANDARD | 摘要、事件抽取、结构化整理 |
| DEEP | 基本面、估值、行业和辩论 |
| CRITIC | 证据审计、风险复核和决策审查 |

关键 Decision Memo 使用不同 Provider 的 CRITIC 调用复核。缺少第二 Provider 时允许输出研究，但必须增加 `independent_review_unavailable` 警告。

## 统一请求

```python
class ModelRequest(BaseModel):
    run_id: UUID
    task_type: str
    messages: list[Message]
    response_schema: dict | None
    privacy_level: str
    reasoning_level: str
    max_output_tokens: int
    deadline: datetime
```

响应记录 Provider、模型、内容、结构化输出、输入/输出 Token、延迟、人民币估算成本和缓存状态。

## 调用管线

```text
ModelRequest
→ 隐私与最小化检查
→ Prompt 版本解析
→ 任务等级与 Provider 路由
→ 缓存检查
→ 限流和并发配额
→ Provider 调用
→ Schema 校验
→ 可修复错误重试
→ 成本与 Trace
→ ModelResponse
```

## 可靠性

- 429、临时网络和 5xx 使用带抖动指数退避；
- 超过任务 deadline 不再发起重试；
- 连续故障触发 Provider 熔断；
- 降级不得绕过隐私和 Schema；
- 同一缓存键包括 Prompt、模型、Schema、输入哈希和工具版本；
- 结构化输出无法修复时返回显式错误，不降级为自由文本。

## 隐私

- P0 可发送；
- P1 经脱敏后发送；
- P2 仅发送必要事实、比例、区间或任务内别名；
- P3 在网关入口直接拒绝；
- 请求日志只保存哈希、分类、Token、成本和安全摘要。

系统不依赖 Ollama 或本地生成式模型。云模型不可用时，确定性服务继续运行，Agent 任务暂停并保留 Checkpoint。
