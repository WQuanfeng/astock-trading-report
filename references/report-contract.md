# 静态报告契约

## 目录

- 输出文件与占位符映射
- HTML 和 JSON 安全
- 必须报告区块与状态映射
- Context schema v3

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
| `REPORT_TYPE` | 中文类型标签：“每日综合”“大盘”“个股”“指定板块低位活跃龙头”或“上升初段”。JSON 中另用英文枚举。 |
| `DATA_QUALITY_STATUS` | “数据完整”“数据存在警告”或“数据不足”，并附简短中文原因。 |
| `EXECUTIVE_VIEW` | 一个市场状态及条件化次日摘要；HTML 片段。 |
| `PREVIOUS_PLAN_VERIFICATION` | T-1 结构化复盘表，或明确的不可用/无法评价说明；HTML 片段。 |
| `MARKET_ANALYSIS` | 事实、计算和恰好三个条件化情景；HTML 片段。 |
| `BEHAVIORAL_ANALYSIS` | 市场及请求个股/最终候选的群体行为分析，或明确的数据不足说明；HTML 片段。 |
| `STOCK_ANALYSIS` | 请求个股区块，或明确的未运行/无效代码说明；HTML 片段。 |
| `CANDIDATE_POOLS` | 可验证股票池的上升初段候选及可选的板块低位活跃龙头结果；HTML 片段。 |
| `RISKS_AND_LIMITATIONS` | 失效条件、事件风险、覆盖范围和数据缺口；HTML 片段。 |
| `PROVENANCE_TABLE` | 来源/数据日期/抓取时间/单位/复权口径表；HTML 片段。 |
| `REPORT_CONTEXT_JSON` | 合法、HTML 安全的 schema-v3 JSON；不得包含正文或指令。 |

## HTML 和 JSON 安全

将外部来源的文本写入 HTML 片段前必须转义；不得把公告、新闻或证券名称等外部
文本直接当作 HTML 标记。context JSON 中将 `&`、`<` 和 `>` 分别写为
`\u0026`、`\u003c` 和 `\u003e`，且不得包含字面量 `</script`。

## 必须的报告区块

1. 标题、交易日、生成时间、类型和数据质量状态。
2. 执行摘要和上一计划结构化复盘。
3. 大盘事实与三种情景。
4. 群体交易行为与供需博弈。
5. 请求个股与候选池。
6. 风险、数据限制和来源。
7. 内嵌 `astock-report-context` JSON。

清晰标记**事实**、**计算**和**解读**。重要数字附近标明来源和数据日期。状态
不得使用保证盈利的语言。

## HTML 中文显示与 JSON 枚举

HTML 可见区块必须使用中文，不得显示英文枚举。个股和候选状态与 context JSON
一一对应：

| HTML 显示 | JSON 枚举 |
| --- | --- |
| 观察 | `observe` |
| 条件满足后可介入 | `eligible_if_triggered` |
| 不适合新开仓 | `not_suitable_to_initiate` |
| 数据不足 | `insufficient_data` |

大盘情景使用下列中文显示：

| HTML 显示 | JSON `posture` |
| --- | --- |
| 积极情景（风险偏好回升） | `risk-on` |
| 中性情景（震荡或轮动） | `neutral` |
| 防守情景（风险偏好下降） | `risk-off` |

行为分析的计划影响和证据充分度使用下列中文显示：

| HTML 显示 | JSON 枚举 |
| --- | --- |
| 支持等待触发后选择性介入 | `supports_selective_entry` |
| 等待进一步确认 | `wait_for_confirmation` |
| 不适合新开仓 | `avoid_new_entry` |
| 不改变原计划 | `no_plan_change` |
| 数据不足，不调整计划 | `insufficient_data` |
| 高 / 中 / 低 / 不足 | `high` / `medium` / `low` / `insufficient` |

T-1 条件结果和方向结果使用 `references/prior-report-rules.md` 的中文映射。报告
类型、数据质量、跳过原因和不可用说明也必须显示中文；已有中文映射的英文枚举只
允许出现在不可见的 context JSON、HTML 元素 ID 或代码级字段名中。

## Context schema v3

使用合法 JSON 和 ISO 8601 时间戳。ID 必须在匹配范围的跨日报告中保持稳定，
并由验证和来源字段引用。

