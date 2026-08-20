# AI Finance Team Workflow

A Claude skill that simulates a 4-person finance department — Revenue Manager, Finance Controller, FP&A Manager, and CFO Advisor — collaborating through structured handoffs to turn raw company data into a board-ready recommendation.

```
Raw data  →  Revenue Manager  →  Finance Controller  →  FP&A Manager  →  CFO Advisor  →  Decision memo
```

## Status

This release ships the coordinator (`SKILL.md`) and the first role in full (`roles/revenue_manager.md`), plus the schemas and folder contracts the whole chain depends on. `finance_controller.md`, `fpa_manager.md`, and `cfo_advisor.md` are not yet written — see [`templates/role_file_template.md`](templates/role_file_template.md) to author them, or ask Claude to draft them following the same pattern as the Revenue Manager file.

## Why structured handoffs

Most multi-step prompt chains pass a plain-text summary from one step to the next and hope the next step reads it correctly. This workflow instead defines:

- An exact **output schema** per role (`schemas/*.json`) — so a role's CSV output is checkable, not just plausible-looking
- A **handoff object** per role (`schemas/handoff_schema.json`) — status, headline, key figures to reuse, data-quality flags, and open questions, separate from the data file itself
- A **quality checklist** per role that must pass before the coordinator advances to the next role

This is what keeps a 4-step chain from silently drifting — by step 4, small unflagged assumptions from step 1 tend to compound into a recommendation nobody would sign off on if they saw the whole path.

## Setup

1. Install this skill in Claude/Cowork.
2. Create the working folder structure described in `SKILL.md` under "Folder structure," with your company's `inputs/` data.
3. Fill in `shared/assumptions.md` (growth/cost assumptions for FP&A) and `shared/company_context.md` (business stage, board priorities, what decision is actually being asked).
4. Ask Claude to "run the finance team workflow" (or invoke this skill directly).

No sample data ships with this skill — bring your own `customer_data.csv`, `subscription_data.csv`, `pricing_plans.csv`, `transactions.csv`, and `expenses.csv`.

## Folder structure

See `SKILL.md` → "Folder structure" for the full layout the coordinator expects.

## Extending

To complete the team, write the three remaining role files using `templates/role_file_template.md` and the pattern in `roles/revenue_manager.md`:

- `roles/finance_controller.md` — validates transactions/expenses, reconciles against Revenue Manager's MRR, computes cash position and burn
- `roles/fpa_manager.md` — builds the 12-month forecast (base/upside/downside) from the Controller's and Revenue Manager's outputs
- `roles/cfo_advisor.md` — synthesizes everything into one recommendation with a stated position

Keep file paths in each new role file in sync with the corresponding Step in `SKILL.md`.

## License

MIT — use, adapt, and share freely.
