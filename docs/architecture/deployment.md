# 部署架构

## 环境

| 环境 | 用途 | Broker |
|---|---|---|
| DEV | 本地开发和测试 | Fake/Paper |
| DEMO | 合成数据演示 | Paper |
| PAPER | 真实数据研究和模拟交易 | Paper |
| LIVE | 人工批准实盘 | QMT |

环境使用独立数据库 Schema、对象存储前缀、密钥和 Broker 配置。Paper 数据不得写入 Live 账户状态。

## 主服务

Docker Compose 运行：

- `web`；
- `api`；
- `worker`；
- `scheduler`；
- `postgres`；
- `redis`；
- `qdrant`；
- `notification`。

Model Gateway 位于 API/Worker 共享包中，所有外发模型请求经过统一出口。原始文件保存在本地加密对象目录，主密钥由系统 Keychain/Secret Store 管理。

## Windows QMT Gateway

Gateway 作为独立 Python 服务运行在安装 MiniQMT 的 Windows 设备，只监听白名单地址。它负责账户、资金、持仓、行情、订单、撤单、回报、心跳和二次风控，不承载 Agent 或模型。

## 配置

环境变量只引用 Secret 名称和非敏感配置：

```text
APP_ENV
DATABASE_URL
REDIS_URL
QDRANT_URL
DEFAULT_MODEL_PROVIDER
DEFAULT_FAST_MODEL
DEFAULT_DEEP_MODEL
QMT_GATEWAY_URL
```

API Key、数据库密码、证书和签名密钥由 Secret Store 注入。

## 备份与恢复

1. 停止写入或创建一致性快照；
2. 备份 PostgreSQL 和对象清单；
3. 加密备份文件；
4. 恢复后验证账本平衡、对象哈希、Artifact 引用和迁移版本；
5. 重建 Qdrant 索引；
6. Live 环境恢复后保持只读，人工确认再开启交易。

## 升级

数据库迁移先在 Demo/Paper 回放，校验通过后进入 Live。涉及 Artifact Schema 的升级必须支持读取旧版本或提供可审计迁移。涉及订单、安全和风控的升级在 Live 默认只读状态完成。
