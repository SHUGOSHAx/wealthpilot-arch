# 领域 Artifact 契约

M00/M01 Walking Skeleton 的 `ImportPreview`、`FinancialSnapshot`、统一 Money 与 Artifact metadata 由 [Walking Skeleton Contract Addendum 1](walking-skeleton-contract-addendum-1.md) 冻结为可执行契约。

## 通用规则

所有 Artifact 包含：

```python
artifact_id: UUID
schema_version: str
created_at: datetime
created_by: str
run_id: UUID | None
```

- 时间使用 UTC ISO 8601；
- 金额和小数以字符串传输并解析为 `Decimal`；
- 已发布 Artifact 不可变；
- Schema 采用语义版本；
- 持久化 Artifact 保存内容哈希。

除本文件定义的跨领域 Artifact 外，`WealthManagementPlan`、`ImportPreview`、`ReconciliationReport`、`GoalPlan`、`AssetAllocationPlan`、`ProtectionAssessment`、`DocumentEvidenceReport`、`FundamentalReport`、`ValuationReport`、`IndustryComparisonReport`、`QuantReport`、`EventAndMacroReport`、`EvidenceAudit`、`ContradictionMatrix`、`ExecutionReport` 和 `PerformanceAttribution` 使用相同通用字段、版本和不可变规则，其业务字段由对应领域文档约束。

## Money

```python
class Money(BaseModel):
    amount: Decimal
    currency: str
```

- `currency` 使用 ISO 4217；
- 不同币种不能直接相加；
- 创建者：所有确定性财务服务；
- 消费者：全部涉及金额的领域；
- 持久化：作为值对象嵌入父 Artifact。

## FinancialSnapshot

字段：`as_of`、`accounts`、`assets`、`liabilities`、`net_worth`、`liquid_assets`、`monthly_expenses`、`emergency_fund_months`、`reserved_goals`、`investable_capital`、`data_quality`。

- 创建者：Personal CFO；
- 消费者：Asset Allocation、Risk、Portfolio Manager；
- 不变量：资产减负债等于净资产；可投资资金不能包含已保留目标资金；
- 持久化：不可变快照。

## InvestmentPolicyStatement

字段：`effective_from`、`minimum_cash_reserve`、`investable_capital_limit`、`target_allocation`、`equity_limit`、`single_security_limit`、`industry_limit`、`maximum_drawdown`、`allowed_instruments`、`prohibited_symbols`、`rebalance_rules`。

- 创建者：用户与 Personal CFO；
- 消费者：Asset Allocation、Risk、Portfolio Manager；
- 不变量：配置权重合法，禁止标的优先于其他规则；
- 持久化：版本化，旧版本只读。

## ResearchPlan

字段：`task_type`、`symbols`、`as_of`、`required_documents`、`required_market_data`、`required_metrics`、`peer_selection_policy`、`macro_topics`、`agent_tasks`、`run_budget`。

- 创建者：Research Director；
- 消费者：Data Steward、Harness；
- 不变量：标的完成身份解析，任务有截止时间和预算；
- 持久化：随 Run 保存。

## CoverageManifest

字段：`symbol`、`as_of`、`requirements`、`available_items`、`missing_items`、`stale_items`、`conflicts`、`completeness_score`。

- 创建者：Financial Intelligence Agent；
- 消费者：Research Director、Data Steward、UI；
- 不变量：每个 Requirement 恰好归入一个结果状态；完整度来源于显式 Requirement；
- 持久化：按检查时点保存。

## ResearchSnapshot

字段：`as_of`、`symbols`、`market_data_versions`、`financial_data_versions`、`document_versions`、`news_event_ids`、`macro_series_versions`、`peer_universe`、`financial_snapshot_id`、`portfolio_snapshot_id`、`model_versions`、`prompt_versions`、`tool_versions`。

- 创建者：Data Steward；
- 消费者：全部 Research Agent、Backtest；
- 不变量：所有输入在 `as_of` 时可用；引用版本存在且不可变；
- 持久化：永久保存，禁止原地修改。

## EvidenceFact

字段：`entity`、`claim`、`value`、`unit`、`currency`、`period`、`source_id`、`document_id`、`page`、`block_id`、`quote`、`source_authority`、`extraction_quality`、`retrieval_score`、`contradiction_status`。

- 创建者：Document/Market/News Tools 和 Evidence Consolidator；
- 消费者：所有分析 Agent、Evidence Auditor、Decision Memo；
- 不变量：文档证据可定位原文；结构化数据可定位数据版本；数值包含单位和期间；
- 持久化：Evidence Ledger append-only。

## ScreeningSpec

字段：`universe`、`as_of`、`hard_filters`、`ranking_factors`、`event_exclusions`、`missing_value_policy`、`max_candidates`、`research_top_k`。

- 创建者：Screener Interpreter；
- 消费者：Validator、Screener Engine、UI；
- 不变量：字段、运算符、单位、窗口和事件类型已注册；`research_top_k <= max_candidates`；
- 持久化：与筛选结果一同保存。

## BullCase 与 BearCase

字段：`symbol`、`claims`、`evidence_ids`、`catalysts_or_risks`、`validation_conditions`、`open_questions`、`response_to_opponent`。

- 创建者：Bull/Bear Researcher；
- 消费者：Research Director、Evidence Auditor、Portfolio Manager；
- 不变量：每项实质主张至少引用一条 Evidence；
- 持久化：随 Research Run 保存。

## RiskAssessment

字段：`financial_constraints`、`portfolio_constraints`、`instrument_constraints`、`market_constraints`、`order_constraints`、`blocking_rules`、`warnings`、`risk_check_version`、`passed`。

- 创建者：确定性 Risk Engine，Risk Manager 生成解释；
- 消费者：Portfolio Manager、Execution Steward；
- 不变量：存在 blocking rule 时 `passed=false`；
- 持久化：绑定 Decision Memo 或 Order Proposal。

## DecisionMemo

字段：`symbol`、`as_of`、`research_snapshot_id`、`ips_version`、`rating`、`confidence`、`coverage_score`、`suitability`、`thesis`、`bear_case`、`fair_value_range`、`scenarios`、`target_weight`、`max_position_value`、`entry_conditions`、`exit_conditions`、`invalidation_conditions`、`risks`、`evidence_ids`、`warnings`、`model_versions`。

- 创建者：Portfolio Manager；
- 消费者：用户、Monitor、Execution Steward、Evolution；
- 不变量：结论引用 Research Snapshot；重要主张引用有效 Evidence；具体仓位需要通过 RiskAssessment；
- 持久化：不可变并永久审计。

## OrderProposal

字段：`account_id`、`symbol`、`side`、`quantity`、`order_type`、`limit_price`、`max_slippage_bps`、`expires_at`、`decision_memo_id`、`risk_check_version`、`idempotency_key`、`environment`。

- 创建者：Execution Steward；
- 消费者：Approval Service、Paper Broker、QMT Gateway；
- 不变量：只允许 `LIMIT`；数量符合交易单位；Proposal 未过期；Decision Memo 和风控存在；
- 持久化：不可变，状态由独立事件表示。

## EvolutionCandidate

字段：`target_type`、`baseline_version`、`candidate_version`、`change_summary`、`expected_metric`、`evaluation_results`、`security_results`、`cost_results`、`approval_status`、`rollout_scope`、`rollback_version`。

- 创建者：Evolution Agent；
- 消费者：Evaluation Harness、用户审批、Version Registry；
- 不变量：一次只改变一个 target type；安全和隐私回归必须通过；
- 持久化：永久保存候选、审批和部署结果。