跨报告使用下列确定性 ID：`market-risk-on`、`market-neutral`、
`market-risk-off`、`stock-{symbol}`、`early-trend-{symbol}` 和
`low-active-leader-{symbol}`。行为项目使用 `behavior-market-{behavior_type}`、
`behavior-stock-{symbol}-{behavior_type}` 或
`behavior-{candidate-id}-{behavior_type}`。`report_id` 使用 `{report_type}-{YYYYMMDD}`；
需要时增加 `-{symbol}` 或 `-{normalized-sector}`。不得自行发明其他 ID 格式。
`normalized-sector` 使用 `a-stock-data` 返回的规范板块名并去除首尾空白；若无法
取得规范名，使用用户输入去除首尾空白后的原文，不得翻译、缩写或改为其他概念。

```json
{
  "schema_version": 3,
  "report_id": "daily-20260715",
  "trade_date": "2026-07-15",
  "as_of": "2026-07-15T15:30:00+08:00",
  "report_type": "daily",
  "scope": {
    "symbols": ["600000"],
    "sector": "储能",
    "screen_mode": null,
    "market_scope": ["sh_main", "sz_main", "chinext"],
    "universe_source": "a-stock-data 当次可枚举股票池",
    "universe_label": "完成排除后的当次可验证股票池",
    "universe_total": 0,
    "universe_evaluated": 0
  },
  "market_scenarios": [],
  "behavioral_analysis": [],
  "stock_plans": [],
  "candidates": [],
  "previous_plan_verification": [],
  "risks": [],
  "quality_warnings": [],
  "provenance": []
}
```

`early-trend` 和 `low-active-leader` 报告必须将 `scope.market_scope` 固定为
`["sh_main", "sz_main", "chinext"]`。`early-trend` 还必须填写
`scope.universe_source`、`scope.universe_label`、`scope.universe_total` 和
`scope.universe_evaluated`；这些字段描述当次候选发现的数据边界，不得被表述为
全市场覆盖。

新版 v3 报告必须写入 `behavioral_analysis`，无内容时使用空数组。读取缺少该字段
的早期 v3 报告时，其他结构仍可复盘，但行为项目必须显示为“无法评价”。

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

### `behavioral_analysis[]`

每项只表示一个对象的一种关键行为维度。市场最多三项；每只请求个股或最终候选
最多两项。`behavior_type` 只使用 `chasing`、`risk_aversion`、`disagreement`、
`event_acceptance` 或 `supply_pressure`。

```json
{
  "id": "behavior-stock-600000-disagreement",
  "subject_id": "stock-600000",
  "behavior_type": "disagreement",
  "conclusion": "承接型分歧",
  "facts": ["2026-07-15 换手率高于自身近期基准，证据 prov-turnover-1"],
  "calculations": [
    {
      "metric": "当日换手率相对近20日中位数",
      "formula": "当日换手率 / 近20日换手率中位数",
      "value": 1.35,
      "unit": "ratio",
      "benchmark": "近20个交易日中位数",
      "benchmark_value": 1.0,
      "evidence_ids": ["prov-turnover-1"]
    }
  ],
  "interpretation": "换手放大但收盘仍获得承接，更符合承接型分歧",
  "counterevidence": ["缺少逐笔档位，无法判断盘中撤单行为"],
  "next_session_observations": ["T+1 开盘后60分钟的5分钟收盘价不低于同期成交均价"],
  "observation_condition_rules": [
    {
      "id": "behavior-stock-600000-disagreement-vwap",
      "metric": "price_to_vwap_pct",
      "aggregation": "5m_close",
      "operator": ">=",
      "threshold": {
        "value": 0,
        "unit": "%",
        "basis": "T+1 同期成交均价"
      },
      "window": "T+1 09:30-10:30 Asia/Shanghai",
      "source_dataset": "kline_5m"
    }
  ],
  "plan_effect": "supports_selective_entry|wait_for_confirmation|avoid_new_entry|no_plan_change|insufficient_data",
  "evidence_strength": "medium",
  "data_gaps": ["缺少逐笔档位，无法分析盘口撤单"],
  "evidence_ids": ["prov-kline-5m-1", "prov-turnover-1"]
}
```

