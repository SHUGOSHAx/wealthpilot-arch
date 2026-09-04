# 运行时架构

## 组件

| 组件 | 职责 |
|---|---|
| Next.js Web | 工作台、审批、Trace 和 SSE 消费 |
| FastAPI | 认证、公开 API、命令校验和查询 |
| Agent Harness | 状态图、预算、权限、Checkpoint 和 Trace |
| Celery Worker | 文档、研究、回测、报告和定时任务 |
| Scheduler | 每日、交易日、周度、月度和事件任务 |
| Model Gateway | 模型路由、隐私、成本、重试和审计 |
| Document Worker | 下载、校验、Docling、OCR、索引 |
| Notification Service | 应用内通知和低敏邮件 |
| PostgreSQL | 领域数据、状态、审计和 Sparse 检索 |
| Qdrant | Dense 向量检索 |
| Redis | 队列、锁、短期状态和事件分发 |
| QMT Gateway | Windows 账户、行情和订单隔离网关 |

## 研究任务

```mermaid
sequenceDiagram
    actor User
    participant Web
    participant API
    participant Harness
    participant Worker
    participant Intelligence
    participant Agents
    participant Store
    User->>Web: 发起研究
    Web->>API: POST /research/runs
    API->>Harness: 创建 Run 与预算
    Harness-->>Web: Run ID + SSE
    Harness->>Worker: ResearchPlan
    Worker->>Intelligence: 检查 Coverage
    Intelligence->>Store: 下载、解析、索引
    Intelligence-->>Worker: ResearchSnapshot
    Worker->>Agents: 并行专业分析
    Agents->>Store: Artifact 与 Evidence
    Worker-->>Harness: DecisionMemo
    Harness-->>Web: run.completed
```

## 文档入库

```mermaid
sequenceDiagram
    participant Trigger
    participant Intelligence
    participant Source
    participant Parser
    participant Index
    Trigger->>Intelligence: DocumentRequirement
    Intelligence->>Source: 发现与下载
    Intelligence->>Intelligence: 授权、类型、哈希、实体、版本校验
    Intelligence->>Parser: 解析文档
    Parser->>Index: Block、Table、Fact、Embedding
    Index-->>Intelligence: 质量报告
    Intelligence-->>Trigger: CoverageManifest
```

## 订单执行

```mermaid
sequenceDiagram
    actor User
    participant API
    participant Risk
    participant Broker
    participant Gateway
    participant QMT
    API->>Risk: OrderProposal
    Risk-->>API: RiskAssessment
    User->>API: WebAuthn 批准订单哈希
    API->>Broker: 已批准不可变订单
    Broker->>Gateway: mTLS + Ed25519 信封
    Gateway->>Gateway: nonce、过期、幂等、二次风控
    Gateway->>QMT: 提交限价单
    QMT-->>Gateway: 订单与成交回报
    Gateway-->>Broker: 状态事件
    Broker-->>API: 对账结果
```

## 恢复语义

- 每个有副作用节点前写入意图与幂等键；
- 完成节点提交 Artifact 和 Checkpoint；
- 重启从最后成功 Checkpoint 恢复；
- 外部调用未知时先查询状态，不重放写操作；
- 取消任务停止新节点，已完成 Artifact 保留。
