# Market daily workflow

## Collect with a-stock-data

For the selected completed trading day, collect:

- Major indices: Shanghai Composite, Shenzhen Component, ChiNext, CSI 300, and
  any index materially relevant to the user-specified sector.
- Index returns, intraday range where available, turnover, and T-1 turnover
  change.
- Rising/falling stock counts, limit-up count, limit-down count, broken-board
  count/rate, highest consecutive-board height, and prior limit-up performance;
  collect each available metric's T-1 change and first-board/continuation-board
  structure when available.
- Leading and lagging industries/concepts, including breadth instead of only
  their top return; compare their rank and breadth with T-1 to distinguish
  continuation from one-day rotation.
- Material market-wide news, policy releases, and macro events, with source and
  publication time.
- Available flow data. Treat it as supporting context, not a standalone signal.

## Interpret in this order

1. **Trend and liquidity**: explain T-1 changes in index direction, turnover,
   and broad participation; do not rely on absolute values alone.
2. **Breadth and sentiment**: advancing/declining counts, limit-up ecosystem,
   broken-board rate, and concentration of gains.
3. **Leadership**: whether the strongest themes held, expanded, rotated, or
   weakened during the session.
4. **Risk events**: overnight-sensitive policy, global, or disclosure risks.
5. **T-1 verification**: verify yesterday's stated scenarios and conditions.

## Required output

State a market regime, then exactly three conditional next-day scenarios:

| Scenario | Conditions to observe | Interpretation | New-position environment |
| --- | --- | --- | --- |
| Risk-on | Concrete opening, volume, breadth, and leadership conditions | Why the thesis is supported | Observe selectively / eligible if triggered |
| Neutral | Concrete mixed conditions | Why a range or rotation is likely | Only high-quality setups |
| Risk-off | Concrete deterioration conditions | Why risk should be reduced | Not suitable to initiate |

The report must identify the first 30–60 minute observations that distinguish
the scenarios. Each scenario needs at least one measurable trigger with an
explicit comparison baseline, such as prior-day same-window turnover or a
recent same-window median. Apply this precedence: classify `risk-off` first;
classify `risk-on` only when all of its stated triggers hold and no risk-off
condition holds; otherwise classify `neutral`. Do not translate a scenario into
an instruction to place an order.
