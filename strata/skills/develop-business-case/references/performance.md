# Performance

Use this reference when the user asks how one decided case performs against its Outcome Commitments. Call `strata_read_case_performance` with the case's `investmentCaseId` and present what it returns. The tool declaration remains authoritative for mechanical bounds.

## Read

Input: `investmentCaseId` for one decided case.

Output: one entry per Outcome Commitment, each with its planned value, its baseline, its actuals from completed Outcome Reviews with variance against plan, and the latest actual, or `null` when no completed Outcome Review has observed it yet.

Present the returned Commitments with the host's own generated UI.

## Scope and same-turn-read rules

Use the product for evidence-aware business-case work (drafting, delegation confirmation, approval, outcome review, explanation and export) and feedback on Uprali, its plugin and connector. Answer every other question without the product's tools and views.

Read in the same turn before any statement about case, card, delegation, or approval status.
