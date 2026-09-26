# Approvals

Use this reference only when `strata_save_approval_path`, `strata_submit_case_for_approval`, `strata_record_approval_step`, and `strata_read_approval_path` are available. When any of these four tools is missing, record no approval data. If the user asks about approvals, say that this Strata connection cannot record them, and continue with the rest of the request.

## Concepts

- An approval path is the ordered list of steps on one case. Each step names one Organization member, a sign-off label such as "Financial Approval", and whether the step is required.
- A submission presents the case to the approvers with a summary, the budget, the NPV, and the payback in months. The member who submits is the submitter.
- A step record is an approval, a change request, or a comment with text on one step. It keeps the member the step names and the member who recorded it. Any member may record on any step.
- The approval status is derived. A Decision outcome of reject gives `rejected`, approve or pilot gives `approved`, and defer or stop shows that outcome. Without a Decision, a case with no submission is `not_submitted`. Otherwise the latest step record made after the latest submission decides: a change request gives `needs_revision`, and anything else gives `pending`. A new submission clears an earlier change request.
- An approval path never blocks or changes a Decision. When the user asks to record a Decision, record it with `strata_create_decision` whatever the approval status.

## Read before each write

Call `strata_read_approval_path` with the exact `investmentCaseId` before each approval write. Pass its `approvalPath.approvalPathRevisionNumber` as `expectedApprovalPathRevisionNumber`, or `null` when `approvalPath` is `null` and you save the first path. When `approvalPath` is `null`, save a path before you submit or record a step. Record on a step with a `stepId` from that read.

When `strata_build_initial_case` is available, read the approval path before you offer to submit, and save one first when it is `null`.

## Save the path

`strata_save_approval_path` replaces all steps with the complete ordered list you send. Take each `memberUserId` from `strata_list_organization_members`. Show the complete ordered steps, with each member's name, label, and required flag, and obtain confirmation first.

## Submit and record

Send `budget` and `npv` in the Money form `{ "kind": "money", "amount": "<decimal>", "currencyCode": "<code>" }`, with the user's figures. Send `paybackMonths` as a whole number of months, or `null` when the user gives none. Show the summary and figures and obtain confirmation first.

For each step record, show the step's sign-off label and named member, the kind, and the text, and obtain confirmation first. Then send one `strata_record_approval_step` with `kind` `approval`, `change_request`, or `comment` and the user's text.

## Retry

Use a fresh `idempotencyKey` for each confirmed write. Retry an ambiguous result only with the same `idempotencyKey` and unchanged input. On `STALE_STATE`, read the path again, show the changed preview, reconfirm, and use a new key.
