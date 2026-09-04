# 股票研究

## 任务类型

- `SINGLE_STOCK_RESEARCH`；
- `PEER_COMPARISON`；
- `NATURAL_LANGUAGE_SCREENING`；
- `PORTFOLIO_DIAGNOSIS`；
- `EVENT_IMPACT_ANALYSIS`；
- `WATCHLIST_MONITORING`；
- `TRADE_DECISION_SUPPORT`。

## 研究入口

Research Director 解析标的、截止时间、研究深度、同业规则、必需指标、资料要求、宏观主题、参与 Agent 和预算，生成 `ResearchPlan`。Financial Intelligence Agent 完成 Coverage，Data Steward 校验数据质量和时点并冻结 `ResearchSnapshot`，随后才能启动专业分析。

## 并行分析

- Fundamental：商业模式、财务质量、资本效率和治理；
- Valuation：相对估值、DCF、DDM、分部和情景；
- Industry：产业链、周期、供需、竞争和政策；
- Technical/Quant：趋势、波动、因子、流动性和回测；
- News/Macro：公司事件、行业政策、利率、通胀、汇率和商品。

所有报告使用结构化字段并引用 Evidence ID。

## 同行比较

同行由注册行业分类、主营收入相似度和规模约束确定。比较前统一会计期间、币种、单位、合并口径和一次性项目。缺少可比口径的指标显示 `NOT_COMPARABLE`。

## 自然语言选股

模型将用户条件转换为 `ScreeningSpec`。Validator 只允许注册字段、运算符、窗口和事件类型。执行器生成参数化查询，不执行模型文本中的 SQL。

筛选顺序为 Universe、硬条件、事件排除、因子排名、候选截断和 Top-K 研究。历史筛选使用 `available_at` 控制数据时点。

## Bull/Bear

双方独立输出主张、Evidence ID、催化剂、验证条件、风险和待查事实。随后各自质询对方三项核心主张。Research Director 只批准一次事实补充循环，最终生成 `ContradictionMatrix`。

## Evidence Audit

检查：

- 引用存在且支持主张；
- 数值、单位和期间一致；
- 来源在研究截止时间可用；
- 实体没有混淆；
- 推导使用确定性计算；
- 冲突和缺失已经显示。

审计失败的主张不能进入高置信度 Decision Memo。

## Decision Memo

Portfolio Manager 综合研究、个人财务和组合风险，输出评级、置信度、覆盖度、适配性、逻辑、反方证据、公允价值、情景、仓位、入场、退出、失效、风险、证据和版本。
