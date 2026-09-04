# 记忆与自进化

## 记忆类型

| 类型 | 内容 |
|---|---|
| Semantic | 用户偏好、目标、IPS 和稳定事实 |
| Episodic | 历史问题、研究、决策、交易和反馈 |
| Procedural | 任务路由、工具使用和研究方法 |
| Evidence | 已验证且有来源的金融事实 |

每条记忆保存来源、作用域、创建时间、有效期、置信度、撤销状态、关联 Run 和版本。过期或被撤销记忆不进入新上下文。

## 结果归因

交易和研究结束后分别计算：

- 原始收益；
- 相对基准和行业收益；
- 市场 Beta；
- 投资逻辑是否兑现；
- 数据和证据是否充分；
- 入场与退出执行；
- 风险规则贡献；
- 模型与流程错误。

只看最终涨跌不能判断研究质量，因此 Evolution 使用多维归因而非单一收益标签。

## 候选对象

- Prompt；
- Tool Description；
- Query Rewrite；
- Source Ranking；
- Chunk、Top-K 和 Rerank 参数；
- Agent 路由；
- 流水分类规则；
- Context Compression；
- Procedural Skill。

候选一次只改变一类变量，并记录基线版本和预期改善指标。

## 评测门禁

```text
用户反馈 / 失败 Trace / 结果归因
→ Failure Classification
→ Root Cause
→ EvolutionCandidate
→ 金融问答回放
→ 文档检索回放
→ 历史研究回放
→ Walk-forward
→ 隐私、安全、成本回归
→ Champion–Challenger
→ 人工批准
→ 灰度
→ 保留或回滚
```

候选必须在目标指标上改善，且不能降低安全、隐私、引用完整性和硬约束遵从率。

## 禁止变更

Evolution Agent 不得修改风险上限、审批、Broker 权限、密钥、实盘开关、数据授权策略或生产代码发布流程。

## 回滚

每次启用记录 Candidate、批准人、版本、灰度范围和开始时间。监控触发回归阈值时自动停用候选，恢复 Champion，并保留完整审计记录。
