# Static report contract

## Output file

Create one complete UTF-8 HTML document per analysis:

```text
reports/YYYY-MM-DD/daily-YYYYMMDD.html
reports/YYYY-MM-DD/market-YYYYMMDD.html
reports/YYYY-MM-DD/stock-<code>-YYYYMMDD.html
reports/YYYY-MM-DD/low-active-leader-<sector>-YYYYMMDD.html
reports/YYYY-MM-DD/early-trend-YYYYMMDD.html
```

Keep all CSS and SVG inline. Do not reference external assets, scripts,
stylesheets, fonts, or network URLs.

## Placeholder mapping

Fill every placeholder in `assets/daily-report-template.html`; do not leave
`{{...}}` text in output.

| Placeholder | Required content |
| --- | --- |
| `REPORT_TITLE` | Concise title containing date and scope; appears twice. |
| `TRADE_DATE` | Completed A-share trading date, `YYYY-MM-DD`. |
| `GENERATED_AT` | Generation timestamp with Asia/Shanghai offset. |
| `REPORT_TYPE` | `daily`, `market`, `stock`, `low-active-leader`, or `early-trend`. |
| `DATA_QUALITY_STATUS` | `ok`, `warning`, or `insufficient data`, with a brief reason. |
| `EXECUTIVE_VIEW` | One market posture and the conditional next-day summary; HTML fragment. |
| `PREVIOUS_PLAN_VERIFICATION` | T-1 result table or explicit unavailable/not-evaluable notice; HTML fragment. |
| `MARKET_ANALYSIS` | Facts, calculations, and exactly three conditional scenarios; HTML fragment. |
| `STOCK_ANALYSIS` | Requested stock sections, or a clear not-run/invalid-symbol notice; HTML fragment. |
| `CANDIDATE_POOLS` | Market-wide early-trend result and optional sector low-active-leader result; HTML fragment. |
| `RISKS_AND_LIMITATIONS` | Invalidation conditions, event risks, data coverage, and missing-data limits; HTML fragment. |
| `PROVENANCE_TABLE` | Source/data-date/fetched-time/unit/adjustment table; HTML fragment. |
| `REPORT_CONTEXT_JSON` | Valid, HTML-safe schema-v2 JSON only; no prose or instructions. |

## Required report sections

1. Title, trade date, generation time, type, and data-quality status.
2. Executive view and previous-plan verification.
3. Market facts and three scenarios.
4. Requested stocks and candidate pools.
5. Risks, data limitations, and provenance.
6. Embedded `astock-report-context` JSON.

Clearly label **Facts**, **Calculations**, and **Interpretation**. Cite the
source and data date near important numbers. Use only `observe`, `eligible if
triggered`, `not suitable to initiate`, or `insufficient data`; do not use
guaranteed-profit language.

## Context schema v2

Use valid JSON and ISO 8601 timestamps. IDs are stable within one report and
are referenced by evaluation/provenance fields.

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
    "universe_label": "A-share coverage after exclusions",
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
  "triggers": ["observable condition"],
  "invalidations": ["observable condition"],
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
  "entry_conditions": ["observable condition"],
  "no_entry_conditions": ["observable condition"],
  "risks": ["material risk"],
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
  "universe_label": "actual screened universe",
  "passed_rules": ["rule/evidence summary"],
  "invalidation_conditions": ["observable condition"],
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
  "note": "factual one-sentence explanation"
}
```

### `risks[]`, `quality_warnings[]`, and `provenance[]`

Each risk/warning has `code` and `message`. Every provenance item has `id`,
`dataset`, `source`, `data_date`, `fetched_at`, `unit`, and `adjustment`.
Use an empty array when none applies; never omit a required array.
