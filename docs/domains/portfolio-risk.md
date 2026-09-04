# 投资组合与风险

## 组合模型

`PortfolioSnapshot` 保存账户现金、持仓批次、价格、成本、可卖数量、资产类别、行业、市场、因子暴露、净值和数据截止时间。

组合计算覆盖：

- 收益、波动和最大回撤；
- 单标的、行业、市场和资产类别集中度；
- Beta、相关性和风险贡献；
- 流动性和预计交易成本；
- 目标配置偏离；
- 现金和目标资金覆盖。

## 回测

事件驱动引擎按交易日处理信号、订单、成交、公司行动和组合估值。支持复权、分红、停牌、涨跌停、最小交易单位、T+1、费用、印花税、滑点、基准和定期再平衡。

所有输入按 `available_at` 过滤。Universe 使用历史成分，避免使用当前成分回测历史。

输出 CAGR、Alpha、Sharpe、Sortino、Calmar、最大回撤、波动、Beta、Turnover、交易成本、行业/因子暴露、市场阶段表现和参数稳定性。

## 目标仓位

目标仓位由确定性函数计算：

```text
min(
  investable_capital,
  cash_reserve_headroom,
  single_security_limit,
  industry_headroom,
  equity_headroom,
  risk_budget_limit,
  liquidity_limit,
  maximum_loss_limit
)
```

任一必要输入缺失时不生成具体仓位。

## 硬风控

- 最低现金储备；
- 可投资资金；
- 单股、行业和权益上限；
- 单笔和单日额度；
- 最大回撤锁定；
- 禁止标的；
- ST、停牌、退市和异常状态；
- 数据最大延迟；
- 重大公告和财报事件；
- 价格偏离和滑点；
- 账户资金和可卖数量；
- Paper/Live 环境隔离。

Risk Manager 可以解释规则结果，但不能修改规则值。硬风控失败生成 `risk.blocked` 事件，并阻止 OrderProposal。

## 再平衡

再平衡由配置偏离、风险预算、目标变化、现金流或重大事件触发。建议必须同时显示调整前后暴露、预计成本、税费假设、流动性影响和未执行后果。
