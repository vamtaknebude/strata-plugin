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
- For a "how was this calculated" question, read the `derivation` projection of `strata_read_calculation_run` with [the derivation rules](explain-derivations.md).
- `strata_read_calculation_run` with `projection: "comparison"` for the proposed Option against its Counterfactual under one Scenario, including a named Pairwise result. Do not read the full Case or Model when the supplied Run, Option, and Scenario selectors suffice. If a selector is missing, reuse an already returned exact Run summary; a needed summary read uses the selected `runId`. Ask for the missing Run or ambiguous Option or Scenario rather than inventing a selector.
- `strata_read_calculation_run` with `projection: "result_group"` when the user requests one ordered group, including its successful and failed Pairwise members. Never load both projections for one question. Summary output does not list group keys: obtain a missing group key from the exact adopted Model definition only when the returned exact Case or Model references permit that read, otherwise ask for it.
- For a "does this Run verify" question, read the `verification` projection of `strata_read_calculation_run` with [the verification rules](explain-verification.md).
