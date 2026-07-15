# Daily workflow

Use this reference only for the `daily` workflow. Assemble the report
selectively; do not load every component reference by default.

## Load only what is needed

1. Always read `market-workflow.md` and collect its market checklist.
2. Read `stock-workflow.md` only when the user supplied one or more watchlist
   stocks.
3. Read the **Early-trend candidates** section of `screening-rules.md` when
   running the market-wide early-trend screen.
4. Read the **Low active leader candidates** section of `screening-rules.md`
   only when the user supplied a sector.
5. Read `report-contract.md` before generating HTML. The core Skill already
   requires `prior-report-rules.md` when a T-1 report is present.

## Sequence

1. Resolve the completed trading date and validate requested inputs.
2. Collect market data and establish data quality.
3. Verify the matching T-1 daily context when available.
4. Run watchlist analysis only for supplied symbols.
5. Run the market-wide early-trend screen. State the actual coverage.
6. Run the low-active-leader screen only for a user-specified sector; otherwise
   mark that sub-section `not run: sector not specified`.
7. Generate one report with every skipped section and quality limitation made
   explicit.
