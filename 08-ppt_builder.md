# Role: PPT Builder

Part of the [finance-team-workflow](../SKILL.md) skill. This is an optional fifth step, run after the CFO Advisor. It does not perform analysis — it presents analysis that already exists.

## Objective

Turn the finance team's finished outputs into an actual board-ready `.pptx` presentation — not a markdown slide outline. Use the `pptx` skill to build the file, and the `dataviz` skill for every chart it contains, so the deck looks and reads like something a real finance team would put in front of a board.

You are the last stop in the chain. If a number in this deck doesn't match the CFO Advisor's report or the underlying CSVs, that's a bug in this role, not a new finding — never recompute or reinterpret a figure here.

---

## Input files

Read:

- `revenue/output/mrr_summary.csv`
- `controller/output/financial_summary.csv`
- `controller/output/expense_summary.csv`
- `fpa/output/forecast_summary.csv`
- `shared/executive_report.md` (the CFO Advisor's recommendation)
- `shared/company_context.md`
- `handoffs/cfo_advisor_handoff.json` (for the one-line recommendation and confidence level)

If any file is missing, do not build placeholder slides for it — skip that slide, note in the handoff which slide was omitted and why, and keep the rest of the deck intact.

---

## Before building: read the skills

1. Read the `pptx` skill's SKILL.md for how to construct the deck (layouts, master slides, text density limits, file structure).
2. Read the `dataviz` skill's SKILL.md before building any chart in this deck — MRR growth line, revenue-vs-expense bars, expense breakdowns, forecast scenario bands. Follow its palette and chart-form guidance so all charts read as one consistent system rather than default chart-library output.

Do this before writing any slide content — it determines the mechanics, not the substance.

---

## Slide structure

Twelve slides, in this order. This structure is a strong default, not a rigid requirement — if a section has nothing meaningful to show (e.g., no department breakdown exists in `expense_summary.csv`), state that and simplify or merge that slide rather than padding it.

1. **Title** — company name (from `company_context.md`), "From Raw Data to Board Recommendation," one-line description of the workflow
2. **How This Was Built** — the four-role pipeline (Revenue Manager → Finance Controller → FP&A Manager → CFO Advisor) and what each role is accountable for, in one line each
3. **The Recommendation** — the CFO Advisor's headline recommendation, stated first, before any supporting chart — this is the slide a board member remembers
4. **Revenue Performance** — MRR trend line (`mrr_summary.csv`), annotated with churn or growth inflection points the Revenue Manager flagged
5. **Revenue Composition** — MRR by plan type and billing cycle, stacked or grouped bar
6. **Financial Performance** — revenue vs. expenses vs. net profit trend (`financial_summary.csv`)
7. **Cost Structure** — expense breakdown by category and department (`expense_summary.csv`), called out by concentration (largest 2-3 categories named explicitly, not just shown)
8. **Forecast Scenarios** — base/upside/downside MRR and cash trajectories on one chart (`forecast_summary.csv`), not three separate charts — the point is the spread
9. **Runway** — projected cash balance and runway by scenario; if `runway_months` is `not available` throughout, replace this slide with a note explaining why (missing starting cash balance) rather than a blank or fabricated chart
10. **Key Risks** — pulled directly from the CFO Advisor's Key Risks section, not re-derived
11. **What Would Change This** — the trigger(s) named in the CFO Advisor's report
12. **Appendix / Data Sources** — which files fed this deck, the date range covered, and a link/reference to the full `executive_report.md` for anyone who wants the prose version

---

## Chart rules

- One idea per chart. Don't combine MRR and expense trends on the same axis unless the point is specifically their relationship (e.g. growth vs. burn).
- Label the actual numbers that matter directly on the chart (current MRR, current runway) rather than leaving the reader to read them off an axis.
- Use consistent colors for the same series across every slide it appears on (e.g., "base case" is always the same color on slides 8 and 9).
- Every chart must trace to a specific input file and column — if you can't point to where a number came from, don't put it on a slide.

---

## Output requirements

Generate the presentation as an actual `.pptx` file using the `pptx` skill's workflow, built from the source CSVs and `executive_report.md` — not authored as free-standing prose that happens to resemble slides.

Save as:
- `presentation/output/finance_presentation.pptx`
- `shared/finance_presentation.pptx`

Produce a handoff note matching `schemas/handoff_schema.json`, `role: "ppt_builder"`, listing any slides that were omitted or simplified due to missing data.

---

## Quality checklist (must pass before delivery)

- [ ] Every number on every slide matches the corresponding CSV or `executive_report.md` exactly — no rounding drift, no recalculation
- [ ] The recommendation slide (Slide 3) states the same recommendation as the CFO Advisor's report, verbatim or near-verbatim
- [ ] No slide is text-dense enough that it reads as a document rather than a presentation — follow the `pptx` skill's density guidance
- [ ] Charts follow the `dataviz` skill's palette and form guidance consistently across all slides
- [ ] Any omitted slide (due to missing input data) is noted in the handoff, not silently dropped without explanation

---

## Execution guidelines

- Do not invent data to fill a slide. A missing department breakdown means a simpler cost slide, not a fabricated one.
- Do not soften or hedge the CFO Advisor's recommendation to make it sound more presentable — present it exactly as made.
- Keep narration minimal per slide; the deck supports the conversation, it doesn't replace it.

---

## Final deliverable

`finance_presentation.pptx` — a real, presentable file built from the finance team's validated outputs, plus `handoffs/ppt_builder_handoff.json`. This is the last artifact in the workflow.
