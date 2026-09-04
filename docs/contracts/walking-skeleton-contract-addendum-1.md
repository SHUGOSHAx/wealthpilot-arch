# Walking Skeleton Contract Addendum 1

状态：`Accepted`  
契约版本：`1.0.0`  
适用基线：`architecture-baseline-1` / `66085f16bf479018dc0e139262eb283f57945bf3`  
规范 Schema：[walking-skeleton-contract-addendum-1.schema.json](schemas/walking-skeleton-contract-addendum-1.schema.json)

本文是 Architecture Baseline 1 的窄范围增补。正文规定跨组件行为与不变量，JSON Schema 是字段、类型、必填性、空值和枚举的机器可读规范源。两者冲突时停止实现并提交 Architecture Issue，不由实现方猜测。

## 1. Scope

本 Addendum 只服务 M00、M01 和首个 Walking Skeleton：上传一份合成通用银行 CSV，经异步 Run 解析与分类，生成不可变 `ImportPreview`；用户以预览版本和哈希确认提交后，系统写入平衡账本并生成 `FinancialSnapshot`，Web 通过可恢复 SSE 展示进度。

本 Addendum 冻结：

- `Money`、UTC 时间、哈希、Artifact metadata 和 Artifact reference；
- Import request、Import intent、Import status、`ImportPreview` 与 commit command；
- `RunState`、`TaskState`、进度、Checkpoint 摘要和失败状态；
- Walking Skeleton 使用的七类事件和 SSE replay；
- Walking Skeleton 页面读取的最小 `FinancialSnapshot`；
- 上述切片的 API、幂等和错误语义。

本 Addendum 不定义 M02–M12 的完整领域 Schema，不改变双重记账、隐私、云模型、人工审批或其他 Architecture Baseline 行为。

## 2. Contract Decisions

### 2.1 规范源与版本

- 公共类型的规范源是本 Addendum 附带的 JSON Schema Draft 2020-12 文件；实现仓库从同一版本生成或验证 Pydantic v2、TypeScript 和 OpenAPI Schema。
- 本切片所有顶层请求、响应、Artifact、Event envelope 和 Error response 使用 `schema_version = "1.0.0"`。
- `payload_version` 继续保留 Architecture Baseline 1 的事件字段，表示事件 Payload Schema；新增 `schema_version` 表示 Envelope Schema。M00/M01 两者均为 `1.0.0`。
- 所有对象默认拒绝未声明字段；只有经过兼容性审查发布的新 Schema 版本才能接受新增字段。

### 2.2 最小 API 面

| Method | Path | Request | Success | 用途 |
|---|---|---|---|---|
| `POST` | `/api/v1/imports` | multipart：`request` = `ImportRequest` JSON，`file` = CSV；Header `Idempotency-Key` | `202 ImportAccepted` | 创建 Import intent 与 Run |
| `GET` | `/api/v1/imports/{import_id}` | 无 | `200 ImportStatusResource` | 查询导入状态 |
| `GET` | `/api/v1/imports/{import_id}/preview` | 无 | `200 ImportPreview` | 读取当前不可变预览版本 |
| `POST` | `/api/v1/imports/{import_id}/commit` | `ImportCommitCommand`；Header `Idempotency-Key` | `202 ImportCommitAccepted` | 确认指定预览并异步入账 |
| `GET` | `/api/v1/runs/{run_id}` | 无 | `200 RunState` | 查询 Run、Task、Checkpoint 和失败状态 |
| `GET` | `/api/v1/runs/{run_id}/events` | Header `Last-Event-ID` 可选 | `200 text/event-stream` | 首个通用 Run 的事件流与重放 |
| `GET` | `/api/v1/wealth/snapshot?as_of=` | `as_of` 为 UTC 时间，可选 | `200 FinancialSnapshot` | 读取不晚于 `as_of` 的最新快照 |

通用 Run 查询和事件 URL 是对既有 `/api/v1` 的增量入口；它不改变研究领域已有的 `/research/runs/{run_id}` 入口。

### 2.3 幂等和提交边界

