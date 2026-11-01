# Risk changes and FP&A reviews

Use the current-state rules in [`resolve-current-state.md`](resolve-current-state.md) to establish the active organization and the exact case, cycle, and revision. A supplied historical revision remains the target of a requested FP&A review.

## Change a risk

For an add, revise, or remove, resolve the current Case Revision head first. Before revising or removing a risk, call `strata_read_business_case_output` with exactly the resolved `investmentCaseId`, `decisionCycleId`, and `caseRevisionId`. Select `riskItemId` from that revision's returned `risks` section. Do not use an ID from another revision.

Show one complete command preview and wait for the member's explicit confirmation before invoking a risk write. The host must obtain this confirmation before it invokes the model-visible tool. A decline, cancellation, or correction authorizes no write; after a correction, prepare a new preview and obtain confirmation again.

- Add with `strata_add_case_risk` using `investmentCaseId`, `decisionCycleId`, `expectedCaseRevisionId`, `changeRationale`, and `risk: { risk, reason, evidence }`.
- Revise with `strata_revise_case_risk` using those case, cycle, expected-head, and rationale fields, plus the selected `riskItemId` and replacement `risk: { risk, reason, evidence }`.
- Remove with `strata_remove_case_risk` using `investmentCaseId`, `decisionCycleId`, `expectedCaseRevisionId`, `changeRationale`, and the selected `riskItemId`. Removal has no `risk` payload.

After confirmation, add one fresh `idempotencyKey` to the exact previewed input and call only the named tool. Do not add tenant, author, or provenance authority. Retry an ambiguous risk write only with the same key and byte-identical input. Preserve the returned Case Revision, Risk Item, and Risk Revision identities. If the expected head moved, read the new current state and prepare a new preview and confirmation. Stop on a concluded cycle, access failure, or other returned error.

## Request and read FP&A findings

Keep the exact `investmentCaseId`, `decisionCycleId`, and `caseRevisionId`; resolve them through the current-state rules only when the user did not supply them. Preview and confirm one request, then call `strata_request_fpa_review` exactly once with those three fields. This request has no `idempotencyKey`: do not add one and do not retry the request after an ambiguous result.

Retain the returned `reviewId` and `runId`, then call `strata_read_fpa_review` with only `{ reviewId }`. Report the returned lifecycle state:

- `running` means the review is not complete.
- `ready` means report each finding's `concern`, `importance`, and `citations`; an empty findings array is a completed review with no findings.
- `failed` means report its `failureReason`. Do not present a failure as an empty review.

If the request returns `MISSING_SUCCESSFUL_RUN`, report the identified `caseRevisionId`. Do not calculate, substitute an earlier Run, or request another review. Never use Assumption Review disposition tools for FP&A findings. For a later status question, read the same returned `reviewId`.
