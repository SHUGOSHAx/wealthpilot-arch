# 演示与交付物

## 最终演示脚本

### 1. 建立财富底账

导入合成支付宝、微信、银行卡、信用卡和券商文件，展示来源识别、去重、分类预览、平衡分录和账户对账。

### 2. 生成资产负债与财务目标

展示净资产、现金流、应急资金、教育目标和 Investment Policy Statement。

### 3. 自然语言条件选股

输入包含增长、质量、估值和事件排除的筛选条件，展示 ScreeningSpec、条件解释、确定性执行和候选结果。

### 4. 自动补齐资料

选择候选股票，展示 Coverage Manifest、资料发现、下载、版本校验、PDF 解析和 Evidence 添加事件。

### 5. Multi-Agent 投研

展示 Fundamental、Valuation、Industry、Quant、News/Macro 并行任务，随后展示 Bull/Bear 质询、矛盾矩阵和 Evidence Audit。

### 6. 个性化决策

展示 Personal CFO 保留应急和目标资金，Risk Manager 计算上限，Portfolio Manager 输出 Decision Memo、入场/退出/失效条件和目标仓位。

### 7. 模拟交易

创建 Order Proposal，展示硬风控、用户确认、Paper 成交、持仓和账本对账。

### 8. 监控与进化

触发一条公告或逻辑失效事件，展示通知、结果归因、用户纠正、EvolutionCandidate、Champion–Challenger 和回滚。

## 演示数据

- 一年合成个人流水；
- 现金、信用卡、贷款、基金和股票账户；
- 教育和应急资金目标；
- 两到三家同行公司三年财报与公告；
- 固定行情、财务、新闻和宏观 Snapshot；
- Paper 订单与成交历史；
- 一组正确和错误 Agent 输出 Fixture。

## 交付物

- 可运行 Web/PWA；
- Docker Compose；
- Windows QMT Gateway；
- OpenAPI；
- 架构、安全和隐私说明；
- Evaluation Report；
- 合成数据和一键 Demo；
- 3–5 分钟演示视频；
- README 架构图和关键指标；
- 秋招简历项目描述；
- 面试讲解提纲和故障案例。

## 演示验收

- 全流程不依赖真实账户或私有数据；
- 每个结论可跳转到证据；
- 每个计算可查看公式；
- Trace 能显示 Agent、Tool、Token、成本和延迟；
- 个人约束能够改变仓位或阻断订单；
- 未经批准不能进入 Live；
- 自进化候选未经批准不生效。
