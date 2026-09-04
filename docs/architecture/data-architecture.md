# 数据架构

## 数据平面

### 个人财富与账本

PostgreSQL 保存账户、JournalEntry、Posting、资产、负债、目标、IPS、持仓批次、组合快照和审批。关键业务事件 append-only。

### 市场、财务与宏观时序

PostgreSQL/TimescaleDB 或分区 Parquet 保存 OHLCV、复权因子、公司行动、财务事实、行业成员和宏观序列。数据包含 `source`、`as_of`、`available_at`、`fetched_at`、`adjustment` 和 `quality_status`。

### 文档、表格与向量

- 加密对象存储：原始文件；
- PostgreSQL：文档、版本、Block、Table、Cell 和事实；
- Qdrant：Dense 向量；
- PostgreSQL：Sparse/BM25；
- Evidence Ledger：标准化证据和冲突状态。

### Agent、Trace 与版本

保存 Run、Task、Checkpoint、Artifact、ToolCall、ModelCall、PromptVersion、DataSnapshot、EvaluationRun 和 EvolutionCandidate。

## 时间语义

- 系统时间存 UTC，界面显示 Asia/Shanghai；
- `as_of` 表示业务观察时点；
- `available_at` 表示信息最早可被研究使用的时点；
- `fetched_at` 表示系统获取时点；
- 回测按 `available_at <= simulation_time` 过滤。

`available_at` 与报告期分离，用于阻止尚未披露的财务数据进入历史决策。

## 版本与不可变性

- 原始文件按 SHA-256 去重；
- 修订文件生成新 `DocumentVersion`；
- 已提交 JournalEntry 通过冲销修改；
- ResearchSnapshot、DecisionMemo 和 OrderProposal 不可变；
- Artifact 使用 `schema_version`；
- 迁移保留原字段语义和审计链。

## 数据质量

质量状态：`VALID`、`PARTIAL`、`STALE`、`CONFLICTED`、`REJECTED`。

进入投资结论前检查：

- 实体一致性；
- 报告期和币种；
- 单位与会计口径；
- 来源权威性；
- 时点可用性；
- 修订状态；
- 多来源冲突。

## 备份

- PostgreSQL 进行加密逻辑备份；
- 对象存储按哈希清单备份；
- Qdrant 可由原始文档和 Embedding 配置重建；
- Secret Store 和 Broker 凭据不进入普通备份；
- 恢复后执行账本平衡、对象哈希和快照引用完整性检查。
