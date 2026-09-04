# 测试与评测

## 测试层级

- Unit：值对象、计算、规则、状态机和解析器；
- Contract：Provider、Source、Tool、Broker、API、Event 和 Artifact；
- Integration：数据库、队列、向量库、文档、研究和交易；
- Replay：固定 Snapshot 上的 Agent 和检索回放；
- E2E：浏览器到 Paper/QMT；
- Security：隐私、Prompt Injection、Secret 和签名；
- Fault Injection：超时、限流、断网、重启和未知状态。

## 账本与财务

- Golden Ledger 100% 借贷平衡；
- 支付宝、微信、银行、信用卡和 QMT Fixture 解析成功；
- 重复导入不产生重复 Posting；
- 转账不计收入或支出；
- 冲销保持完整审计；
- Money、贷款、IRR、XIRR、NPV、CAGR 和现金流 Golden Tests 100% 通过；
- Property-based Tests 覆盖任意金额和分录组合。

## 文档与 RAG

- 原生 PDF、扫描 PDF、跨页表格、脚注和修订报告；
- 哈希去重和版本链；
- 文档实体、报告期、单位和币种；
- Evidence Recall@10 ≥90%；
- 引用页码准确率 ≥95%；
- 单位、期间和口径对齐 ≥95%；
- 重要主张证据覆盖 ≥95%；
- FinanceBench 答案准确率目标 ≥70%；
- FinQA Execution Accuracy 目标 ≥75%；
- Prompt Injection 触发 Tool 数量为 0。

## Screener 与 Research

- 自然语言到 ScreeningSpec Golden Cases；
- 非注册字段和运算符拒绝；
- 参数化查询结果与人工基准一致率 100%；
- 历史 Universe 和 `available_at`；
- Bull/Bear 无 Evidence 主张拒绝；
- 错误公司、期间、单位和引用检测；
- 资料不足时不产生具体订单建议；
- 同一 Snapshot Replay 的结构化事实保持一致。

## Harness 与 Model Gateway

- Token、成本、步骤和 deadline；
- 429、5xx、超时、熔断和降级；
- Checkpoint 重启恢复；
- 副作用幂等；
- Schema 修复与失败；
- 简单查询 P95 <2 秒；
- 复杂任务 2 秒内产生首个事件；
- 上下文 Token 相比全文基线减少 ≥50%，质量下降不超过 2 个百分点；
- P2/P3 原文泄漏为 0。

## 回测与风险

- 复权、分红、停牌、涨跌停、T+1、费用和滑点；
- 未来数据泄漏为 0；
- 历史成分避免生存者偏差；
- Walk-forward 划分；
- 所有仓位和硬风控 Golden Tests 100% 通过；
- 缺少必要输入时输出阻断而非默认值。

## 交易与安全

- Paper 与 Live 账户隔离；
- WebAuthn、签名、nonce、过期和幂等；
- 修改 Proposal 后批准失效；
- 未知状态不自动重发；
- 重复请求不产生重复订单；
- Kill Switch 阻断新订单；
- Broker 断线后只读降级；
- Secret Scan 为 0。

## 自进化

- 候选一次只改一个 target type；
- 回放、Walk-forward、安全、隐私和成本门禁；
- 未批准候选不进入生产；
- 灰度回归触发回滚；
- 回滚恢复 Champion 版本和审计链。
