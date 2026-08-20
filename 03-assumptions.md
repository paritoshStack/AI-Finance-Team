# Assumptions

Read by the FP&A Manager (`roles/fpa_manager.md`) to build the 12-month forecast, and referenced by the Finance Controller (`roles/finance_controller.md`) for the starting cash balance.

Fill in every bracketed value before running the workflow. If a value doesn't apply to your business, write `[not applicable]` rather than deleting the line — the FP&A Manager treats a missing line differently from one explicitly marked not applicable.

---

## Revenue growth

- Base case monthly MRR growth rate: [add your own number, e.g. a %]
- Upside case delta vs. base: [add your own number, e.g. +X percentage points]
- Downside case delta vs. base: [add your own number, e.g. −X percentage points]
- Basis for these rates: [state where these came from — trailing 3-month actuals, sales pipeline, board target, etc.]

---

## Cost structure

- Fixed cost monthly growth rate: [add your own number, e.g. a %, typically driven by headcount]
- Variable cost ratio: [add your own number, e.g. variable costs as a % of revenue]
- Cost categories to treat as fixed: [list them, e.g. payroll, rent, core SaaS tooling]
- Cost categories to treat as variable: [list them, e.g. ad spend, usage-based infra, sales commissions]

---

## Hiring plan

- Planned headcount additions (by month, by team): [add your own plan, or write "none planned"]
- Scenario-specific hiring changes: [e.g. "downside case pauses all hiring from month 4" — leave blank if hiring plan is identical across all three scenarios]

---

## One-time items

- Known one-time costs and the month they hit: [add your own, e.g. "$[X] office buildout in month 3" — or write "none known"]
- Known one-time revenue events (e.g. a large contract signing): [add your own, or write "none known"]

---

## Cash position

- Starting cash balance (as of the most recent actuals month): [add your own number]
- Date/month this balance is as of: [add your own date]
- Source of this figure: [e.g. bank statement, latest board deck, accounting system export]

If this is left as `[not applicable]` or blank, the Finance Controller and FP&A Manager will report cash balance and runway as `not available` rather than guessing — the rest of the forecast still runs normally.

---

## Notes for whoever fills this in

- Round numbers are fine. This file drives a forecast, not a legal filing — reasonable estimates beat leaving a field blank.
- If you're unsure of a rate, use the trailing 3-month actual as your base case and say so in "Basis for these rates" above, rather than picking an arbitrary target.
- Update this file each planning cycle. The FP&A Manager uses whatever is here at the time the workflow runs — it does not remember prior versions.