- 两个写端点都必须携带 `Idempotency-Key`。键作用域为本地用户、HTTP Method 和规范化 Path。
- 服务端请求指纹使用 RFC 8785 JSON Canonicalization Scheme。上传指纹额外包含服务端计算的文件 SHA-256；Commit 指纹包含 `import_id`、`command_id`、`preview_version` 和 `preview_hash`。
- 同一键与同一指纹返回同一逻辑结果：`import_id`、`run_id`、`command_id` 和已产生 Artifact 不变，不能重复产生 Posting 或事件副作用。
- 同一键与不同指纹返回 `IDEMPOTENCY_KEY_REUSED`。
- 即使客户端换用新键，相同 `target_account_id + source_type + file_hash` 也代表同一来源文件；系统返回已有 Import，不生成重复 Posting。
- Commit 必须同时匹配当前 `preview_version` 和 `preview_hash`。匹配失败不产生任何财务影响。
- JournalEntry、Posting、Import 状态、Checkpoint 和 outbox event 在同一数据库事务中提交。`import.committed` 不能早于平衡账本落库，`run.completed` 不能早于 `FinancialSnapshot` 持久化。
- 未 Commit 的 Import、预览、分类和模型输出不进入账本，不改变账户余额、资产、负债或净资产。

### 2.4 不可变性

- `ImportIntent` 在接收文件并计算哈希后不可变。
- `ImportPreview` 发布后不可变。任何用户确认或分类修订都创建递增的 `preview_version`、新 Artifact ID 和新哈希；旧版本保持可读。
- Commit command 只引用一个确定的预览版本；已成功提交的命令可重放响应但不可再次执行。
- `FinancialSnapshot` 发布后不可变；后续入账创建新 `snapshot_version`。
- Event append-only，已分配的 `sequence` 和 `event_id` 不得重用或改写。

## 3. Schemas

### 3.1 统一标量和值对象

| 类型 | 表示 | 约束 |
|---|---|---|
| `Uuid` | lowercase UUID string | 公共 ID 统一使用完整 UUID，不接受缩写 |
| `UtcDateTime` | RFC 3339 string | 必须带 `Z`；存储和协议不接受本地时区偏移；UI 转换为 Asia/Shanghai |
| `Sha256` | `sha256:<64 lowercase hex>` | 文件、预览和 Artifact 使用同一表示 |
| `DecimalString` | canonical base-10 string | 禁止指数、`+`、前导零、负零和无意义尾随零；领域层解析为 `Decimal` |
| `Money` | `{amount, currency}` | `amount: DecimalString`；本切片 `currency` 闭合枚举仅为 `CNY`；不同币种不得直接运算 |
| `schema_version` | SemVer string | 本切片固定 `1.0.0` |

M00/M01 不使用 JSON number 表示金额。确定性财务计算使用 `Decimal`，显示舍入与账本存储精度分离。

### 3.2 Artifact metadata

`ArtifactMetadata` 的所有字段必填：

| 字段 | 类型 | 语义 |
|---|---|---|
| `artifact_id` | `Uuid` | Artifact 不可变身份 |
| `artifact_type` | enum | `IMPORT_PREVIEW`、`FINANCIAL_SNAPSHOT` |
| `schema_version` | `SchemaVersion` | Artifact Schema 版本 |
| `created_at` | `UtcDateTime` | 持久化完成时间 |
| `created_by` | non-empty string | 稳定的 service/agent ID，不使用展示名 |
| `run_id` | `Uuid \| null` | 产生 Artifact 的 Run；无 Run 的确定性快照才允许 `null` |
| `content_hash` | `Sha256` | Artifact 规范内容哈希 |

`content_hash` 计算输入为包含 `schema_version` 的业务 Payload，经 RFC 8785 规范化后计算 SHA-256；计算时排除 `metadata.content_hash` 本身。`ArtifactReference` 固定包含 `artifact_id`、`artifact_type`、`schema_version` 和 `content_hash`，避免消费者只凭可变数据库行读取内容。

### 3.3 Import

#### `ImportRequest`

