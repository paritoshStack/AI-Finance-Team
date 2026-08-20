# Company Context

Read by the CFO Advisor (`roles/cfo_advisor.md`) to calibrate the recommendation, and by the PPT Builder (`roles/ppt_builder.md`) for the title slide. Also referenced by the FP&A Manager for tone, though the numbers in `shared/assumptions.md` drive the actual forecast.

Fill in every bracketed value before running the workflow. This file shapes how the final recommendation is framed — it does not change the underlying math.

---

## Company basics

- Company name: [add your own]
- Business model: [add your own, e.g. B2B SaaS, marketplace, usage-based platform]
- Stage: [add your own, e.g. pre-seed, seed, Series A, Series B, profitable/bootstrapped]
- Primary customer segment: [add your own, e.g. SMB, mid-market, enterprise]

---

## Current phase

- Growth vs. efficiency priority right now: [add your own, e.g. "prioritizing growth over margin," "focused on extending runway," "targeting profitability this fiscal year"]
- Last fundraise (amount, round, date), if applicable: [add your own, or write "not applicable — bootstrapped / profitable"]

---

## The decision this cycle needs to answer

This is the single most important field in this file — the CFO Advisor's recommendation is framed around it.

- What decision is the board or leadership team actually trying to make right now: [add your own, e.g. "decide whether to raise a bridge round or extend runway through cuts," "approve or reject the FY[X] hiring plan," "no specific decision — standing quarterly review"]
- Timeline this decision needs to be made by: [add your own, e.g. "before the next board meeting on [date]"]

---

## Board priorities and constraints

- Stated constraints from the board or investors: [add your own, e.g. "no further dilution before 18 months of runway," "hold headcount flat until Q3"]
- Metrics the board is currently most focused on: [add your own, e.g. net revenue retention, gross margin, runway]
- Anything explicitly off the table this cycle: [add your own, e.g. "not considering layoffs," "not raising before [month]"]

---

## Notes for whoever fills this in

- Be specific in "The decision this cycle needs to answer" — a vague answer here produces a vague recommendation from the CFO Advisor. A blank answer defaults to the most natural reading of the numbers (is the current trajectory financeable, and what should change if not).
- This file is qualitative by design — no financial figures belong here, those live in `shared/assumptions.md` and the `inputs/` data.
- Update this file whenever the board's priorities shift, even if the underlying data hasn't changed — the same numbers can justify different recommendations depending on what the board actually asked for.
