# 0006：实盘交易逐单人工批准

## 状态

Accepted

## 适用范围

Order Proposal、Risk、Approval、Broker、QMT Gateway 和 Live 运维。

## 最终决定

Agent 只能生成不可变限价 `OrderProposal`。Live 订单必须通过确定性硬风控、用户审阅和 WebAuthn 逐单批准，再以 Ed25519 签名信封发送 Windows QMT Gateway。网关完成二次风控后调用 xtquant。

## Non-obvious rationale

将批准绑定不可变订单哈希，可以防止模型、网络重试或界面状态在用户确认后改变标的、方向、数量和价格。独立网关让 Agent 运行环境无法接触 Broker 凭据。

## 实现约束

- Live 默认关闭；
- 只允许限价单；
- Proposal 字段变化使批准失效；
- 信封包含 nonce、过期时间和幂等键；
- 未知状态不得自动重发；
- Kill Switch 阻止新订单并保留查询与撤单。

## 验收方式

- 无人工批准实盘订单数为 0；
- 重复请求产生重复订单数为 0；
- 修改 Proposal 后原批准无法使用；
- Agent 环境无法读取 Broker 凭据。
