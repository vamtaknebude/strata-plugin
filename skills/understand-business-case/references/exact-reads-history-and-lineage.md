# Exact reads, history, and lineage

Use this reference for the precise Strata read-tool sequence. Tool results use a success envelope with operation data under `structuredContent.data`; error results provide a stable error code and retry disposition.

Case facts come only from the Strata MCP results described here. Do not inspect a repository, working directory, environment, or public web to supplement them.

If the active organization is not already present in host or conversation state, call `strata_connection_context` before every tool sequence below. It must be the first Strata call, never a later check after a list or read. Do not call it more than once in a normal journey.

## Tool inputs

### Read review, decision, and learning records

Use `strata_read_assumption_review` when the user names or supplies a visible `reviewId` or `assumptionRevisionId`. The input `reference` is either `{ "kind": "review", "reviewId": "<uuid>" }` or `{ "kind": "assumption_revision", "assumptionRevisionId": "<uuid>" }`. Include `limit` from 1 through 20. Add a top-level `cursor` only when Strata returned `nextCursor` for the same reference and limit. For Disposition history, preserve the same Review reference and top-level `limit`; send `dispositionHistory.findingId`, a nested `limit` from 1 through 10, and the returned `dispositionHistory.nextCursor` as the next `dispositionHistory.cursor`.

Use `strata_read_decision` only with a visible `decisionId`. The result is one immutable Decision. It includes the exact Case Revision, outcome, rationale, selected revisions, non-calculated declaration, Conditions, disagreements, and Outcome Commitments.

Use `strata_read_outcome_review` when the user names or supplies a visible `outcomeReviewId` or `outcomeReviewRevisionId`. The input `reference` is either `{ "kind": "review", "outcomeReviewId": "<uuid>" }` or `{ "kind": "review_revision", "outcomeReviewRevisionId": "<uuid>" }`. Use `limit: 1`. Preserve returned cursors only for the same reference and exact question.

When `strata_read_approval_path` is available and the user asks for a case's Decision or Outcome Review without a visible ID, resolve the case and call `strata_read_approval_path` with its exact `investmentCaseId`. Pass the returned `decisionId` to `strata_read_decision`. Pass the returned `outcomeReviewReference` unchanged as `reference` to `strata_read_outcome_review`. When either value is `null`, state that the case has no Decision or no Outcome Review. When `strata_read_approval_path` is not available, this rule does not apply.

Use `strata_read_learning` only with a visible `decisionLearningId`. Include `limit` from 1 through 100 and a returned cursor only for that same Learning. The result includes content, source Review Revision, Observations, lifecycle, source status, target summary, and bounded explicit references.

Protected read cursors are opaque and context-bound. Do not edit them, decode them, store them as durable identifiers, use them with another reference, or use them after a write changes the current state being explained.

### Resolve a case

Call `strata_list_cases` with `{ "limit": 25 }`. A returned case contains `caseKey`, `title`, `investmentCaseId`, `decisionCycleId`, and its current `caseRevisionId`. Reuse `nextCursor` only to read the next list page.

Do not use a title or case key as a substitute for UUID inputs. If title matching produces multiple candidates, ask the user to choose from the returned candidates.

When a single question must join scenario adjustments to their assumptions and evidence, call `strata_get_case` with both case IDs. Omit `caseRevisionId` only to resolve the current full graph, then retain the returned exact `caseRevisionId`. Prefer the focused reads below for every question that does not require this cross-subject join.

### Read one projection

Every `strata_read_case` input contains `investmentCaseId`, `decisionCycleId`, and exactly one projection. Include `caseRevisionId` for exact reads; omit it only to resolve current state.

Projection shapes:

```json
{ "kind": "overview" }
{ "kind": "assumptions", "driversOnly": false, "limit": 20 }
{ "kind": "scenario", "scenarioId": "<uuid>", "limit": 20 }
{ "kind": "evidence_lineage", "assumptionId": "<uuid>", "limit": 20 }
{ "kind": "evidence_lineage", "evidenceItemId": "<uuid>", "limit": 20 }
```

Use only one of `assumptionId` or `evidenceItemId`. A page continuation adds the returned `cursor` and retains the same exact `caseRevisionId`, projection kind, selection ID, and limit.

The result's `caseReference` identifies the selected exact revision, current revision, optional parent revision, case IDs, case key, author, recorded time, and change reason. `resolvedFromCurrent: true` means the server resolved an omitted revision ID; it does not permit a later unpinned read.

### Read history

Call `strata_list_case_revisions` with both case IDs and a `limit` from 1 through 25. A history item identifies the immutable `revisionId`, `revisionNumber`, `operation`, author, and recorded time. Visible operations include a change reason and, when applicable, exact subject IDs. A restricted assumption history item may expose only `access: masked` and `operation: case.assumption.changed`.

