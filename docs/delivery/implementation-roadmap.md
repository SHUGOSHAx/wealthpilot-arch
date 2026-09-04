# 16 周实施路线

## 第 1 周：工程与领域基座

交付：Monorepo、Docker Compose、CI、Money/时间/证据/RunState/订单模型、PostgreSQL、Qdrant、Redis、类型检查、Secret Scan 和基础 Trace。

验收：一条命令启动基础服务；迁移和测试可重复执行；合成数据与真实数据路径隔离。

## 第 2 周：Model Gateway 与 Harness

交付：Provider Adapter、FAST/STANDARD/DEEP/CRITIC 路由、隐私检查、限流、重试、熔断、成本、Agent/Tool Registry、Checkpoint、SSE 和 FakeModelAdapter。

依赖：第 1 周领域基类和数据库。

验收：Provider 故障可恢复；结构化输出不合法时不进入领域层；P2/P3 Fixture 不进入请求和日志。

## 第 3–4 周：账本与个人资产

交付：双重记账、账户、ImportBatch、支付宝/微信/银行/信用卡/QMT Parser、去重、分类、冲销、对账、资产负债、预算和现金流页面。

验收：导入合成一年数据后得到准确净资产、收支、账户余额和预算；重复导入不新增分录。

## 第 5 周：财务规划

交付：财务目标、应急资金、贷款、保险覆盖、Monte Carlo、IPS、Personal CFO、Goal Planner 和 Asset Allocation Strategist。

验收：给定收入、支出和目标后生成资金缺口、储蓄计划、可投资资金和投资约束。

## 第 6–7 周：主动金融情报

交付：Source Registry、DocumentRequirement、Coverage Manifest、授权、下载、哈希、版本、Docling、PaddleOCR、Block/Table/Cell 和文档工作台。

验收：研究目标股票时自动补齐缺失年报和公告；重复与修订文件处理正确；低质量解析显式失败。

## 第 8 周：RAG、Evidence 与计算

交付：Dense、Sparse、RRF、Rerank、四级上下文、Evidence Ledger、冲突检测、原文高亮和金融计算 DSL。

验收：跨三份文档完成比较，答案具有页码、引用、公式、中间结果、单位和期间。

## 第 9–10 周：专业股票投研团队

交付：Research Director、Data Steward、Fundamental、Valuation、Industry、Quant、News/Macro、Bull、Bear、Evidence Auditor、Risk 和 Portfolio Manager；Research Snapshot、Contradiction Matrix、Decision Memo 和 Trace UI。

验收：对支持标的生成完整、可审计、带反方证据和失效条件的报告。

## 第 11 周：智能选股与同行比较

交付：ScreeningSpec、字段注册、Validator、参数化查询、硬条件、因子排名、Top-K 研究和同行比较矩阵。

验收：自然语言筛选与人工构造 Spec 结果一致；历史筛选不读取未来数据。

## 第 12 周：回测与组合

交付：A 股事件驱动回测、复权、分红、停牌、涨跌停、T+1、费用、Walk-forward、泄漏检测、组合风险、再平衡和目标仓位。

验收：Golden 策略结果稳定；所有数据满足 `available_at`；缺少风险输入时不输出具体仓位。

## 第 13 周：模拟交易与主动管家

交付：Paper Broker、Order Proposal、状态机、成交和对账、日报、周报、月报、持仓和 Decision Memo 失效条件监控。

验收：模拟订单端到端闭环；任务补跑一次；通知不重复且邮件不包含敏感明细。

## 第 14 周：记忆与自进化

交付：四类记忆、结果归因、Failure Classification、EvolutionCandidate、Champion–Challenger、人工审批、灰度和回滚。

验收：用户纠正生成单变量候选；安全和隐私回归失败时不能批准；未批准版本不生效。

## 第 15 周：QMT Gateway

交付：xtquant Adapter、mTLS、Ed25519、WebAuthn、nonce、过期、幂等、二次风控、回报、对账、Kill Switch 和只读降级。

验收：断网、重复请求和重启不产生重复订单；无有效批准无法提交 Live 订单。

## 第 16 周：稳定性与公开交付

交付：故障注入、性能、安全、长时间运行、合成 Demo、README、架构与安全文档、Eval 报告、演示视频、简历和面试材料。

验收：完成最终演示脚本；公开仓库 Secret Scan 为零；一键 Demo 可在全新环境启动。
