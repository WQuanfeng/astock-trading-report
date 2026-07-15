# Stock daily workflow

## Collect with a-stock-data

For the requested stock and the selected completed trading day, collect:

- Latest quote and daily K-line history with an explicit adjustment basis.
- Volume, turnover, relative position, moving-average state, and recent price
  performance using data/indicators already provided by `a-stock-data`.
- Industry/concept membership and the sector's same-day strength.
- Same-day and recent announcements, company replies, and news. Preserve
  original source, time, and whether it is an official disclosure.
- Available fund flow, dragon-tiger data, major unlocking/reduction events, and
  material risk information.
- T-1 context for this exact stock, when its report exists.

## Interpret in this order

1. Separate official facts from media interpretation.
2. Explain price/volume behavior in the context of its sector and market regime.
3. Identify evidence that supports and contradicts continuation.
4. Verify whether yesterday's stated trigger or invalidation condition occurred.
5. Produce a conditional next-day plan rather than a direct buy/sell call.

## Required output for each stock

- Status: `observe`, `eligible if triggered`, `not suitable to initiate`, or
  `insufficient data`.
- Three or fewer supporting facts, with dates and sources.
- At least one contrary fact or unresolved risk.
- A precise trigger condition; it may be a price/volume/sector condition, but
  must be explicitly conditional.
- A precise invalidation or no-entry condition.
- Relevant event, liquidity, sector, and market-regime risks.

Never use an announcement headline alone to infer a durable financial impact.