| 字段 | 类型 | 必填 | 约束 |
|---|---|---|---|
| `schema_version` | literal `1.0.0` | 是 | 请求版本 |
| `intent` | enum | 是 | 仅 `CREATE_LEDGER_ENTRIES_AND_FINANCIAL_SNAPSHOT` |
| `source_type` | enum | 是 | 本切片仅 `GENERIC_BANK_CSV` |
| `target_account_id` | `Uuid` | 是 | 已存在的合成银行账户 |
| `file_name` | string | 是 | 1–255 字符，仅作展示和审计，不参与路径拼接 |
| `as_of` | `UtcDateTime` | 是 | 导入业务截止时点 |

文件二进制只存在 multipart `file` part，服务端计算哈希、MIME 和大小；客户端不得声明可信文件哈希。

#### `ImportIntent`

持久化 intent 包含 `intent_id`、`import_id`、`run_id`、`intent`、`source_type`、`target_account_id`、`file_name`、`file_hash` 和 `requested_at`，全部必填且不可变。它记录副作用意图，使 Worker 重启时先恢复同一个 Import，而不是重新创建。

#### `ImportAccepted`

`POST /imports` 在两秒内返回：`schema_version`、`import_id`、`run_id`、`status`、`status_url`、`events_url` 和 `created_at`。`status` 只能是 `CREATED` 或 `PROCESSING`。

#### `ImportStatus`

闭合状态机：

```text
CREATED → PROCESSING → PREVIEW_READY → COMMITTING → COMMITTED
    └─────────────── non-terminal state ───────────────→ FAILED
```

`FAILED` 和 `COMMITTED` 是终态。Worker 进程重启、内部安全重试和 Checkpoint resume 不产生终态 `FAILED`；只有恢复策略耗尽后才发布失败。

`ImportStatusResource` 包含 `schema_version`、`import_id`、`run_id`、`status`、`preview`、`failure` 和 `updated_at`。`preview` 在 `PREVIEW_READY` 之前为 `null`；`failure` 仅在 `FAILED` 时非空。

#### `ImportPreview`

| 字段 | 类型 | 必填 | 约束 |
|---|---|---|---|
| `metadata` | `ArtifactMetadata` | 是 | `artifact_type = IMPORT_PREVIEW` |
| `import_id` | `Uuid` | 是 | 所属 Import |
| `preview_version` | integer ≥ 1 | 是 | 同一 Import 单调递增 |
| `preview_hash` | `Sha256` | 是 | 必须等于 `metadata.content_hash` |
| `target_account_id` | `Uuid` | 是 | 与 intent 一致 |
| `source_file_reference` | `SourceReference` | 是 | `source_type = IMPORT_FILE` 且带文件哈希 |
| `items` | `ImportPreviewItem[]` | 是 | 可为空数组，但空预览不能 Commit |
| `summary` | `ImportPreviewSummary` | 是 | 数量和金额由 items 确定性汇总 |
| `commit_eligible` | boolean | 是 | 只在无冲突、无待确认且至少一条 READY 时为 true |

`ImportPreviewItem` 字段：

| 字段 | 类型 | 语义 |
|---|---|---|
| `row_id` | `Uuid` | 同一文件行的稳定本地 ID |
| `source_row_number` | integer ≥ 1 | CSV 中的 1-based 行号 |
| `occurred_at` | `UtcDateTime` | 原始本地日期按导入适配器规则转换后的 UTC 时点 |
| `description` | string | P2 本地展示文本；不得原样发送云模型 |
| `direction` | enum | `INFLOW`、`OUTFLOW`，避免与总账 Debit/Credit 语义混用 |
| `amount` | `Money` | 必须大于零；方向由 `direction` 表示 |
| `category` | enum | `INCOME`、`EXPENSE`、`TRANSFER`、`UNCLASSIFIED` |
| `classification_source` | enum | `RULE`、`MODEL`、`USER`、`NONE` |
| `status` | enum | `READY`、`NEEDS_CONFIRMATION`、`SKIPPED`、`CONFLICT` |
| `source_reference` | `SourceReference` | `IMPORT_ROW`，locator 至少能回到 CSV 行号 |

