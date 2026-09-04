# 0005：研究与回测使用 Point-in-Time 快照

## 状态

Accepted

## 适用范围

股票研究、自然语言选股、同行比较、回测、结果归因和自进化。

## 最终决定

每次研究创建不可变 `ResearchSnapshot`，绑定行情、财务、文档、新闻、宏观、同行、个人财务、组合、模型、Prompt 和 Tool 版本。历史任务只能使用 `available_at` 不晚于研究时点的数据。

## Non-obvious rationale

报告期、发布时间和系统获取时间不是同一概念。只按报告期过滤会让未来披露、修订财报或当前新闻进入历史决策，造成不可见的未来数据泄漏。

## 实现约束

- 数据必须保存 `as_of`、`available_at` 和 `fetched_at`；
- 修订资料创建新版本；
- 历史 Universe 使用历史成分；
- Decision Memo、Backtest 和 Attribution 保存 Snapshot ID。

## 验收方式

- 未来数据泄漏测试 100% 通过；
- 历史研究重放仍引用原版本；
- 修订报告不改变已有 Snapshot。
