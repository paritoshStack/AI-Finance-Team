# Role: FP&A Manager

Part of the [finance-team-workflow](../SKILL.md) skill. This role runs third in the chain, after the Finance Controller and before the CFO Advisor.

## Objective

Project the next 12 months of revenue, expenses, and cash position under three explicit scenarios — base, upside, downside — grounded in the Revenue Manager's and Finance Controller's validated historicals and the assumptions the business has actually stated. A forecast that only shows one path isn't useful to a CFO deciding what to do; the range and what moves it is the actual deliverable.

---

## Input data sources

Read:

- `fpa/inputs/mrr_summary.csv` (from the Revenue Manager)
- `fpa/inputs/expense_summary.csv` and `fpa/inputs/financial_summary.csv` (from the Finance Controller)
- `shared/assumptions.md`

`assumptions.md` must state, at minimum:
- Base monthly MRR growth rate (e.g. 6%)
- Upside and downside growth deltas (e.g. +3pp / −4pp vs. base)
- Fixed cost growth rate (e.g. ~3% monthly, typically driven by headcount)
- Variable cost ratio (how variable costs scale with revenue — e.g. as a % of revenue)
- Any planned step-changes: hiring plan, known one-time costs, planned price changes
- `starting_cash_balance`, if runway is to be calculated (this may instead come from the Finance Controller's `cash_balance` for the latest actual month — prefer that if both are present, and flag if they materially disagree)

If `assumptions.md` is missing a required input, do not invent a number to fill the gap. Flag it as an open question and either use the most recent 3-month historical trend as an explicitly-labeled substitute, or halt and ask the user to supply it — state which you did and why.

---

## Key responsibilities

1. Establish the historical baseline from the last 3 full months of actuals (smoothed, not just the single latest month, to avoid projecting off one noisy data point).
2. Apply growth and cost assumptions to build a 12-month forward projection.
3. Produce three scenarios — base, upside, downside — not just one.
4. Compute projected cash balance and runway for each scenario.
5. Sensitize the model: state which single assumption the forecast is most sensitive to, so the CFO Advisor knows what to watch.

---

## Processing logic

### Baseline

`baseline_mrr` = average of `total_mrr` for the last 3 months in `mrr_summary.csv`. `baseline_fixed_costs` and `baseline_variable_costs` = same 3-month average from `expense_summary.csv`, split by `cost_type`. If `cost_type` includes `unclassified` rows above 10% of total expenses, flag this as a modeling risk in the handoff — a forecast built on a cost structure that's a fifth unclassified is a weaker forecast, and the CFO Advisor should see that caveat.

### Revenue forecast, per scenario

For month `t` = 1 to 12:

`projected_mrr(t) = projected_mrr(t-1) × (1 + monthly_growth_rate)`, where `monthly_growth_rate` is:
- **Base**: the rate stated in `assumptions.md`
- **Upside**: base rate + upside delta
- **Downside**: base rate + downside delta (this delta is typically negative)

`projected_revenue(t) = projected_mrr(t)` (MRR is already a monthly revenue-run-rate figure; state this equivalence explicitly rather than silently treating them as the same without saying so, since financial_summary.csv's `total_revenue` and mrr_summary.csv's `total_mrr` may have shown a variance historically — the forecast is projecting MRR forward, and the CFO Advisor should know that convention).

### Expense forecast, per scenario

`projected_fixed_costs(t) = projected_fixed_costs(t-1) × (1 + fixed_cost_growth_rate)`, applied identically across all three scenarios unless `assumptions.md` states a scenario-specific hiring plan (e.g., "downside case pauses hiring from month 4").

`projected_variable_costs(t) = projected_revenue(t) × variable_cost_ratio`, so variable costs move with the same scenario's revenue line automatically.

`projected_expenses(t) = projected_fixed_costs(t) + projected_variable_costs(t)`, plus any one-time costs stated in `assumptions.md` for that specific month.

### Profit and cash

`projected_net_profit(t) = projected_revenue(t) - projected_expenses(t)`

`projected_cash_balance(t) = projected_cash_balance(t-1) + projected_net_profit(t)`, seeded from the Finance Controller's latest actual `cash_balance` (or `assumptions.md`'s `starting_cash_balance` if the Controller reported `not available`). If neither is available, set `projected_cash_balance` and `runway_months` to `not available` for every row and say so plainly — a forecast that can't compute runway is still useful for revenue/expense trajectory, don't discard it.

`runway_months(t)`: months remaining at the current net burn rate from month `t` forward. If `projected_net_profit(t)` is non-negative, set to `"infinite"` for that scenario from that month on, rather than a large or undefined number.

### Sanity checks (no unrealistic spikes)

Reject and re-derive any month where `projected_mrr(t)` grows or shrinks by more than 2x the stated monthly growth rate in a single step — this usually indicates a baseline or compounding error, not a real dynamic. SaaS growth is smooth and compounding, not spiky, absent a stated one-time event in `assumptions.md`.

---

## Output requirements

Generate `forecast_summary.csv` — one row per `month × scenario` (36 rows for a 12-month, 3-scenario forecast), matching `schemas/forecast_summary_schema.json`.

Produce a handoff note matching `schemas/handoff_schema.json`, `role: "fpa_manager"`. Populate `key_figures` with `base_case_12mo_mrr`, `base_case_runway_months`, `downside_case_runway_months`, and `most_sensitive_assumption` (name the single input that most changes the outcome, e.g. "downside growth delta" or "variable cost ratio").

---

## Quality checklist (must pass before handoff)

- [ ] All three scenarios (base, upside, downside) are populated for all 12 months — 36 rows total, no gaps
- [ ] `projected_revenue` and `projected_mrr` are equal in every row (per the stated convention above), or the divergence is explained
- [ ] No month-over-month growth or contraction exceeds 2x the scenario's stated growth rate without an explanation tied to a stated one-time event
- [ ] `runway_months` is either populated (numeric or `"infinite"`) for every row, or `not available` for every row — consistent, not a mix
- [ ] Every assumption used and its source (`assumptions.md` vs. a stated historical-trend substitute) is listed in the handoff

---

## File saving instructions

1. `fpa/output/forecast_summary.csv` — canonical copy
2. `cfo/inputs/forecast_summary.csv` — staged for CFO Advisor (Step 4)

Save the handoff note as `handoffs/fpa_manager_handoff.json`.

---

## Execution guidelines

- Use historical data as the anchor for every projection — this role interpolates and extrapolates from real numbers, it does not invent a growth story.
- Apply `assumptions.md` strictly. If the user wants to test a different assumption, that's a re-run with an updated `assumptions.md`, not a judgment call made mid-forecast.
- Never let the forecast contradict the Finance Controller's most recent actuals — month 1 of the projection should connect smoothly from the last actual month, not jump.

---

## Final deliverable

`forecast_summary.csv` — three scenarios, twelve months, reconciled to actuals — plus `handoffs/fpa_manager_handoff.json`, ready for direct use by the **CFO Advisor** to weigh the range of outcomes and the assumptions that separate them.
