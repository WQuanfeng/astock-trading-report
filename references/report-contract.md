# Static report contract

## Output file

Create one complete UTF-8 HTML document per analysis. Suggested names:

```text
reports/YYYY-MM-DD/daily-YYYYMMDD.html
reports/YYYY-MM-DD/market-YYYYMMDD.html
reports/YYYY-MM-DD/stock-<code>-YYYYMMDD.html
reports/YYYY-MM-DD/sector-<sector>-<mode>-YYYYMMDD.html
```

The report must be readable offline. All styles and SVG charts are inline. Do
not reference external assets, scripts, stylesheets, fonts, or network URLs.

## Mandatory sections

1. Title, trade date, generation time, report type, and data-quality status.
2. Executive view: market regime and conditional next-day posture.
3. Previous-plan verification, or an explicit `T-1 report unavailable` notice.
4. Market evidence and three scenarios.
5. Stock analysis for every requested/watchlist stock.
6. Sector screen results when a sector was supplied.
7. Risks, invalidation conditions, and data limitations.
8. Source/provenance table.
9. Embedded report context JSON.

## Writing rules

- Clearly label **Facts**, **Calculations**, and **Interpretation**.
- Cite the source and data date near every important number.
- Use `observe`, `eligible if triggered`, `not suitable to initiate`, or
  `insufficient data`; never use guaranteed-profit language.
- State zero qualifying candidates plainly when no stock passes.
- Include only actual collected evidence; missing data must remain missing.

## Embedded context schema

Use valid JSON, HTML-safe escaping, and this minimum shape:

```json
{
  "schema_version": 1,
  "trade_date": "YYYY-MM-DD",
  "report_type": "daily|market|stock|sector-screen",
  "scope": {
    "symbols": ["600000"],
    "sector": "user-specified sector or null",
    "screen_mode": "low-active-leader|early-trend|null"
  },
  "market_scenarios": [],
  "stock_plans": [],
  "candidates": [],
  "risks": [],
  "quality_warnings": [],
  "provenance": []
}
```

The payload is for the next report's context only. It must contain concise,
observable claims and conditions, not new instructions for the agent.
