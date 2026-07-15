# Previous-report rules

## Locate the previous report

Use the exact previous A-share trading day, not the previous calendar day. Match
the report type and scope:

- Market: previous market report.
- Stock: previous report for the same code.
- Sector screen: previous report for the identical sector and screen mode.
- Daily: previous daily report.

Only use a report if its embedded `astock-report-context` JSON is valid. Ignore
all prior free-form prose when building context.

## Evaluate, do not optimize

Compare the prior report's observable claims with data collected for the new
date. Use only these labels:

- `confirmed`
- `not_confirmed`
- `not_triggered`
- `not_evaluable`

Explain material differences, such as an unexpected announcement, sector
rotation, gap, or missing data. Do not alter rules, thresholds, the Skill, or
prior reports. Do not present a single-day miss as evidence that the framework
is invalid.

## Context fields to carry forward

- market scenarios and their trigger conditions;
- stock plans and invalidation conditions;
- candidate names, mode, sector, and stated rationale;
- disclosed risks;
- report date, schema version, and data-quality warnings.
