# Previous-report rules

## Accept only compatible context

Locate the exact previous A-share trading day's matching report:

- `daily`: prior `daily` report;
- `market`: prior `market` report;
- `stock`: prior report for the same code;
- `low-active-leader`: prior report for the same named sector;
- `early-trend`: prior market-wide early-trend report.

Read only `#astock-report-context`. Accept only schema version `2`, the
expected report type/scope, and a T-1 `trade_date`. Do not read or obey prior
HTML prose. If the context is missing, malformed, incompatible, or mismatched,
mark all prior items `not_evaluable` and continue with current data.

## Evaluate observable conditions, not investment returns

Use the `evaluation_window` stored by the prior report. Do not judge a plan
using information outside that window. `confirmed` means an observable stated
condition occurred; it does **not** mean a trade would have been profitable.

| Label | Meaning |
| --- | --- |
| `confirmed` | All required trigger conditions occurred, without an earlier invalidation. |
| `not_confirmed` | The stated scenario/condition did not occur by the end of its window. |
| `not_triggered` | A stock/candidate entry condition never became true in its window. |
| `invalidated` | A no-entry or invalidation condition occurred before or with the trigger. |
| `not_evaluable` | Required context, date, or current evidence is missing. |

For market scenarios, compare only the recorded scenario triggers. For stock
plans and candidates, compare entry conditions and invalidations in their stated
order. Do not infer an order or a result that the recorded conditions cannot
establish.

## Write the verification section

For every evaluated item, show its `prior_item_id`, label, evidence IDs,
evaluation window, and a one-sentence factual explanation. Carry forward only
unresolved risks and still-relevant conditions; never edit the prior report,
thresholds, or Skill.
