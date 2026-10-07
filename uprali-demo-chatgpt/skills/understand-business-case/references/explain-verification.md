# Explain exact retained verification

Use this reference when the user asks whether a retained Run verifies, for
example "does this Run verify". Answer only from the `verification`
projection of `strata_read_calculation_run`. Never invent outcomes, counts,
or identities the tool did not return.

## Select the verification

- Always resolve the case first: call `strata_list_cases` with `{ "limit": 25 }`,
  match the case title, then call `strata_read_case` for the current overview.
  Read the verification only after the overview.
- Then read the verification in one call. Do not read the summary, evaluation,
  comparison, or derivation projection first.
- Call `strata_read_calculation_run` with `projection: "verification"` and the
  exact `runId`, for example `{ "projection": "verification",
"runId": "<uuid>" }`.
- Take `runId` from returned results, such as the `strata_read_case` case
  reference or a prior calculation read. Never invent it.

## Distinguish the outcomes

State the returned status word first, for example "status: match".

- `match` on a successful Run: state that recomputation reproduces the retained
  success, quote the exact `runId`, and name the reconstructed scope.
- `match` on an unsuccessful Run: state that recomputation reproduces the
  retained failure, quote the exact `runId` and the retained diagnostic codes,
  and state that the Run still matches. Never use the word "mismatch" for a
  matching Run.
- `mismatch`: state that recomputation disagrees with the retained output and
  quote `VERIFICATION_NOT_MATCHED`. Never describe it as a retained
  calculation failure and never name a failed item as its cause.
- `unavailable`: quote the exact reason (`exact_source_unavailable` or
  `evaluator_unavailable`), name the missing dependency the reason identifies,
  and stop. Never guess which source is missing beyond the returned reason.
- `execution_failed`: quote the exact code (`CALCULATION_LIMIT_EXCEEDED` or
  `VERIFICATION_EXECUTION_FAILED`) and stop.

## Copy exact identities

- Copy `runId`, `caseRevisionId`, `modelRevisionId`, `investmentCaseId`, and
  `evaluatorContractKey` verbatim.

## Stay read-only

Never recommend an option, repair a model, create a replacement Run, or write
anything. Never advise repair, and never present a next step as a fact about
the Run. A mismatch, unavailable result, or execution failure is a statement
about this verification attempt only.
