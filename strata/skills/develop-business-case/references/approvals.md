# Approvals

Build the approval path with `strata_save_approval_path`, submit with `strata_submit_case_for_approval`, and record each step with `strata_record_approval_step`. Read one exact Case Revision with `strata_read_approval_path` and the board and the inbox with `strata_list_case_approvals`. Claude follows this reference against the production tools only. When a required submission tool is unavailable, say that this Uprali connection cannot submit the case.

## Concepts

- An approval path is the ordered list of steps on one case. Each step names one tenant member, a sign-off label such as "Financial Approval", and whether the step is required.
- The named member and the label are information only: any active tenant member may record on any step.
- Each published template carries a default approval path. The first save sends one member per default position, and the server takes each label and required flag from the template default. A step list whose length does not match the default is rejected.
- A submission pins one Case Revision, its selected Option Revision, one approval path revision, and the confirmed Outcome Commitments. Each Commitment names the published template success-metric key that it tracks. The member who submits is the submitter.
- A step record is an approval or a change request with text on one step. It keeps the member the step names and the member who recorded it. A comment exists only on reads.
- Final approval is the approval record that leaves every required step approved since the latest submission. It creates the `approve` Decision over the pinned Case Revision in the same transaction, carrying the submitted Option Revision and Commitments unchanged.

## Status

Approval status is read for an exact Case Revision. The status is `approved` from an `approve` Decision, `defer` or `stop` from one of those Decisions with or without a submission, `needs_revision` when a change request is the latest step record since the latest submission, `pending` when a submission awaits its completing records, and `not_submitted` otherwise. No `rejected` status exists.

Report the status returned by `strata_read_approval_path` for the exact Case Revision named. The board rows returned by `strata_list_case_approvals` carry the same derived status for each case's current revision.

## Read before each write

Resolve the case and its current Case Revision through `strata_read_case`. Read `strata_read_case_intake` for its recorded `answers` and published `successMetrics` definitions. Read `strata_read_case_draft` with the known opaque `draftId` from the current request or starting context; use its ready `document.successMetricValues` for drafted targets. If the draft ID is absent, ask for that existing reference before preparing a submission proposal. Call `strata_read_approval_path` with the exact `investmentCaseId` and `caseRevisionId` before submission. Pass its `approvalPath.approvalPathRevisionNumber` as `expectedApprovalPathRevisionNumber`, or `null` when `approvalPath` is `null` and you save the first path. Save a path before submission when none exists. Choose the user-selected alternative in the case-read overview and pass its `alternativeRevisionId` as `selectedOptionRevisionId`.

## Save the path

`strata_save_approval_path` replaces all steps with the complete ordered list you send. Take each `memberUserId` from `strata_list_organization_members`. On the first save send one member per template default position; on later saves send each member's `signOffLabel` and `required` flag as well. Show the complete ordered steps and obtain confirmation first: each member's name on the first save, plus each label and required flag on later saves. When a save is refused because the step count does not match the template path, report the refusal and ask the user for the member list; do not guess the count.

## Submit and record

Build one proposal for every published template success metric. Pair each proposed `templateSuccessMetricKey` with its drafted value and the selected Option Revision. For each Commitment, ask for any missing `indicator`, typed `expectedResult`, `evaluationPeriodStart`, `evaluationPeriodEnd`, `reviewDate`, `accountableUserId`, `actionCriteria`, and `plannedEvidenceSource`. Ask how each metric will be tracked and preserve the member's distinct data source or proposed method in that metric's `plannedEvidenceSource`. Use only active tenant members for accountable IDs.

Show the complete Commitment list and exact case, revision, option, and path selectors. Accept edits, show the revised complete list, and wait for explicit confirmation before calling `strata_submit_case_for_approval`. Use the confirmed values unchanged in its arguments. After a submitted receipt, read the same exact revision with `strata_read_approval_path` and report the captured Commitments. A refusal names missing prerequisites or invalid accountable members; resolve them and obtain a new confirmation before a fresh submit.

Read the inbox with `strata_list_case_approvals` and `inbox` set, which keeps only steps that name the viewing member and await a record.

For each step record, show the step's sign-off label and named member, the kind, and the text, and obtain confirmation first. Then send one `strata_record_approval_step` with `kind` `approval` or `change_request` and the user's text. After a change request, the case is revised and submitted again.

A press of "Approve step" in the workspace App, or of "Send change request" after the user typed the requested changes, is the user's confirmation, and the App records that step record itself. The step is recorded: report it and send no `strata_record_approval_step` for it. The App's model context names the case key, `stepId`, and `stepRecordId`, and keeps that action until the App's next action; report it once. The App's model context is optional, and a host can drop it. Before you state the approval status or any step record, call `strata_read_approval_path` for the case in the same turn, or read the board with `strata_list_case_approvals` in the same turn, and report what it returns. To find the case the user calls the new case, follow [the new-case rule](assumption-cards.md#current-card-status-and-the-new-case).

## Retry

Use a fresh `idempotencyKey` for each confirmed write. Retry an ambiguous result only with the same `idempotencyKey` and unchanged input. On `STALE_STATE`, read the path again, show the changed preview, reconfirm, and use a new key.

## Defer and stop

A case that will not proceed is recorded as a defer or stop Decision through the Decision flow, not through these approval tools. Follow [decision-review-and-learning.md](decision-review-and-learning.md) to record one of those two Decisions.
