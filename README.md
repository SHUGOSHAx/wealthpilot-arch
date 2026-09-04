# WealthPilot Arch

WealthPilot 是一个以个人完整财务状况为基础、由多 Agent 专业团队持续提供资产管理、财务规划、证券研究、组合风控与交易协助的个人财富管理系统。

本仓库是 WealthPilot 的架构基线，只保存最终产品规划、系统架构、领域模型、公开契约、实施路线、验收标准和已采用的架构决策。生产代码、实验代码、真实用户数据、凭据、草稿和未采用设计不得进入本仓库。

## 架构基线

- 基线状态：`Architecture Baseline 1`
- 默认分支：`main`
- 目标实现周期：16 周
- 产品边界：单用户、本地数据优先、A 股优先、云端生成式模型、实盘逐单人工批准
- 实现仓库必须保存本仓库的 Git Commit 或 Tag，作为对应版本的架构依据

## 阅读顺序

1. [完整项目计划](PLAN.md)
2. [产品愿景](docs/product/product-vision.md)
3. [系统上下文](docs/architecture/system-context.md)
4. [运行时架构](docs/architecture/runtime-architecture.md)
5. [领域设计](docs/domains/wealth-management.md)
6. [领域 Artifact](docs/contracts/domain-artifacts.md)
7. [实施路线](docs/delivery/implementation-roadmap.md)
8. [最终决策记录](docs/decisions/0001-local-data-cloud-model.md)

## 文档导航

### 产品

- [产品愿景](docs/product/product-vision.md)
- [用户体验](docs/product/user-experience.md)
- [验收场景](docs/product/acceptance-scenarios.md)

### 架构

- [系统上下文](docs/architecture/system-context.md)
- [运行时架构](docs/architecture/runtime-architecture.md)
- [数据架构](docs/architecture/data-architecture.md)
- [Agent Harness](docs/architecture/agent-harness.md)
- [Model Gateway](docs/architecture/model-gateway.md)
- [安全与隐私](docs/architecture/security-privacy.md)
- [部署架构](docs/architecture/deployment.md)

### 领域

- [个人财富管理](docs/domains/wealth-management.md)
- [主动金融情报](docs/domains/financial-intelligence.md)
- [股票研究](docs/domains/equity-research.md)
- [组合与风险](docs/domains/portfolio-risk.md)
- [交易执行](docs/domains/trading-execution.md)
- [记忆与自进化](docs/domains/memory-evolution.md)

### 契约

- [领域 Artifact](docs/contracts/domain-artifacts.md)
- [API 契约](docs/contracts/api-contracts.md)
- [事件契约](docs/contracts/event-contracts.md)
- [错误契约](docs/contracts/error-contracts.md)

### 交付

- [实施路线](docs/delivery/implementation-roadmap.md)
- [测试与评测](docs/delivery/testing-evaluation.md)
- [演示与交付物](docs/delivery/demo-deliverables.md)

### 最终决策

- [本地数据与云模型](docs/decisions/0001-local-data-cloud-model.md)
- [双重记账](docs/decisions/0002-double-entry-ledger.md)
- [证据优先研究](docs/decisions/0003-evidence-first-research.md)
- [类型化 Agent Artifact](docs/decisions/0004-typed-agent-artifacts.md)
- [Point-in-Time 快照](docs/decisions/0005-point-in-time-snapshots.md)
- [实盘人工逐单批准](docs/decisions/0006-human-approved-live-trading.md)

## 变更规则

- `PLAN.md` 是产品和架构总纲，领域文档不得与其冲突。
- 新增或修改公开接口时，同步更新 `docs/contracts/`。
- 修改产品行为时，同步更新 `docs/product/acceptance-scenarios.md` 和测试标准。
- 修改已采用架构决策时，更新相应最终决策记录和受影响的架构文档。
- 只写最终行为和 non-obvious rationale，不保留讨论过程、候选方案或中间尝试。
- Mermaid 图中的组件、Artifact、API 和事件名称必须与正文一致。

## 许可证

本仓库使用 [Apache License 2.0](LICENSE)。
