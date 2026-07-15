# Screening rules

Run **low active leader** only after the user explicitly names a sector. Use
that sector's members from `a-stock-data`; do not replace it with a guessed
concept.

Run **early trend** across the eligible A-share universe and do not require a
sector. If `a-stock-data` cannot establish full-universe coverage, disclose the
actual universe used and do not describe the result as a market-wide ranking.

## Shared exclusion rules

Exclude or explicitly flag stocks when available data shows:

- ST/other special-treatment or delisting-risk status;
- insufficient trading history for the requested horizon;
- suspended or severely illiquid trading;
- imminent material unlocking, reduction, regulatory, or financial-risk events;
- missing data that prevents a required criterion from being evaluated.

Return at most two candidates per mode. Returning none is valid.

## Low active leader candidates

This is a **low-position active-leader watch pool**, not a long-term buy list.

### Evidence to collect

- At least one to two years of available price, limit-up, turnover, and sector
  performance evidence.
- Repeated historical limit-up or other objectively available active-trading
  evidence.
- Historical sector-strength evidence during the stock's strong periods.
- Current two-year price position/drawdown or other available low-position
  measure.
- Current financial snapshot, cash-flow/earnings condition where available, and
  material risk disclosures.

### Inclusion logic

Describe a stock as having a **historical leadership proxy**, never as a proven
sector leader, unless the collected source directly establishes that fact. The
proxy must combine:

1. repeated historical active-trading evidence;
2. strong relative performance during a documented sector-strength period; and
3. adequate current liquidity.

It must also have a clearly stated low-position/basing condition and no obvious
fundamental or event-risk disqualifier. Explain every included and excluded
criterion in the HTML.

## Early-trend candidates

This is a **trend-continuation watch pool**, not a probability claim.

Screen the eligible market-wide universe. Sector strength remains a confirmation
factor for each stock, but is not a user-input filter or a prerequisite.

### Evidence to collect

- Daily K-line and available moving-average/return indicators.
- Whether the current rise has included a limit-up.
- Short- and medium-window returns to test whether the move is already extended.
- Volume/turnover behavior, sector confirmation, available flow context, and
  upcoming material events.

### Inclusion logic

Require all of the following, subject to available data:

1. price above rising medium-term trend measures;
2. a positive but not clearly extended recent rise;
3. no limit-up in the identified current rising phase;
4. non-blow-off participation/volume behavior;
5. sector conditions that do not contradict the trend; and
6. no material near-term event risk.

For every candidate, show why it passed and the specific condition that would
invalidate the setup. Do not describe any candidate as having a "very high"
probability unless an explicitly disclosed, separate historical test supports
that phrase.