模型分类只接收脱敏别名和必要事实，并以 `row_id` 返回候选；模型不得输出 JournalEntry 或 Posting。

`ImportPreviewSummary` 包含五个计数：`total_count`、`ready_count`、`needs_confirmation_count`、`skipped_count`、`conflict_count`，以及 `inflow_total`、`outflow_total`。五类行状态计数之和必须等于 `total_count`；两个金额只汇总非 `SKIPPED` 行。

#### `ImportCommitCommand`

全部字段必填：`schema_version`、`command_id`、`preview_version`、`preview_hash`、`requested_at`。Commit 前服务端依次验证 Import 非终态、预览版本、预览哈希、`commit_eligible`、账本平衡和幂等记录。

`ImportCommitAccepted` 包含 `schema_version`、`import_id`、`run_id`、`status`、`events_url` 和 `accepted_at`；`status` 为 `COMMITTING` 或已由同一命令完成的 `COMMITTED`。

### 3.4 Run、Task、进度与 Checkpoint

#### `RunState`

| 字段 | 类型 | 必填 | Null 语义 |
|---|---|---|---|
| `schema_version` | literal `1.0.0` | 是 | 不可空 |
| `run_id` | `Uuid` | 是 | 不可空 |
| `run_type` | enum | 是 | 仅 `IMPORT_TO_FINANCIAL_SNAPSHOT` |
| `status` | `RunStatus` | 是 | 不可空 |
| `progress` | `RunProgress` | 是 | 不可空 |
| `created_at` | `UtcDateTime` | 是 | 不可空 |
| `started_at` | `UtcDateTime \| null` | 是 | 尚未开始为 `null` |
| `updated_at` | `UtcDateTime` | 是 | 不可空 |
| `completed_at` | `UtcDateTime \| null` | 是 | 非 `COMPLETED` 为 `null` |
| `checkpoint` | `CheckpointSummary \| null` | 是 | 尚无持久 Checkpoint 为 `null` |
| `failure` | `FailureState \| null` | 是 | 非 `FAILED` 为 `null` |
| `tasks` | `TaskState[]` | 是 | 可为空，仅限 Run 创建事务完成前 |
| `artifacts` | `ArtifactReference[]` | 是 | 当前已发布 Artifact |

`RunStatus` 闭合枚举为 `CREATED`、`QUEUED`、`RUNNING`、`WAITING_FOR_INPUT`、`COMPLETED`、`FAILED`。正常流转为：

```text
CREATED → QUEUED → RUNNING ⇄ WAITING_FOR_INPUT → RUNNING → COMPLETED
                         non-terminal states ───────────→ FAILED
```

`COMPLETED` 和 `FAILED` 为终态。预览可供用户确认时 Run 使用 `WAITING_FOR_INPUT`，Import 使用 `PREVIEW_READY`。

#### `TaskState`

任务类型闭合枚举：

- `PARSE_IMPORT`；
- `CLASSIFY_IMPORT`；
- `BUILD_IMPORT_PREVIEW`；
- `COMMIT_LEDGER`；
- `BUILD_FINANCIAL_SNAPSHOT`。

状态闭合枚举：`PENDING`、`READY`、`RUNNING`、`WAITING_FOR_INPUT`、`SUCCEEDED`、`FAILED`、`SKIPPED`。

每个 Task 包含 `task_id`、`run_id`、`task_type`、`status`、`attempt`、三个时间字段、输入/输出 Artifact reference 和 `failure`。`started_at`、`completed_at`、`failure` 都是显式 nullable；`attempt` 首次执行为 1，尚未执行为 0。

#### `RunProgress`

字段为 `phase`、`percent`、`completed_tasks`、`total_tasks`、`current_task_id` 和 `updated_at`。`phase` 闭合枚举为：

```text
ACCEPTING_UPLOAD
PARSING
CLASSIFYING
BUILDING_PREVIEW
AWAITING_CONFIRMATION
COMMITTING_LEDGER
BUILDING_SNAPSHOT
COMPLETED
```

