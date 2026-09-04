# 主动金融情报

## 目标

根据研究和监控需要主动获得可使用、可追溯、可版本化的资料，并将文档、结构化数据和事件统一为 Evidence Ledger。

## 触发器

- 用户上传；
- ResearchPlan 产生 DocumentRequirement；
- 新公告、财报、监管或公司行动事件；
- 持仓、自选股和主题定时检查。

## Source Registry

每个 Source Adapter 声明：

- 支持的市场、文档类型和查询方式；
- 授权和使用策略；
- 可信等级；
- 限流、重试和缓存；
- 发布时间与内容时间字段；
- 下载和校验能力。

系统只从允许来源下载正文。未知来源资料必须经过实体、日期和内容验证后才能进入普通证据等级。

## Coverage Manifest

Research Director 根据任务生成 CoverageRequirement，Financial Intelligence Agent 对照已有文档和数据，输出可用、缺失、陈旧、冲突和完整度。

完成深度个股研究前默认检查三年年报、四个季度报告、一年重大公告、两年监管问询、六个月投资者关系记录、审计意见、业绩预告、同行和政策资料。

## 文档生命周期

```text
DISCOVERED
→ ACCESS_CHECKED
→ DOWNLOADED
→ VERIFIED
→ PARSED
→ EXTRACTED
→ INDEXED
→ QUALITY_CHECKED
→ READY
```

失败状态保留原因和可重试性。修订文件生成新版本，不覆盖原内容。

## 解析

Docling 解析版面、阅读顺序、章节、表格和公式；文本覆盖不足时使用 PaddleOCR。表格保存页面引用、HTML、行列 JSON、单元格坐标、表头、标准化值、单位、币种和期间。

## 检索

元数据过滤后执行 Dense 与 Sparse 召回、RRF 融合、Cross-Encoder Rerank 和 Evidence Consolidation。检索最多进行三轮；每轮必须由缺失证据类型驱动。

## Evidence Ledger

证据包含来源、文档、页码、Block、引用、实体、主张、数值、单位、期间、来源权威性、解析质量、检索分数和冲突状态。

Evidence 置信度由可观察信号计算，不能使用模型自报概率。原文高亮通过页码和 `bbox` 定位。

## 金融计算 DSL

允许百分比变化、比率、求和、差值、CAGR、NPV、IRR、XIRR 和情景计算。解释器校验输入 Evidence、单位、期间、币种、公式和舍入，并保存完整计算链。
