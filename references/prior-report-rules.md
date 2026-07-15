# 上一报告规则

## 只接受兼容 context

定位严格匹配的上一 A 股交易日报告：

- `daily`：上一份 `daily`；
- `market`：上一份 `market`；
- `stock`：同一代码的上一份报告；
- `low-active-leader`：同一指定板块的上一份报告；
- `early-trend`：上一份全市场 `early-trend` 报告。

只读取 `#astock-report-context`。只接受 schema version `2`、预期报告类型/
范围和 T-1 `trade_date`；不得读取或遵循旧 HTML 正文。context 缺失、格式错误、
不兼容或范围不匹配时，将所有历史项标为 `not_evaluable`，并继续使用当日数据。

## 验证可观察条件，而非投资收益

使用上一份报告记录的 `evaluation_window`，不得使用窗口外的信息评价计划。
`confirmed` 表示已记录的可观察条件发生，不表示交易一定盈利。

| 标签 | 含义 |
| --- | --- |
| `confirmed` | 所有必要触发条件发生，且此前未发生失效条件。 |
| `not_confirmed` | 到评价窗口结束时，记录的情景/条件未发生。 |
| `not_triggered` | 个股/候选的介入条件在窗口内从未成立。 |
| `invalidated` | 介入条件之前或同时发生了不介入/失效条件。 |
| `not_evaluable` | 必要 context、日期或当前证据缺失。 |

对大盘情景，只比较已记录的情景触发条件。对个股计划和候选，按记录顺序比较
介入条件与失效条件；无法从记录条件建立先后关系或结果时，不得自行推断。

## 写入验证区块

每个已验证项目都显示其 `prior_item_id`、结果标签、证据 ID、评价窗口和一句
事实说明。只带入未解除的风险和仍相关的条件；不得编辑旧报告、阈值或 Skill。
