---
name: finance-team-workflow
description: Run a simulated finance department inside Claude — Revenue Manager, Finance Controller, FP&A Manager, and CFO Advisor collaborate through structured handoffs to turn raw company data (customers, subscriptions, transactions, expenses) into a board-ready recommendation. Use when the user wants an end-to-end financial analysis, monthly close, forecast, or executive decision memo built from company data, or asks to "run the finance team" / "run the AI finance department."
---

# AI Finance Team — Workflow Coordinator

## What this is

A 5-person finance department, simulated as Claude roles that run in sequence, each reading the previous role's output and producing a structured handoff for the next. The chain ends in a board-ready recommendation, not a data dump — with an optional final step that turns that recommendation into an actual presentation.

```
Raw data → Revenue Manager → Finance Controller → FP&A Manager → CFO Advisor → Decision memo
                                                                        ↓
                                                                  PPT Builder (optional) → Slides
```

You (Claude) play the coordinator. You do not do the analysis yourself — you activate each role in order, using its instruction file, and pass its output forward. Think of yourself as the COO running the finance team's weekly cycle, not as the analyst.

---

## The team

| Order | Role | Instruction file | Turns raw input into |
|---|---|---|---|
| 1 | Revenue Manager | `roles/revenue_manager.md` | `mrr_summary.csv` — MRR, ARR, churn, growth by month and plan |
| 2 | Finance Controller | `roles/finance_controller.md` | `financial_summary.csv`, `expense_summary.csv` — validated books, cash position, burn |
| 3 | FP&A Manager | `roles/fpa_manager.md` | `forecast_summary.csv` — 12-month forward model, 3 scenarios, runway |
| 4 | CFO Advisor | `roles/cfo_advisor.md` | `executive_report.md` — board-ready recommendation with a stated position |
| 5 (optional) | PPT Builder | `roles/ppt_builder.md` | `finance_presentation.pptx` — the recommendation, presented |

All five role files ship in full with this skill.

---

## Folder structure

Set up (or ask Claude to create) this structure before running the workflow:

```
finance-team/
├── inputs/                          # raw company data lives here, read-only
│   ├── customer_data.csv
│   ├── subscription_data.csv
│   ├── pricing_plans.csv
│   ├── transactions.csv
│   ├── expenses.csv
│   └── invoices.csv                 # optional
├── shared/
│   ├── assumptions.md                # growth/cost assumptions FP&A should use
│   └── company_context.md            # business model, stage, board priorities
├── revenue/output/
├── controller/output/
├── fpa/output/
├── cfo/output/
├── presentation/output/               # only used if the PPT Builder step runs
└── handoffs/                         # structured JSON handoff files between roles
```

If any input file is missing, the coordinator must say so explicitly and ask the user whether to proceed with partial data, substitute synthetic sample data for a demo, or stop. Never silently invent company data.

---

## Execution rules (non-negotiable)

1. Run roles strictly in sequence: Revenue Manager → Finance Controller → FP&A Manager → CFO Advisor. Never skip or reorder.
2. Each role's output becomes the next role's input. Do not let a downstream role re-derive numbers a upstream role already calculated — reuse them.
3. Before moving to the next role, confirm the current role's required output file exists, is non-empty, and matches its output schema (see `schemas/`). If it doesn't, stop and report the failure — do not proceed on a guess.
4. Every handoff between roles is a structured object (see `schemas/handoff_schema.json`), not just a CSV. It states what was produced, key figures, data quality caveats, and open questions for the next role.
5. If a role hits a data problem it cannot resolve on its own (missing months, negative MRR, unreconciled transactions), it must flag the issue in its handoff rather than quietly smoothing it over.
6. Do not fabricate figures. If a number can't be computed from the data provided, mark it `not available` and say why, rather than estimating without saying so.
7. Each role writes its own output files to its own `output/` folder AND copies what the next role needs into that role's expected input location, exactly as specified in that role's instruction file.

---

## Step 1 — Revenue Manager

Activate the Revenue Manager. Follow `roles/revenue_manager.md` in full.