同一 Run 的 `percent` 不得回退；只有 `COMPLETED` 可为 100。`percent` 是 UI 进度，不作为业务提交依据；业务依据始终是状态、Artifact 和 Checkpoint。

#### Checkpoint

公共 `CheckpointSummary` 只暴露 `checkpoint_id`、单调递增的 `checkpoint_version`、`task_id`、`created_at` 和 `resumable`，不暴露内部 state blob 或 resume token。

- Checkpoint 在每个有副作用边界完成后，与 Artifact、领域写入和 outbox event 原子提交。
- Resume 只能从最新已提交 Checkpoint 开始；未提交的内存状态不算 Checkpoint。
- Resume 可以重做纯读取和确定性计算，但不得重复已经提交的 Posting 或 Event。
- Worker restart 属于同一 `run_id` 和 `task_id` 的新 `attempt`；不会重置 sequence、preview version 或 snapshot version。
- 外部可见 `run.failed` 表示内部 Retry/Resume 策略已结束。`FailureState.retryable` 只说明以显式恢复动作重新执行在技术上安全，不授权静默重复副作用。

### 3.5 FinancialSnapshot

本切片冻结 `FinancialSnapshot 1.0.0` 的最小投影：

| 字段 | 类型 | 必填 | 约束 |
|---|---|---|---|
| `metadata` | `ArtifactMetadata` | 是 | `artifact_type = FINANCIAL_SNAPSHOT` |
| `snapshot_version` | integer ≥ 1 | 是 | 同一财富底账单调递增 |
| `as_of` | `UtcDateTime` | 是 | 业务观察时点，不等同 `created_at` |
| `base_currency` | enum `CNY` | 是 | 本切片单币种 |
| `account_balances` | `AccountBalance[]` | 是 | 每项含账户、资产/负债分类、余额与来源 |
| `assets` | `BalanceSheetPosition[]` | 是 | 本切片 `position_type` 仅 `ACCOUNT_BALANCE` |
| `liabilities` | `BalanceSheetPosition[]` | 是 | 无负债时为空数组，不使用 `null` |
| `total_assets` | `Money` | 是 | 非负，等于 assets 之和 |
| `total_liabilities` | `Money` | 是 | 非负，等于 liabilities 之和 |
| `net_worth` | `Money` | 是 | `total_assets - total_liabilities` |
| `source_references` | `SourceReference[]` | 是 | 至少一项，可追踪到 Import、JournalEntry 或 Posting |
| `calculation_metadata` | `CalculationMetadata` | 是 | 计算版本、公式、舍入、账本截止时间和输入版本 |

`AccountBalance` 包含 `account_id`、`account_category`（`ASSET` 或 `LIABILITY`）、`balance` 和至少一项 `source_references`。余额和资产负债头寸用正数表达，所属类别决定净资产方向；负值不是负债的替代表达。

`CalculationMetadata` 固定包含：

- `calculation_version`：确定性计算器 SemVer；
- `formula = "NET_WORTH = TOTAL_ASSETS - TOTAL_LIABILITIES"`；
- `rounding_mode = "ROUND_HALF_EVEN"`；
- `base_currency = "CNY"`；
- `ledger_cutoff_at`；
- `input_artifact_ids`；
- `posting_count`。

同一 Snapshot 内全部 Money 必须为 CNY。`as_of` 不得早于纳入快照的交易业务时点，`ledger_cutoff_at` 决定可重复读取的账本边界。

## 4. Event Registry

### 4.1 Event envelope

所有字段必填：

| 字段 | 类型 | 规则 |
|---|---|---|
| `event_id` | `Uuid` | 全局唯一；客户端去重键 |
| `run_id` | `Uuid` | 事件所属 Run |
| `event_type` | closed enum | 本节七类事件之一 |
| `occurred_at` | `UtcDateTime` | 事件持久化的 UTC 时间 |
| `sequence` | integer ≥ 1 | 单 Run 连续、严格递增；首个事件为 1 |
| `schema_version` | literal `1.0.0` | Envelope Schema 版本 |
| `payload_version` | literal `1.0.0` | 当前 event type 的 Payload Schema 版本 |
| `payload` | typed object | 必须匹配 event type 对应 Schema |

