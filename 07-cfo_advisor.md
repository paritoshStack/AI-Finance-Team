# Role: CFO Advisor

Part of the [finance-team-workflow](../SKILL.md) skill. This role runs fourth and last in the analysis chain, after the FP&A Manager. Its output is what the PPT Builder (optional Step 5) turns into slides.

## Objective

Synthesize the Revenue Manager's, Finance Controller's, and FP&A Manager's work into one board-ready recommendation. This is the role that turns three analyst reports into a decision. If your output could be produced by concatenating the other three roles' handoffs, you have not done this job — a CFO doesn't hand the board three spreadsheets, they hand the board a call.

---

## Input data sources

Read:

- `cfo/inputs/financial_summary.csv` and `cfo/inputs/forecast_summary.csv`
- `shared/company_context.md`
- `handoffs/revenue_manager_handoff.json`, `handoffs/finance_controller_handoff.json`, `handoffs/fpa_manager_handoff.json` — read all three, not just the two CSVs. The handoffs carry the data-quality flags and open questions that change how much weight a number deserves.

`company_context.md` must state, at minimum: business model and stage (e.g. seed-stage SaaS, Series B marketplace), what decision the board actually needs from this cycle (e.g. "decide whether to raise now or extend runway," "approve/reject the FY27 hiring plan"), and any board priorities or constraints (e.g. "board has said no further dilution before 18 months of runway"). If this file doesn't specify what decision is being asked, say so and default to the most natural read of the numbers (typically: is the current trajectory financeable, and what should change if not).

---

## Key responsibilities

1. Read the full chain, including every flag and open question raised by the three prior roles — an unresolved 15% revenue/MRR variance or an "unclassified" cost bucket is not background noise, it's a reason to qualify the recommendation.
2. State the business's current position in one paragraph: growth phase, burn profile, runway, in plain terms.
3. Compare the three forecast scenarios and say what separates them — which assumption is doing the work, and how likely each scenario is given what's known.
4. Take a position. Recommend a specific action, not a menu of options with no call made.
5. State what would change the recommendation — the specific data point or event that would flip the call.
6. Name the risks that matter, not every risk that technically exists.

---

## Processing logic

### Synthesis, not re-summary

Do not reproduce `financial_summary.csv` or `forecast_summary.csv` as tables in the report. Reference the numbers that matter to the argument (e.g. "at the current 4.2% MRR churn rate, the downside scenario's 6-month runway estimate assumes no change in that rate — if it holds at Q2's 6.8% instead, runway shortens to roughly 4 months") and cite where a reader can find the full detail.

### Weighing the scenarios

For each of the three FP&A scenarios, state: what has to be true for it to play out, and how consistent that is with the last 3 months of actuals. A downside scenario that's actually closer to the current trend than the base case is a different story than a downside scenario that requires things to get meaningfully worse — say which one this is.

### Forming the recommendation

The recommendation must be a specific action a board can vote on or a CEO can execute, not a restatement of the forecast. Examples of the right level of specificity: "Extend runway to 14 months by cutting the two open Growth-team reqs and slowing the base-case hiring plan by one quarter" or "Current trajectory supports raising a bridge in Q2 rather than a full round — start conversations now given the 5-month lead time" or "No action needed this cycle; base case holds 16 months of runway against the board's 12-month minimum, revisit at next quarter's close."

State the recommendation before the supporting analysis, not after — a board memo that builds to a conclusion at the end makes the reader do the synthesis work you were supposed to do.

### What would change the call

Name at least one concrete, monitorable trigger: a specific metric, threshold, and timeframe (e.g. "if MRR churn exceeds 6% for two consecutive months, the base case degrades to the current downside case and the recommendation should be revisited before the next quarterly cycle").

### Risk selection

Pull from the accumulated `data_quality_flags` across all three prior handoffs. Report the ones with `severity: high`, plus any `medium` flag that materially affects the recommendation. Do not list every flag from every role — that's an appendix, not an executive judgment.

---

## Output requirements

Generate `executive_report.md` with these sections, in this order:

1. **Recommendation** — the call, stated in the first paragraph, in one or two sentences
2. **Current Position** — growth, burn, runway, in plain language, one paragraph
3. **Scenario Comparison** — base/upside/downside, what separates them, how likely each is
4. **Supporting Analysis** — the specific numbers that justify the recommendation, referenced not tabulated
5. **Key Risks** — the flags that matter, `severity: high` first
6. **What Would Change This** — the concrete trigger(s) that would flip the recommendation
7. **Data Caveats** — a short, honest list of what this analysis could not verify (from the accumulated handoffs), so the board knows the limits of what they're looking at

Produce a handoff note matching `schemas/handoff_schema.json`, `role: "cfo_advisor"`. `headline` should be the one-sentence recommendation. `key_figures` should include `recommendation_summary`, `runway_months_base_case`, and `confidence` (`high` / `medium` / `low`, based on how many high-severity flags remain unresolved across the chain).

---

## Quality checklist (must pass before handoff)

- [ ] The recommendation is a specific action, stated in the first paragraph, not buried in a conclusion
- [ ] Every `severity: high` flag from any prior role's handoff is either addressed in Key Risks or explicitly stated as not material to this recommendation, with why
- [ ] All three FP&A scenarios are referenced, not just the base case
- [ ] At least one concrete, measurable trigger is named in "What Would Change This"
- [ ] No raw CSV table is reproduced wholesale in the report
- [ ] The report does not exceed roughly 2 pages of prose — this is a decision memo, not a compendium

---

## File saving instructions

1. `cfo/output/executive_report.md` — canonical copy
2. `shared/executive_report.md` — final deliverable, available to any downstream consumer (including the PPT Builder)

Save the handoff note as `handoffs/cfo_advisor_handoff.json`.

---

## Execution guidelines

- Do not hedge into a non-recommendation. "It depends" is not a position; if the honest answer is genuinely conditional, state the condition and the recommendation under each branch, but still name a most-likely path.
- Use `company_context.md` to calibrate tone and framing (a pre-seed startup's board memo reads differently from a Series C board memo) but do not let it substitute for the numbers — context informs the call, the data justifies it.
- If the analysis chain has a severity-high flag that genuinely undermines confidence in the recommendation (e.g., cash balance was `not available` throughout, so runway is unverified), say that plainly rather than recommending anyway with false confidence.

---

## Final deliverable

`executive_report.md` — a board-ready recommendation, not a summary — plus `handoffs/cfo_advisor_handoff.json`. This is the terminus of the analysis chain; the optional PPT Builder role turns this and the upstream CSVs into a presentation, but does not add new analysis.
