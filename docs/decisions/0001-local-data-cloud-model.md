# 0001：本地数据与云模型

## 状态

Accepted

## 适用范围

个人数据存储、文档处理、Model Gateway、Agent 推理和部署。

## 最终决定

个人账本、资产、合同、持仓、原始文件和审计数据保存在用户设备。所有生成式 Agent 通过统一 Model Gateway 使用云模型 API；国内 OpenAI-Compatible API 为默认 Provider，保留 OpenAI 和 Anthropic Adapter。系统不依赖 Ollama 或本地生成式模型。

## Non-obvious rationale

长文档和多 Agent 并发需要稳定的长上下文与推理能力，本地消费级显卡无法作为可重复部署基线。隐私通过数据最小化、确定性本地计算和 P0–P3 出口控制实现，而不是依赖本地生成式推理。

## 实现约束

- P2 原文禁止发送，P3 在网关入口拒绝；
- 所有模型请求保存版本、Token、成本和安全摘要；
- 云模型不可用时保存 Checkpoint，确定性服务继续；
- 业务 Agent 不直接依赖厂商 SDK。

## 验收方式

- P2/P3 Fixture 泄漏为 0；
- Provider 故障可恢复或安全暂停；
- 无 Ollama 和本地 GPU 时完整系统可运行。