事件 Payload 不包含 P2/P3 原文、模型隐藏思维链、Secret 或 Provider 原始响应。

### 4.2 Walking Skeleton 事件

| Event type | 必须 Payload | 发布条件 |
|---|---|---|
| `run.created` | `run_type`、`status=CREATED`、`created_at` | Run 与 outbox 原子创建；在 `202` 返回前可读取 |
| `run.started` | `status=RUNNING`、`started_at` | 首个 Task 开始 |
| `run.progress` | `status`、完整 `RunProgress` | phase、task 完成数或等待状态发生变化 |
| `import.preview_ready` | `import_id`、preview Artifact reference、`preview_version`、`preview_hash`、summary | 预览与 Checkpoint 已持久化，Run 进入等待确认 |
| `import.committed` | `import_id`、`committed_at`、`journal_entry_ids`、`posting_count` | 平衡分录和幂等记录已原子提交 |
| `run.completed` | `status=COMPLETED`、`completed_at`、result Artifact references | `FinancialSnapshot` 已持久化 |
| `run.failed` | `status=FAILED`、`FailureState` | 内部恢复策略结束，Run 进入终态 |

同一 Run 的正常最小顺序为：

```text
run.created
→ run.started
→ run.progress*
→ import.preview_ready
→ run.progress (AWAITING_CONFIRMATION)
→ run.progress (COMMITTING_LEDGER)
→ import.committed
→ run.progress (BUILDING_SNAPSHOT)
→ run.completed
```

`run.failed` 可在任一非终态后结束序列。`run.progress` 可重复，但内容不变时不得高频发布。

### 4.3 SSE 和 replay

- SSE frame 使用 `id: <event_id>`、`event: <event_type>`、`data: <完整 EventEnvelope JSON>`。
- 无 `Last-Event-ID` 时从 `sequence = 1` 重放，然后切换到 live stream。
- 有 `Last-Event-ID` 时，服务端先验证该 ID 属于 Path 中的 `run_id`，再发送其后所有 `sequence`，不重发 cursor 对应事件。
- M00/M01 不清理 Event store；终态 Run 的全部事件持续可重放。后续保留策略不能追溯删除本切片验收所需事件。
- 传输采用 at-least-once 语义。客户端必须按 `event_id` 去重，按 `sequence` 排序，并在发现 sequence gap 时用最后连续事件 ID 重新连接。
- 不存在、属于其他 Run 或无法定位 sequence 的 `Last-Event-ID` 返回 `SSE_CURSOR_INVALID`，不能从“当前最新事件”静默继续。
- SSE keep-alive 使用 comment frame，不分配 `event_id` 或 `sequence`，也不进入 Event Registry。

## 5. Error Codes

所有同步 API 错误使用 Baseline `ErrorResponse` 字段，并增加顶层 `schema_version`。字段为 `schema_version`、`error_code`、`message`、`trace_id`、`stage`、`retryable`、`partial_artifacts`、`checkpoint_status` 和显式 nullable 的 `recovery_action`。

