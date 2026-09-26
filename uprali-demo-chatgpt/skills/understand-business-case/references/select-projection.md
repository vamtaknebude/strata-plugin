# Select one projection

- `overview` for framing, alternatives, or subject counts.
- `assumptions` for assumptions or sensitivity drivers; set `driversOnly: true` only when the user asks about drivers.
- `scenario` for one known `scenarioId` and its ordered adjustments.
- `evidence_lineage` for one known `assumptionId` or `evidenceItemId`.
- `strata_read_assumption_review` for one known Review or Assumption Revision.
- `strata_read_decision` for one known Decision.
- `strata_read_outcome_review` for one known Outcome Review or Outcome Review Revision.
- `strata_read_learning` for one known Decision Learning.
- Use `strata_get_case` only when one question must cross scenarios, assumptions, and evidence or when their selection IDs are not known. Its exact full graph is broader than a focused projection, so do not use it for a single-subject question.
- For a scenario-to-evidence request that names a scenario by label and supplies no subject IDs, resolve the case with `strata_list_cases` and then call `strata_get_case` directly. Do not insert an overview or scenario projection before the full-graph read.
