# 个人财富管理

## 领域目标

维护可对账的个人财务事实，形成现金流、目标、投资政策和可投资资金，为所有投资建议提供约束。

## 双重记账

`JournalEntry` 包含两个或以上 `Posting`，借方总额等于贷方总额。导入数据先进入 `ImportBatch` 预览，确认后生成分录。已提交分录只能通过冲销修正。

导入适配器首批支持支付宝、微信、通用银行 CSV/Excel、信用卡、QMT 导出、OFX/QFX 和自定义列映射。

导入流程：

```text
上传
→ 文件和哈希检查
→ Parser 识别
→ 字段映射
→ 去重、退款、分期和转账识别
→ 规则分类
→ 脱敏模型分类候选
→ 用户确认
→ 平衡分录
→ 对账
```

## 资产负债

资产覆盖现金、存款、股票、ETF、基金、债券、保险现金价值、房产和其他资产。负债覆盖信用卡、消费贷、房贷和其他负债。

估值保存来源、估值日、取得时间和质量。缺少近期估值的非流动资产必须显示陈旧状态，不能默认为当前价值。

## 财务目标

`FinancialGoal` 包含目标金额、目标日、优先级、最低成功概率、允许资金来源和流动性要求。Goal Planner 计算每月储蓄、预计缺口、不同收益/通胀情景和 Monte Carlo 成功率。

目标资金在 Investable Capital 计算中先于证券投资保留。

## Investment Policy Statement

IPS 保存：

- 最低现金储备；
- 可投资资金上限；
- 目标资产配置；
- 权益、单股和行业上限；
- 投资期限和最大回撤；
- 允许品种和禁止标的；
- 再平衡触发条件；
- 大额支出和目标资金约束。

IPS 版本化。每份 Decision Memo 记录使用的 IPS 版本。

## 确定性计算

财务引擎负责净资产、收支、储蓄率、负债率、流动比率、应急资金、预算、现金缺口、贷款摊销、提前还款、IRR、XIRR、NPV、CAGR、目标成功率、保险缺口和可投资资金。模型只负责解释和生成问题。

## Agent 产物

- Account Steward：`ImportPreview`、`ReconciliationReport`；
- Personal CFO：`FinancialSnapshot`、`InvestableCapitalAssessment`；
- Goal Planner：`GoalPlan`、`GoalConflictReport`；
- Asset Allocation Strategist：`AssetAllocationPlan`；
- Insurance and Debt Analyst：`ProtectionAssessment`、`DebtOptimizationPlan`。