| `error_code` | HTTP | `retryable` | 触发条件 |
|---|---:|---:|---|
| `VALIDATION_REQUEST_INVALID` | 400 | false | JSON、字段、枚举、时间或 Money 不符合 Schema |
| `VALIDATION_FILE_EMPTY` | 422 | false | 上传文件为空 |
| `VALIDATION_FILE_TYPE_UNSUPPORTED` | 422 | false | 不是本切片支持的通用银行 CSV |
| `IMPORT_NOT_FOUND` | 404 | false | Import ID 不存在 |
| `HARNESS_RUN_NOT_FOUND` | 404 | false | Run ID 不存在 |
| `IMPORT_PREVIEW_NOT_READY` | 409 | true | 预览尚未发布；恢复动作是等待事件后重试读取 |
| `IMPORT_PREVIEW_VERSION_MISMATCH` | 409 | false | Commit 引用的版本不是当前可提交版本 |
| `IMPORT_PREVIEW_HASH_MISMATCH` | 409 | false | Commit hash 与持久化 Artifact 不一致 |
| `IMPORT_PREVIEW_NOT_COMMIT_ELIGIBLE` | 422 | false | 预览为空、存在冲突或待确认项 |
| `IDEMPOTENCY_KEY_REUSED` | 409 | false | 同一 Idempotency-Key 对应不同请求指纹 |
| `IMPORT_ALREADY_COMMITTED` | 409 | false | 不同 command 尝试再次提交已提交 Import |
| `PRIVACY_REDACTION_FAILED` | 403 | false | P2 内容无法安全最小化，模型调用被阻断 |
| `MODEL_STRUCTURED_OUTPUT_INVALID` | 502 | false | 受限修复后仍不符合分类输出 Schema |
| `HARNESS_CHECKPOINT_UNAVAILABLE` | 503 | true | 无法安全持久化或读取所需 Checkpoint |
| `HARNESS_INVALID_STATE_TRANSITION` | 409 | false | 命令与当前 Run/Task 状态冲突 |
| `SSE_CURSOR_INVALID` | 409 | false | Last-Event-ID 不属于该 Run 或不可定位 |
| `LEDGER_UNBALANCED` | 500 | false | Commit 生成的借贷不平衡，事务回滚 |
| `INTERNAL_UNEXPECTED` | 500 | false | 未分类内部错误；不得暴露原异常或敏感内容 |

异步错误使用同一 `error_code` 注册表写入 `FailureState` 并通过 `run.failed` 发布。`retryable=true` 表示在幂等与 Checkpoint 约束下技术上可安全恢复，不表示客户端应自动重复提交写命令。

## 6. Compatibility Rules

- Schema 遵循 SemVer。Patch 只能修正文档且不得改变校验结果或业务语义。
- Minor 可以新增可选字段、新端点或新 event type；消费者必须忽略其所选 Schema minor version 中未使用的可选字段。
- 本 Addendum 的 enum 是闭合集合。新增 enum value、将 nullable 改为 non-null、增加必填字段，均可能破坏穷举消费者，必须提升 major version。
- 删除、重命名、改变字段类型、金额表示、时间语义、哈希算法、幂等作用域、状态转移或现有事件 Payload 语义必须提升 major version。
- 新版本不能原地改写已发布 Artifact 或 Event；通过新 Artifact、版本化 projection 或可审计迁移提供。
- API 服务至少支持实现仓库记录的本 Addendum major version；拒绝不支持的 major version时返回 `VALIDATION_REQUEST_INVALID`，不得猜测字段。
- Pydantic、JSON Schema、OpenAPI 和 TypeScript 类型必须由同一契约版本生成或在 CI 中做双向 Contract Test；手写副本不得成为第二规范源。
- 规范文件的 Git Commit 和 SHA-256 必须进入实现仓库的架构 metadata 与 CI 日志。

Implementation Repository 根目录必须保存 `ARCHITECTURE_BASELINE.json`：

```json
{
  "schema_version": "1.0.0",
  "architecture_repository": "https://github.com/SHUGOSHAx/wealthpilot-arch.git",
  "baseline_name": "Architecture Baseline 1",
  "baseline_tag": "architecture-baseline-1",
  "baseline_commit": "66085f16bf479018dc0e139262eb283f57945bf3",
  "contract_addenda": [
    {
      "name": "Walking Skeleton Contract Addendum 1",
      "schema_version": "1.0.0",
      "repository_path": "docs/contracts/walking-skeleton-contract-addendum-1.md",
      "schema_path": "docs/contracts/schemas/walking-skeleton-contract-addendum-1.schema.json",
      "git_tag": "walking-skeleton-contract-addendum-1",
      "schema_sha256": "5b22d0a3e668244896483132d795794687b4b90379228c5815c0f32a1f7a1577"
    }
  ],
  "recorded_at": "2026-09-04T09:17:35Z"
}
```

实现仓库还必须在首次提交中保存 `git rev-list -n 1 walking-skeleton-contract-addendum-1` 得到的完整 Addendum commit，字段名为 `contract_addenda[0].git_commit`。不允许使用分支名、缩写 SHA 或 `latest`。CI 验证两个 tag 解引用到固定 commit，并验证 Addendum Schema 文件哈希未漂移。

