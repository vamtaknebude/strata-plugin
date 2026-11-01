# Approvals

For submission, use `strata_read_approval_path` and `strata_submit_case_for_approval`; use `strata_save_approval_path` when the case needs a path. A missing `strata_record_approval_step` does not block submission. When a required submission tool is unavailable, say that this Uprali connection cannot submit the case.

## Concepts

- An approval path is the ordered list of steps on one case. Each step names one Organization member, a sign-off label such as "Financial Approval", and whether the step is required.
- A submission pins one Case Revision, its selected Option Revision, one approval path revision, and the confirmed Outcome Commitments. Each Commitment names the published template success-metric key that it tracks. The member who submits is the submitter.
- A step record is an approval, a change request, or a comment with text on one step. It keeps the member the step names and the member who recorded it. Any member may record on any step.
- Approval status is read for an exact Case Revision. Without a Decision or submission for that revision, it is `not_submitted`; a submission without a later change request is `pending`. Report the status returned by `strata_read_approval_path`.
- An approval path never blocks or changes a Decision. When the user asks to record a Decision, record it with `strata_create_decision` whatever the approval status.

## Read before each write

Resolve the case and its current Case Revision through `strata_read_case`. Read `strata_read_case_intake` for its recorded `answers` and published `successMetrics` definitions. Read `strata_read_case_draft` with the known opaque `draftId` from the current request or starting context; use its ready `document.successMetricValues` for drafted targets. If the draft ID is absent, ask for that existing reference before preparing a submission proposal. Call `strata_read_approval_path` with the exact `investmentCaseId` and `caseRevisionId` before submission. Pass its `approvalPath.approvalPathRevisionNumber` as `expectedApprovalPathRevisionNumber`, or `null` when `approvalPath` is `null` and you save the first path. Save a path before submission when none exists. Choose the user-selected alternative in the case-read overview and pass its `alternativeRevisionId` as `selectedOptionRevisionId`.

## Save the path

`strata_save_approval_path` replaces all steps with the complete ordered list you send. Take each `memberUserId` from `strata_list_organization_members`. Show the complete ordered steps, with each member's name, label, and required flag, and obtain confirmation first.

## Submit and record

Build one proposal for every published template success metric. Pair each proposed `templateSuccessMetricKey` with its drafted value and the selected Option Revision. For each Commitment, ask for any missing `indicator`, typed `expectedResult`, `evaluationPeriodStart`, `evaluationPeriodEnd`, `reviewDate`, `accountableUserId`, `actionCriteria`, and `plannedEvidenceSource`. Ask how each metric will be tracked and preserve the member's distinct data source or proposed method in that metric's `plannedEvidenceSource`. Use only active Organization members for accountable IDs.

Show the complete Commitment list and exact case, revision, option, and path selectors. Accept edits, show the revised complete list, and wait for explicit confirmation before calling `strata_submit_case_for_approval`. Use the confirmed values unchanged in its arguments. After a submitted receipt, read the same exact revision with `strata_read_approval_path` and report the captured Commitments. A refusal names missing prerequisites or invalid accountable members; resolve them and obtain a new confirmation before a fresh submit.

For each step record, show the step's sign-off label and named member, the kind, and the text, and obtain confirmation first. Then send one `strata_record_approval_step` with `kind` `approval`, `change_request`, or `comment` and the user's text.

A press of "Approve step" in the workspace App, or of "Send change request" after the user typed the requested changes, is the user's confirmation, and the App records that step record itself. The step is recorded: report it and send no `strata_record_approval_step` for it. The App's model context names the case key, `stepId`, and `stepRecordId`, and keeps that action until the App's next action; report it once. The App's model context is optional, and a host can drop it. Before you state the approval status or any step record, call `strata_read_approval_path` for the case in the same turn and report what it returns. State Assumption Card status only from the `assumptionCardStatusCounts` of that read, or from `strata_list_assumption_cards` called in the same turn; never from an earlier card read. To find the case the user calls the new case, follow [the new-case rule](assumption-cards.md#current-card-status-and-the-new-case).

## Retry

Use a fresh `idempotencyKey` for each confirmed write. Retry an ambiguous result only with the same `idempotencyKey` and unchanged input. On `STALE_STATE`, read the path again, show the changed preview, reconfirm, and use a new key.