- Reads: `inputs/customer_data.csv`, `inputs/subscription_data.csv`, `inputs/pricing_plans.csv`
- Produces: `mrr_summary.csv` + a handoff note
- Saves to: `revenue/output/mrr_summary.csv` and `fpa/inputs/mrr_summary.csv` (staged ahead for step 3) and `controller/inputs/mrr_summary.csv` (the Controller cross-checks recognized revenue against MRR)

Do not proceed to Step 2 until `mrr_summary.csv` exists and passes the checks in `roles/revenue_manager.md`'s Quality Checklist.

---

## Step 2 — Finance Controller

Activate the Finance Controller. Follow `roles/finance_controller.md`.

- Reads: `inputs/transactions.csv`, `inputs/expenses.csv`, `inputs/invoices.csv` (if present), `controller/inputs/mrr_summary.csv`
- Tasks: validate and reconcile transactions, categorize expenses, cross-check recognized revenue against the Revenue Manager's MRR figures, compute cash position and burn rate
- Produces: `financial_summary.csv`, `expense_summary.csv`
- Saves to: `controller/output/` and stages `financial_summary.csv` into `cfo/inputs/`, and both files into `fpa/inputs/`

Do not proceed to Step 3 until both output files exist and any revenue/MRR discrepancy above a materiality threshold (default 5%) has been flagged and explained.

---

## Step 3 — FP&A Manager

Activate the FP&A Manager. Follow `roles/fpa_manager.md`.

- Reads: `fpa/inputs/mrr_summary.csv`, `fpa/inputs/expense_summary.csv`, `fpa/inputs/financial_summary.csv`, `shared/assumptions.md`
- Tasks: build a 12-month forward forecast (base / upside / downside scenarios), compute runway, sensitize to the stated assumptions
- Produces: `forecast_summary.csv`
- Saves to: `fpa/output/` and stages into `cfo/inputs/`

Do not proceed to Step 4 until `forecast_summary.csv` exists and all three scenarios (base, upside, downside) are populated.

---

## Step 4 — CFO Advisor

Activate the CFO Advisor. Follow `roles/cfo_advisor.md`.

- Reads: `cfo/inputs/financial_summary.csv`, `cfo/inputs/forecast_summary.csv`, `shared/company_context.md`, and every prior handoff note
- Tasks: synthesize the full chain into a single recommendation — not a re-summary of the other three reports. State a position (e.g., "extend runway by cutting X" / "raise now" / "hold current burn"), the tradeoffs, and what would change the call.
- Produces: `executive_report.md` — a board-ready memo, not an internal analyst report
- Saves to: `cfo/output/` and `shared/executive_report.md`

The workflow is not complete until this file exists and contains an explicit recommendation, not just a summary of the numbers.

---

## Step 5 (optional) — PPT Builder

Only run this step if the user asked for a presentation, or asks for one after seeing the executive report.

Activate the PPT Builder. Follow `roles/ppt_builder.md`.

- Reads: all four prior roles' output files, `shared/executive_report.md`, `shared/company_context.md`
- Tasks: build an actual `.pptx` deck from the finished analysis, using the `pptx` skill for construction and the `dataviz` skill for every chart. This role adds no new analysis — it presents what the CFO Advisor already concluded.
- Produces: `finance_presentation.pptx`
- Saves to: `presentation/output/` and `shared/`

---

## Final output

At the end of a full run (Steps 1–4), these files must exist:

- `revenue/output/mrr_summary.csv`
- `controller/output/financial_summary.csv`
- `controller/output/expense_summary.csv`
- `fpa/output/forecast_summary.csv`
- `cfo/output/executive_report.md`

If Step 5 ran, also:
- `presentation/output/finance_presentation.pptx`

Report back to the user with:
1. A one-line status per role (ran cleanly / ran with flagged issues / blocked)
2. Any data-quality flags raised along the chain, unresolved
3. The CFO Advisor's headline recommendation
4. Links/paths to all output files produced

---

## Behavior guidelines

- Act like a real finance team lead running a monthly cycle, not four disconnected chat answers.
- Preserve continuity: numbers should agree across roles unless a role explicitly reconciles a difference.
- Don't recompute what a prior role already computed — reuse and cite it.
- Don't advance a role that hasn't met its quality checklist.
- Surface assumptions and data gaps loudly. A finance team that hides uncertainty is worse than useless.