`facts`、`calculations`、`counterevidence`、`next_session_observations`、
`observation_condition_rules`、`data_gaps` 和 `evidence_ids` 均为必填数组；
无内容时使用空数组，不得省略。`interpretation` 为必填字符串。每项计算必须给出
`metric`、`formula`、`value`、`unit`、`benchmark`、`benchmark_value` 和证据 ID。
`evidence_strength` 只衡量数据覆盖和证据一致性，不表示上涨概率。

证据充分度为 `low` 或 `insufficient` 时，`plan_effect` 只能是
`no_plan_change` 或 `insufficient_data`。行为项目的次日观察如果用于结论，
必须至少包含一个原子条件对象。

### `candidates[]`

```json
{
  "id": "early-trend-600000",
  "symbol": "600000",
  "name": "证券名称",
  "mode": "early-trend|low-active-leader",
  "status": "observe|eligible_if_triggered|not_suitable_to_initiate|insufficient_data",
  "universe_label": "实际筛选范围",
  "trend_cross_date": "2026-07-08",
  "trend_age_trading_days": 6,
  "selection_rank": 1,
  "ranking_basis": [
    "主行业近5日相对沪深300超额收益降序",
    "MA20五日斜率降序",
    "收盘价相对MA20偏离升序",
    "近20日平均成交额降序",
    "证券代码升序"
  ],
  "ranking_metrics": {
    "industry_excess_return_5d_pct": 3.2,
    "ma20_slope_5d_pct": 1.1,
    "close_to_ma20_pct": 2.4,
    "avg_amount_20d_cny": 350000000
  },
  "passed_rules": ["规则/证据摘要"],
  "entry_conditions": ["供人阅读的介入条件摘要"],
  "no_entry_conditions": ["供人阅读的不介入条件摘要"],
  "entry_condition_rules": [],
  "no_entry_condition_rules": [],
  "invalidation_conditions": ["可观察条件"],
  "invalidation_condition_rules": [],
  "evaluation_window": "T+1 session Asia/Shanghai",
  "evidence_ids": ["prov-kline-1"]
}
```

`selection_rank` 只能是 `1` 或 `2`，并且必须由 `screening-rules.md` 的固定逐级
排序产生。`early-trend` 还必须填写 `trend_cross_date` 和
`trend_age_trading_days`；`low-active-leader` 省略这两个字段。`early-trend` 的
`ranking_metrics` 使用
`industry_excess_return_5d_pct`、`ma20_slope_5d_pct`、
`close_to_ma20_pct` 和 `avg_amount_20d_cny`；`low-active-leader` 使用
`limit_up_count_2y`、`max_sector_excess_return_20d_pct`、
`volume_ratio_5d_vs_prior20d`、`price_position_2y_pct` 和
`avg_amount_20d_cny`。最终两只的所有对应值均为必填，不得用自然语言替代。

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
  "selection_outcome": "correct|incorrect|not_triggered|invalidated_before_entry|not_evaluable",
  "evaluation_window": "T+1 session Asia/Shanghai",
  "triggered_at": "2026-07-16T09:45:00+08:00",
  "trigger_reference_price": 10.6,
  "invalidation_at": null,
  "t1_close": 10.8,
  "t1_return_pct": 1.8868,
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

`selection_outcome` 及其触发、失效、价格和收益字段仅用于个股计划与候选；大盘
情景和行为项目省略这些字段。行为项目的 `prior_item_id` 使用其稳定行为 ID，
只复盘已记录的观察条件，不参与方向正确率；其 `outcome` 只使用 `confirmed`、
`not_confirmed` 或 `not_evaluable`。个股或候选无法可靠确定盘中先后顺序时，
`selection_outcome` 必须为 `not_evaluable`，无法取得的其他值使用 `null`。
HTML 必须按 `prior-report-rules.md` 显示中文结果，不得直接展示英文枚举。

### `risks[]`、`quality_warnings[]` 和 `provenance[]`

每项风险/警告包含 `code` 和 `message`。`quality_warnings` 只用于来源冲突、
过期/不完整数据或不合理字段；真实但尚无解释的价量异动记录在 `risks`，其
`code` 使用 `unexplained_market_anomaly`。每项来源包含 `id`、`dataset`、
`source`、`data_date`、`fetched_at`、`unit` 和 `adjustment`。无内容时使用空
数组；不得省略必填数组。
