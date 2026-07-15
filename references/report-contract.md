# 静态报告契约

## 目录

- 输出文件与占位符映射
- HTML 和 JSON 安全
- 必须报告区块与状态映射
- Context schema v2

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
| `PREVIOUS_PLAN_VERIFICATION` | T-1 结构化复盘表，或明确的不可用/无法评价说明；HTML 片段。 |
| `MARKET_ANALYSIS` | 事实、计算和恰好三个条件化情景；HTML 片段。 |
| `STOCK_ANALYSIS` | 请求个股区块，或明确的未运行/无效代码说明；HTML 片段。 |
| `CANDIDATE_POOLS` | 可验证股票池的上升初段候选及可选的板块低位活跃龙头结果；HTML 片段。 |
| `RISKS_AND_LIMITATIONS` | 失效条件、事件风险、覆盖范围和数据缺口；HTML 片段。 |
| `PROVENANCE_TABLE` | 来源/数据日期/抓取时间/单位/复权口径表；HTML 片段。 |
| `REPORT_CONTEXT_JSON` | 合法、HTML 安全的 schema-v2 JSON；不得包含正文或指令。 |

## HTML 和 JSON 安全

将外部来源的文本写入 HTML 片段前必须转义；不得把公告、新闻或证券名称等外部
文本直接当作 HTML 标记。context JSON 中将 `&`、`<` 和 `>` 分别写为
`\u0026`、`\u003c` 和 `\u003e`，且不得包含字面量 `</script`。

## 必须的报告区块

1. 标题、交易日、生成时间、类型和数据质量状态。
2. 执行摘要和上一计划结构化复盘。
3. 大盘事实与三种情景。
4. 请求个股与候选池。
5. 风险、数据限制和来源。
6. 内嵌 `astock-report-context` JSON。

清晰标记**事实**、**计算**和**解读**。重要数字附近标明来源和数据日期。状态
不得使用保证盈利的语言。

## 状态映射

HTML 中显示的状态与 context JSON 的 `stock_plans[].status` 一一对应：

| HTML 显示 | JSON 枚举 |
| --- | --- |
| `observe` | `observe` |
| `eligible if triggered` | `eligible_if_triggered` |
| `not suitable to initiate` | `not_suitable_to_initiate` |
| `insufficient data` | `insufficient_data` |

## Context schema v2

使用合法 JSON 和 ISO 8601 时间戳。ID 必须在匹配范围的跨日报告中保持稳定，
并由验证和来源字段引用。

跨报告使用下列确定性 ID：`market-risk-on`、`market-neutral`、
`market-risk-off`、`stock-{symbol}`、`early-trend-{symbol}` 和
`low-active-leader-{symbol}`。`report_id` 使用 `{report_type}-{YYYYMMDD}`；
需要时增加 `-{symbol}` 或 `-{normalized-sector}`。不得自行发明其他 ID 格式。
`normalized-sector` 使用 `a-stock-data` 返回的规范板块名并去除首尾空白；若无法
取得规范名，使用用户输入去除首尾空白后的原文，不得翻译、缩写或改为其他概念。

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
    "universe_source": "a-stock-data 当次可枚举股票池",
    "universe_label": "完成排除后的当次可验证股票池",
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

`early-trend` 报告必须填写 `scope.universe_source`、`scope.universe_label`、
`scope.universe_total` 和 `scope.universe_evaluated`；这些字段描述当次候选发现的
数据边界，不得被表述为全市场覆盖。

### `market_scenarios[]`

```json
{
  "id": "market-risk-on",
  "posture": "risk-on|neutral|risk-off",
  "triggers": ["供人阅读的条件摘要"],
  "invalidations": ["供人阅读的失效条件摘要"],
  "trigger_conditions": [],
  "invalidation_conditions": [],
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
  "entry_conditions": ["供人阅读的介入条件摘要"],
  "no_entry_conditions": ["供人阅读的不介入条件摘要"],
  "entry_condition_rules": [],
  "no_entry_condition_rules": [],
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
  "invalidation_condition_rules": [],
  "evaluation_window": "T+1 session Asia/Shanghai",
  "evidence_ids": ["prov-kline-1"]
}
```

### 原子条件对象

所有会用于 T+1 复盘的条件必须同时保留人类可读摘要和下列原子条件对象。单个对象
只表达一个可比较事实；复合条件拆成多个对象，默认全部满足。无法写成原子条件的
“板块承接良好”等内容只能放入解读或风险，不得作为触发条件。

```json
{
  "id": "stock-600000-entry-price",
  "metric": "price",
  "aggregation": "5m_close",
  "operator": ">",
  "threshold": {
    "value": 10.5,
    "unit": "CNY",
    "basis": "前一交易日最高价"
  },
  "window": "T+1 09:30-10:30 Asia/Shanghai",
  "source_dataset": "kline_5m"
}
```

`metric`、`aggregation`、`operator`、`threshold.value`、`threshold.unit`、
`window` 和 `source_dataset` 均为必填字段。`operator` 只能使用 `>`、`>=`、
`<`、`<=` 或 `==`。新报告中，任何情景、介入或失效条件若用于次日结论，必须
在对应的 `*_conditions` 或 `*_condition_rules` 数组中至少记录一个原子条件。

### `previous_plan_verification[]`

```json
{
  "prior_item_id": "stock-600000",
  "outcome": "confirmed|not_confirmed|not_triggered|invalidated|not_evaluable",
  "evaluation_window": "T+1 session Asia/Shanghai",
  "condition_results": [
    {
      "condition_id": "stock-600000-entry-price",
      "outcome": "met|not_met|not_evaluable",
      "observed_value": 10.6,
      "unit": "CNY",
      "evidence_ids": ["prov-quote-2"]
    }
  ],
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