History order is not a parent pointer. Use `caseReference.parentCaseRevisionId` from an exact projection to determine “previous.”

### Compare exact revisions

Call `strata_compare_case_revisions` with both case IDs, `fromCaseRevisionId`, `toCaseRevisionId`, and a `limit` from 1 through 25. Preserve all five fields when adding a returned cursor.

For “what changed since the previous revision,” first read the current `overview`. If its `parentCaseRevisionId` is present, call the comparison directly with that parent as `fromCaseRevisionId` and the overview's `caseRevisionId` as `toCaseRevisionId`. Do not call `strata_list_case_revisions`; the overview already supplies the immutable parent edge, and history order is not a parent pointer. If no parent ID is present, the selected revision is a root and has no previous revision to compare.

Narrative comparison items return `changedFields` and exact from/to lineage. Alternative, assumption, and scenario items return a stable subject ID, `added`, `changed`, or `removed`, and available exact subject-revision IDs and lineage. Masked assumption changes disclose no IDs or lineage.

Summarize every comparison item. In particular, enumerate every narrative `changedFields` value (for example, both `objective` and `deadline`) rather than inferring a single headline from the first changed field. When the selected current overview contains the field, include that exact current value beside the field name. This uses data already returned; do not add a parent-overview read merely to obtain the old value.

## Read and pin exact revisions

These rules are steps 5 and 6 of the Read workflow in `SKILL.md`.

- Read: Call `strata_read_case` for a focused case projection. For an Assumption Review, Decision, Outcome Review, or Decision Learning read, call the named protected read tool from [the projection selection rules](select-projection.md) and apply the exact read guidance in [Read review, decision, and learning records](#read-review-decision-and-learning-records). Do not add `caseRevisionId` to those protected-read inputs. Record exact case revision fields only when a protected read returns them. When `strata_get_case` is necessary, record its returned `caseRevisionId` and pin any follow-up case read to it.
- Pin: Pin every follow-up `strata_read_case` call to the recorded `caseRevisionId`. For history, preserve `investmentCaseId`, `decisionCycleId`, and any returned cursor. For comparison, use exact `fromCaseRevisionId` and `toCaseRevisionId` values selected from returned fields. For protected-read pagination, preserve the same record reference and top-level limit. When the read includes Disposition history, also preserve `dispositionHistory.findingId` and its nested limit, and map each returned `nextCursor` to the corresponding input `cursor`. Never send `caseRevisionId` to `strata_read_decision`, `strata_read_outcome_review`, `strata_read_learning`, or `strata_read_assumption_review`. Never merge results from different revisions into an unlabeled answer.

## Interpretation rules

- Label the selected revision before presenting facts. If `caseRevisionId` differs from `currentCaseRevisionId`, call it historic or superseded.
- Keep revision-local identifiers together. Do not attach evidence read from one revision to an assumption or scenario read from another.
- For a scenario, order adjustments by returned `ordinal`. Do not apply the adjustments or calculate an outcome.
- For a scenario-to-evidence trace, state the complete returned chain: scenario label and ID; adjustment operator/value; assumption label, ID, and revision ID; evidence role, title or citation, source URI, and evidence item ID. Each link is part of the answer even when the user names only the endpoints.
- For evidence, preserve source kind, citation, role, captured/observed/recorded times, and supersession separately. `supersedesEvidenceItemId` is a lineage edge, not deletion of the older item.
- Counts describe records in Strata; they do not establish completeness or correctness.

## Restricted data and failures

Treat masked values as an authorization boundary. Repeat only returned mask labels, field labels/keys, ordinals, roles, and counts. Do not reconstruct masked IDs or values from comparison, history, scenario, or evidence output.

Use stable errors literally:

- `INPUT_INVALID` or `MALFORMED_CURSOR`: correct the rejected shape or restart pagination against the same exact revision.
- `NOT_FOUND`: the referenced case, revision, scenario, assumption, or evidence item is unavailable in the active organization.
- `CASE_NOT_FOUND`: the protected fact is missing or inaccessible. Stop on that request. Do not infer whether the record exists, test alternate identifiers, fall back to case resolution, or name hidden relationships.
- `RESOURCE_UNAVAILABLE`: the protected fact is missing or inaccessible. Do not infer whether it exists, test alternate identifiers, or disclose hidden relationships.
- `AUTHORIZATION_UNAVAILABLE`: access could not be verified; do not claim the record does not exist.
- `PERSISTENCE_UNAVAILABLE` or `CAPABILITY_TIMEOUT`: a read may be retried unchanged once; never switch revision IDs as a recovery tactic.
- `CAPABILITY_CANCELLED`: stop unless the user asks to try again.
