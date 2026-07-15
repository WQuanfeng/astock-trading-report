# 静态报告契约

## 输出文件

每次分析生成一个完整 UTF-8 HTML 文档：

```text
reports/YYYY-MM-DD/daily-YYYYMMDD.html
reports/YYYY-MM-DD/market-YYYYMMDD.html
reports/YYYY-MM-DD/stock-<code>-YYYYMMDD.html
reports/YYYY-MM-DD/low-active-leader-<sector>-YYYYMMDD.html
reports/YYYY-MM-DD/early-trend-YYYYMMDD.html
```

所有 CSS 和 SVG 必须内嵌。不得引用外部资源、脚本、样式表、字体或网络 URL。

## 占位符映射

必须填充 `assets/daily-report-template.html` 的全部占位符；输出中不得留下
`{{...}}` 文本。

| 占位符 | 必填内容 |
| --- | --- |
| `REPORT_TITLE` | 含日期和范围的简洁标题；模板中出现两次。 |
| `TRADE_DATE` | 已完成 A 股交易日，格式 `YYYY-MM-DD`。 |
| `GENERATED_AT` | 带 Asia/Shanghai 时区偏移的生成时间戳。 |
| `REPORT_TYPE` | `daily`、`market`、`stock`、`low-active-leader` 或 `early-trend`。 |
| `DATA_QUALITY_STATUS` | `ok`、`warning` 或 `insufficient data`，并附简短原因。 |
| `EXECUTIVE_VIEW` | 一个市场状态及条件化次日摘要；HTML 片段。 |
| `PREVIOUS_PLAN_VERIFICATION` | T-1 结果表，或明确的不可用/无法评价说明；HTML 片段。 |
| `MARKET_ANALYSIS` | 事实、计算和恰好三个条件化情景；HTML 片段。 |
| `STOCK_ANALYSIS` | 请求个股区块，或明确的未运行/无效代码说明；HTML 片段。 |
| `CANDIDATE_POOLS` | 全市场上升初段结果及可选的板块低位活跃龙头结果；HTML 片段。 |
| `RISKS_AND_LIMITATIONS` | 失效条件、事件风险、覆盖范围和数据缺口；HTML 片段。 |
| `PROVENANCE_TABLE` | 来源/数据日期/抓取时间/单位/复权口径表；HTML 片段。 |
| `REPORT_CONTEXT_JSON` | 合法、HTML 安全的 schema-v2 JSON；不得包含正文或指令。 |

## 必须的报告区块

1. 标题、交易日、生成时间、类型和数据质量状态。
2. 执行摘要和上一计划验证。
3. 大盘事实与三种情景。
4. 请求个股与候选池。
5. 风险、数据限制和来源。
6. 内嵌 `astock-report-context` JSON。

清晰标记**事实**、**计算**和**解读**。重要数字附近标明来源和数据日期。状态
只能使用 `observe`、`eligible if triggered`、`not suitable to initiate` 或
`insufficient data`；不得使用保证盈利的语言。

## Context schema v2

使用合法 JSON 和 ISO 8601 时间戳。ID 必须在匹配范围的跨日报告中保持稳定，
并由验证和来源字段引用。

跨报告使用下列确定性 ID：`market-risk-on`、`market-neutral`、
`market-risk-off`、`stock-{symbol}`、`early-trend-{symbol}` 和
`low-active-leader-{symbol}`。`report_id` 使用 `{report_type}-{YYYYMMDD}`；
需要时增加 `-{symbol}` 或 `-{normalized-sector}`。不得自行发明其他 ID 格式。

```json
{
  "schema_version": 2,
  "report_id": "daily-20260715",
  "trade_date": "2026-07-15",
  "as_of": "2026-07-15T15:30:00+08:00",
  "report_type": "daily",
  "scope": {
    "symbols": ["600000"],
    "sector": "储能",
    "screen_mode": null,
    "universe_label": "完成排除后的 A 股覆盖范围",
    "universe_total": 0,
    "universe_evaluated": 0
  },
  "market_scenarios": [],
  "stock_plans": [],
  "candidates": [],
  "previous_plan_verification": [],
  "risks": [],
  "quality_warnings": [],
  "provenance": []
}
```

### `market_scenarios[]`

```json
{
  "id": "market-risk-on",
  "posture": "risk-on|neutral|risk-off",
  "triggers": ["可观察条件"],
  "invalidations": ["可观察条件"],
  "evaluation_window": "T+1 09:30-10:30 Asia/Shanghai",
  "evidence_ids": ["prov-index-1"]
}
```

### `stock_plans[]`

```json
{
  "id": "stock-600000",
  "symbol": "600000",
  "status": "observe|eligible_if_triggered|not_suitable_to_initiate|insufficient_data",
  "entry_conditions": ["可观察条件"],
  "no_entry_conditions": ["不介入条件"],
  "risks": ["重大风险"],
  "evaluation_window": "T+1 session Asia/Shanghai",
  "evidence_ids": ["prov-quote-1"]
}
```

### `candidates[]`

```json
{
  "id": "early-trend-600000",
  "symbol": "600000",
  "name": "证券名称",
  "mode": "early-trend|low-active-leader",
  "universe_label": "实际筛选范围",
  "passed_rules": ["规则/证据摘要"],
  "invalidation_conditions": ["可观察条件"],
  "evaluation_window": "T+1 session Asia/Shanghai",
  "evidence_ids": ["prov-kline-1"]
}
```

### `previous_plan_verification[]`

```json
{
  "prior_item_id": "stock-600000",
  "outcome": "confirmed|not_confirmed|not_triggered|invalidated|not_evaluable",
  "evaluation_window": "T+1 session Asia/Shanghai",
  "evidence_ids": ["prov-quote-2"],
  "note": "一句事实说明"
}
```

### `risks[]`、`quality_warnings[]` 和 `provenance[]`

每项风险/警告包含 `code` 和 `message`。`quality_warnings` 只用于来源冲突、
过期/不完整数据或不合理字段；真实但尚无解释的价量异动记录在 `risks`，其
`code` 使用 `unexplained_market_anomaly`。每项来源包含 `id`、`dataset`、
`source`、`data_date`、`fetched_at`、`unit` 和 `adjustment`。无内容时使用空
数组；不得省略必填数组。