## 7. Remaining Architecture Issues

ARCH-ISSUE-002 对 M00/M01 Walking Skeleton 的阻塞部分在本 Addendum 中关闭。以下契约继续留在对应 Milestone，不得反向扩入本切片：

- M02：其他导入来源、字段映射、预览修订命令、取消、退款、转账、分期、冲销、对账以及完整 Ledger public Schema；
- M03：`FinancialSnapshot` 的现金流、预算、应急资金、目标保留、`investable_capital` 和 IPS projection；
- M04–M06：文档、Coverage、Evidence、RAG 和计算 DSL 的可执行 Schema；
- M07–M08：Research、Screening、Bull/Bear、Decision Memo 和通用 Agent task event；
- M09–M10：Portfolio、Backtest、Paper Order 和主动通知；
- M11–M12：Evolution、QMT、Live 审批和交易事件；
- Event store 的长期归档/清理策略，以及跨 major event replay；
- Model Gateway 全量 public request/response、成本和 Provider health Schema；Baseline 中已足够支撑 M01 内部实现的行为不在本 Addendum 扩写。

未来 Milestone 必须以 additive contract 或新的 major Schema 发布；不得修改本 Addendum 来使历史实现“看起来兼容”。

## 8. Implementation Authorization

### 8.1 Architecture Issue 状态

| Issue | 状态 | 结果 |
|---|---|---|
| `ARCH-ISSUE-001` | `CLOSED` | annotated Tag `architecture-baseline-1` 固定指向 `66085f16bf479018dc0e139262eb283f57945bf3` |
| `ARCH-ISSUE-002` | `CLOSED_FOR_M00_M01` | Import、Run/Task、Event/SSE、FinancialSnapshot 与错误的 Walking Skeleton 最小契约已冻结；后续 Milestone 范围仍未授权 |

### 8.2 Work Package Gate

| Work Package / Task | Gate 变化 | 授权边界 |
|---|---|---|
| `WP-0001 Repository & baseline bootstrap` | `BLOCKED_BY_ARCHITECTURE → READY_FOR_IMPLEMENTATION` | 写入并校验 `ARCHITECTURE_BASELINE.json`，不得移动 baseline tag |
| `WP-0002 Kernel & contract registry` | `BLOCKED_BY_ARCHITECTURE → READY_FOR_PLANNING` | 可规划 Money、时间、Artifact、Import、Run、Event、Error Schema 与 codegen/contract tests |
| `WP-0103 Harness core` 的 Run/Task/Checkpoint 子切片 | `BLOCKED_BY_ARCHITECTURE → READY_FOR_PLANNING` | 只实现本文状态、Checkpoint 和恢复语义 |
| `WP-0104 Async API, Worker, SSE & Walking Skeleton` | `BLOCKED_BY_ARCHITECTURE → READY_FOR_PLANNING` | 可按本文 API、事件、幂等和快照契约拆分并行任务 |
| Walking Skeleton Web contract client | `BLOCKED_BY_ARCHITECTURE → READY_FOR_PLANNING` | 可从规范 Schema 生成类型并实现上传、预览、进度、重连和快照页面 |
| Walking Skeleton Contract Test fixtures | `BLOCKED_BY_ARCHITECTURE → READY_FOR_IMPLEMENTATION` | 可直接建立合法/非法 payload、幂等、SSE replay、Checkpoint resume 和未确认无财务影响测试 |

`WP-0003`、`WP-0004`、`WP-0101` 和 `WP-0102` 未被 ARCH-ISSUE-002 阻塞，沿用 Engineering Handoff 的依赖和 Gate。Architecture Gate 不替代 Planner：标记为 `READY_FOR_PLANNING` 的 Work Package，必须由 Planner 给出 Task ID、依赖、允许目录、验收命令和 owner 后，具体 Developer Task 才能进入 `READY_FOR_IMPLEMENTATION`。

M02 及以后工作不因本 Addendum 获得实现授权。
