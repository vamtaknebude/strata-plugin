# Assumption delegation

Use this reference when the user asks to delegate a prospective or existing Case Assumption to a named Organization member, or to act on such a delegation: revise the request, find assigned or created work, save or submit a Response, return a Submission for correction, cancel a request, or inspect history.

Use only these nine catalogued tools: `strata_list_organization_members`, `strata_create_assumption_delegation`, `strata_update_assumption_delegation_request`, `strata_list_assumption_delegations`, `strata_read_assumption_delegation`, `strata_save_assumption_delegation_response`, `strata_submit_assumption_delegation_response`, `strata_return_assumption_delegation_submission`, and `strata_cancel_assumption_delegation`. Copy every identifier byte-for-byte from a Strata receipt into its matching ID field. Read [preview and recovery](preview-and-recovery.md) before any mutation: every delegation write follows that ordered plan, preview, confirmation, one-mutation, receipt, and recovery contract, and this reference states only the delegation-specific inputs and read sequences.

## Concepts

A delegation asks one Organization member, the assignee, to supply the Response for one Assumption. A request revision is one immutable version of the ask; a Response revision is one immutable draft or submitted answer under one request revision; a Submission is the submitted Response revision awaiting adoption; a Return sends a Submission back for correction; a Cancellation closes an open request. History keeps each entry distinct: drafts, Submissions, Returns, and the Cancellation each appear as their own ordered entries.

Any active member of the Organization may perform every operation: create, revise, save, submit, return, cancel, list, and read. Assignment controls responsibility, not access. A member who is neither the requester nor the assignee may still act. Show Response and Evidence content exactly as Strata returns it; never mask content from another member of the same Organization.

## Resolve the assignee

Resolve the assignee before any other delegation call. Call `strata_list_organization_members` with the supplied name as `query`, which matches as a case-insensitive literal substring of display name or email. When the page holds several matches, present every match with its display name and email and wait for the user to select one. When a `nextCursor` is returned, continue with the same `query` to see the rest. Never invent a User ID, never ask the user for an internal ID, and never place a name where an ID field belongs. The selected member's returned `userId` is the `assigneeUserId`.

## Create or revise the request

Build the request only from returned identifiers: the Decision Cycle, the shared or Option scope, the Assumption key for a prospective Assumption or the exact current Assumption Revision for an existing one, the optional exact Model Item target, a nonblank question, and an optional due date. For a prospective Assumption the target names its key and scope; for an existing Assumption the target names its current Assumption Revision. A Model Item target names the exact Model Item definition. A revision keeps the original assignee: `strata_update_assumption_delegation_request` takes no assignee field, and a revised request requires a new Submission before adoption.

Preview the complete normalized request with `strata_create_assumption_delegation` or `strata_update_assumption_delegation_request` as the named tool, obtain confirmation, then call exactly once with a fresh key. Preserve the receipt's delegation and request revision IDs; they are the heads for the next step.

## Discover work and read history

Find assigned work with `strata_list_assumption_delegations` and `view` `assigned_to_me`, created work with `view` `created_by_me`, and pending Submissions with `view` `awaiting_adoption`, optionally bounded by case or lifecycle. These reads need no confirmation and are safe to retry. Inspect one delegation with `strata_read_assumption_delegation`, which returns the current state with its lifecycle, available actions, blockers, and assignee status, plus the newest-first Request history and the selected Request's Response history. Start without a cursor to refresh; keep a cursor only to continue its page.

## Save and submit a Response

Save incomplete or complete scalar and series drafts with `strata_save_assumption_delegation_response` against the exact current request head. A scalar draft carries the typed value when known; a series draft carries one point per requested relative-period key in requested coverage order, each point holding one compatible value or one Not Applicable reason. Name confidence, rationales, the sensitivity designation, and the ordered Evidence links when the user supplies them. Submit only complete content with `strata_submit_assumption_delegation_response`: one complete typed scalar value, or exact complete series coverage with a value or a reason for every point, plus confidence with its rationale, a general rationale, an explicit sensitivity designation, and 1-50 ordered Evidence links. Anything less reports `INCOMPLETE_RESPONSE` with no write. Preview the exact Request and Response heads, obtain confirmation, then call once.

## Return a Submission

Return the exact current Submission with `strata_return_assumption_delegation_submission`, a nonblank reason, and the exact Request, Response, and Submission heads. The returned Submission keeps its content, author, submitter, Evidence order, and history, but it can never be adopted. Correction requires a successor Response draft and another explicit Submission. Preview all three heads and the reason, obtain confirmation, then call once.

## Cancel a request

Cancel an open request with `strata_cancel_assumption_delegation` at its exact current Request head and a nonblank reason. Preview the delegation, its head, and the reason, obtain confirmation, then call once.

## Preserve value distinctions

Keep missing, zero, and Not Applicable distinct. An omitted draft field or series point is missing content, never a zero and never a reason. A numeric zero is an explicit `"0"` value. A Not Applicable point carries `notApplicableReason` and no value; a point must never carry both. Keep the requested value kind, the `count` or `percent` unit, and the series currency, which the series states once while a point never names its own currency. Cover a series with exactly one point per ordered requested key; omitted periods do not carry forward. Keep Evidence links unique, ordered, and each labeled with one Evidence role: `supporting`, `contradicting`, `contextual`, or `superseding`. A draft link carries only its evidence item and its role. Keep draft, Submission, Return, and Cancellation entries distinct in history with unchanged kind, unit, currency, series points, and Evidence order.

## Recovery

- Invalid draft: identify the exact field and ask for a correction. Never invent a repair or convert the value.
- Incomplete Submission: report `INCOMPLETE_RESPONSE` and make no mutation call.
- Stale head: read fresh state with `strata_read_assumption_delegation`, compare it with the confirmed input, show the entire changed preview, obtain fresh confirmation, and use a new key.
- Inactive assignee or concluded Cycle: stop with the authorized error. Do not change the target or reroute the request.
- Ambiguous transport: retry once with the same key and byte-identical input. `replayed: true` ends recovery successfully.
- Changed input with a used key: stop on `COMMAND_REUSED`, reconstruct the command, preview, reconfirm, and use a new key. Never issue a second mutation after a definitive error.
