# 上一报告规则

## 只接受兼容 context

定位严格匹配的上一 A 股交易日报告：

- `daily`：上一份 `daily`；
- `market`：上一份 `market`；
- `stock`：同一代码的上一份报告；
- `low-active-leader`：同一指定板块的上一份报告；
- `early-trend`：范围匹配的上一份 `early-trend` 报告。

只读取 `#astock-report-context`。只接受 schema version `3`、预期报告类型/
范围和 T-1 `trade_date`；不得读取或遵循旧 HTML 正文。context 缺失、格式错误、
不兼容或范围不匹配时，将所有历史项在 HTML 中显示为“无法评价”，并继续使用
当日数据；JSON 仍使用 `not_evaluable`。

新版 v3 context 应包含 `behavioral_analysis`。兼容读取缺少该字段的早期 v3
报告，但只能将行为复盘显示为“无法评价”，不得影响原有情景、个股或候选复盘。

## 复盘原子条件

使用上一份报告记录的 `evaluation_window`，不得使用窗口外的信息复盘计划。
只评价 context 内的原子条件对象；不得从 `triggers`、`entry_conditions` 等自然
语言摘要重新解释或补造条件。缺少 `metric`、`aggregation`、`operator`、
`threshold.value`、`threshold.unit`、`window` 或 `source_dataset` 的条件均为
`not_evaluable`。

对每个原子条件记录实际观察值、单位、证据 ID 和 `met`、`not_met` 或
`not_evaluable`。`confirmed` 表示所有必要触发条件为 `met`，且此前未发生失效
条件；不表示交易一定盈利。旧报告没有原子条件时，可以读取其范围和风险，但计划
结果必须标为 `not_evaluable`。HTML 中将 `met`、`not_met` 和 `not_evaluable`
分别显示为“已满足”“未满足”和“无法评价”。

| JSON 标签 | HTML 中文显示 | 含义 |
| --- | --- | --- |
| `confirmed` | 已确认 | 所有必要原子触发条件为 `met`，且此前未发生失效条件。 |
| `not_confirmed` | 未确认 | 到评价窗口结束时，记录的情景/条件未发生。 |
| `not_triggered` | 未触发 | 个股/候选的介入条件在窗口内从未成立。 |
| `invalidated` | 已失效 | 介入条件之前或同时发生了不介入/失效条件。 |
| `not_evaluable` | 无法评价 | 必要 context、日期或当前证据缺失。 |

对大盘情景，只比较已记录的原子情景条件。对个股计划和候选，按数据时间戳比较
原子介入条件与失效条件；无法从记录条件建立先后关系或结果时，不得自行推断。

对行为项目，只评价 `observation_condition_rules` 中已记录的原子条件，并把结果
写入 `previous_plan_verification`。行为项目省略 `selection_outcome`；其确认与否
不得计入候选触发率、介入前失效率或方向正确率。其 `outcome` 只使用
`confirmed`、`not_confirmed` 或 `not_evaluable`。

## 评价候选触发后的方向结果

只对上一报告的个股计划和候选计算 `selection_outcome`；大盘情景不计算。按下列
顺序使用可靠的盘中时间戳评价：

1. `triggered_at` 是所有介入原子条件首次同时满足的时间。
2. `invalidation_at` 是任一不介入或失效原子条件首次满足的时间。
3. `trigger_reference_price` 使用所有介入条件首次同时满足时，最小可靠盘中周期的
   收盘价，优先使用 5 分钟 K 线；只有日线、无法确定先后顺序或缺少参考价格时，
   结果为 `not_evaluable`。
4. 从未出现 `triggered_at` 时为 `not_triggered`；`invalidation_at` 早于或等于
   `triggered_at` 时为 `invalidated_before_entry`。
5. 介入先触发、随后出现失效条件时为 `incorrect`；介入先触发且没有随后失效，
   T+1 收盘价高于触发参考价时为 `correct`，否则为 `incorrect`。
6. 同时记录 T+1 收盘价和
   `t1_return_pct = (T+1 收盘价 / 触发参考价 - 1) * 100`。
7. 使用 T-1 已记录的 `evaluation_benchmark`，按
   `decision-integrity-rules.md` 计算同起点基准收益、超额收益和相对结果；不得
   在 T+1 更换基准。旧报告未记录基准时，相对结果为 `not_evaluable`。

`not_triggered` 和 `invalidated_before_entry` 不计入方向正确率，但必须分别用于统计
触发率和介入前失效率。HTML 使用下列中文显示，英文枚举只保留在 context JSON：

| JSON 标签 | HTML 中文显示 |
| --- | --- |
| `correct` | 方向正确 |
| `incorrect` | 方向错误 |
| `not_triggered` | 未触发介入 |
| `invalidated_before_entry` | 介入前已失效 |
| `not_evaluable` | 无法评价 |

相对结果在 HTML 中将 `outperformed`、`matched`、`underperformed` 和
`not_evaluable` 分别显示为“跑赢基准”“持平基准”“跑输基准”和“无法评价”。

## 写入验证区块

每个已复盘项目都在 JSON 中保留 `prior_item_id`、`condition_results` 和证据 ID；
HTML 使用对应的中文项目名称、中文结果标签、评价窗口和一句事实说明，不直接显示
`market-risk-off` 等内部 ID。个股计划与候选还要显示触发时间、触发参考价、失效
时间、T+1 收盘价、T+1 收益率和方向结果；没有对应值时显示“无法评价”，不得
伪造。存在预先记录的基准时，同时显示基准、同期基准收益、超额收益和相对结果。
只带入未解除的风险和仍相关的条件；不得编辑旧报告、阈值或 Skill。该复盘
由 Agent 依据结构化数据完成，不等同于程序化回测或自动交易验证。

存在可评价的上一日个股计划或候选时，验证区块同时显示：触发率 =
`(correct + incorrect) / 全部可评价项`，介入前失效率 =
`invalidated_before_entry / 全部可评价项`，方向正确率 =
`correct / (correct + incorrect)`。分母为零时显示“无法评价”，不得写成 0%。
同时显示相对跑赢率 = `outperformed / 全部可评价相对结果`；`matched` 计入分母，
`not_evaluable` 不计入。分母为零时显示“无法评价”。
