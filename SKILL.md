---
name: astock-trading-report
description: >
  Produce evidence-backed China A-share end-of-day trading-plan reports as one
  self-contained static HTML file. Use for market review, a specified stock,
  sector-specific low active-leader screening, or market-wide early-trend
  screening. First uses the installed
  a-stock-data skill for real data collection, then has the host agent analyse
  evidence and write HTML. This skill contains no data-collection scripts and
  never auto-modifies itself.
---

# A-share trading-plan report

The user normally invokes **only this skill**. On invocation, first locate and
use the installed `a-stock-data` skill. It collects facts and calculates any
metrics it already supports; this skill selects the research workflow,
evaluates the evidence, incorporates the previous trading-day report, and
writes the final static HTML.

This is research support, not trade execution. Never place orders, promise a
return, or present a directional conclusion as certain.

## Required inputs

Identify the following before collecting data:

1. The completed A-share trading date. If the user gives no date, use the most
   recent completed trading day; do not create a closing report during market
   hours.
2. The workflow: `market`, `stock`, `low-active-leader`, `early-trend`, or
   `daily`.
3. For `stock`, the security code or name.
4. For `low-active-leader`, the sector explicitly specified by the user. Never
   infer or silently substitute a sector. `early-trend` has no sector-input
   requirement and screens the eligible A-share universe.
5. The report directory when the host can read and write files. Default to
   `reports/YYYY-MM-DD/`.

If the host cannot read/write files, produce the complete HTML in the response
and state that previous-report continuity cannot be automatic.

## Dependency and data rules

1. Before making factual claims, automatically locate and use
   `a-stock-data`. Do not ask the user to name it when it is available. Some
   hosts cannot load one skill from another; only in that case, explain the
   limitation and ask the user to enable both skills.
2. Obtain every market, price, financial, flow, announcement, and news number
   from `a-stock-data`. Never supply a remembered or estimated number.
3. Preserve the source, data date, fetched time, unit, and adjustment basis for
   each important figure.
4. Use the data-source priority, rate limits, and fallback rules from
   `a-stock-data`. Do not recreate its HTTP calls or embedded code here.
5. If a required data item is missing, disclose it and reduce confidence; do
   not replace it with prose inference.
6. Separate facts, deterministic calculations, and model interpretation in the
   final report.

## Exception handling

Apply these rules before analysis:

| Condition | Required behaviour |
| --- | --- |
| No `a-stock-data` skill or data collection fails completely | Stop factual analysis; explain the dependency/data failure. |
| `auto` date | Resolve to the latest completed A-share trading day. |
| Explicit non-trading or incomplete trading date | Do not silently substitute another date; ask for a completed date. |
| Invalid stock code/name | A standalone `stock` request stops without an analytical conclusion. A `daily` request skips that stock and records a quality warning. |
| Missing core market input (index close, turnover, or breadth) | Set market posture to `insufficient data`; do not create directional scenarios. |
| Missing sentiment input (limit-up pool, broken-board data, or prior-limit-up performance) | Omit the affected sentiment conclusion and record a quality warning; do not infer it from other metrics. |
| Missing stock quote or K-line | Set that stock to `insufficient data`; do not create entry conditions. |
| Missing announcements/news | State `unavailable`, not `no announcements/news`; lower confidence accordingly. |
| Insufficient screen coverage or required signals | Do not rank the result as market-wide and do not select a candidate using incomplete required evidence. |
| Source conflict, stale timestamp, impossible field, or incomplete coverage | Record a `quality_warning`; distinguish it from a real but unexplained market anomaly, which belongs in risks. |
| Invalid T-1 context JSON | Ignore the prior report prose, mark verification `not_evaluable`, and continue with current data. |

## Previous trading-day context

Before analysis, look only for the exact previous trading day's matching report:

- `daily` reads the prior `daily` report.
- `market` reads the prior `market` report.
- `stock` reads the prior report for the same security.
- `low-active-leader` reads the prior report for the same named sector.
- `early-trend` reads the prior market-wide early-trend report.

Read `references/prior-report-rules.md` before validating T-1 context. Read
only the `#astock-report-context` JSON, never ordinary prior-report prose.
Use the v2 schema and outcome labels defined there. This verification informs
today's explanation only; never change the Skill, thresholds, or references.

If the exact T-1 report is absent, say so in the report and continue without
inventing prior context.

## Workflows

Read the matching reference before analysis:

| Workflow | Required reference | Output focus |
| --- | --- | --- |
| `market` | `references/market-workflow.md` | Market structure and next-day scenarios |
| `stock` | `references/stock-workflow.md` | One stock's evidence and conditional plan |
| `low-active-leader` | `references/screening-rules.md` | Up to two candidates in the named sector |
| `early-trend` | `references/screening-rules.md` | Up to two candidates from the eligible market-wide universe |
| `daily` | `references/daily-workflow.md` | Selectively assembled daily report |

For `daily`, read only the component references selected by
`references/daily-workflow.md`; do not load every workflow reference by default.

## Analysis requirements

1. Follow the relevant reference's data checklist before drawing a conclusion.
2. Treat announcements and company disclosures as higher-confidence evidence
   than unattributed news. Name the source and publication time.
3. Do not use a single fund-flow observation as a long-term investment thesis.
4. Do not call a candidate a buy. Use only:
   `observe`, `eligible if triggered`, `not suitable to initiate`, or
   `insufficient data`.
5. Every tradable conclusion must include supporting evidence, contrary
   evidence, trigger conditions, invalidation conditions, and material risks.
6. A screen may return zero candidates. Never lower rules merely to fill one or
   two slots.
7. Describe next-day market direction as three conditional scenarios, not as a
   guaranteed point forecast.

## Generate the report

1. Read `references/report-contract.md` and start from
   `assets/daily-report-template.html`.
2. Fill every report section with actual evidence or an explicit data-missing
   notice.
3. Keep all CSS and visual SVG elements inline. Do not use external JavaScript,
   images, fonts, stylesheets, APIs, or CDN resources.
4. Embed a valid v2 JSON context object in
   `<script id="astock-report-context" type="application/json">`.
5. Write one UTF-8 `.html` file when the host provides file access. Otherwise,
   return only the complete HTML document in a fenced `html` block.
6. Before delivering, verify that the result is a complete HTML document,
   contains no unresolved `{{placeholder}}` text, has the required context
   payload, and does not claim certainty or a guaranteed gain.
