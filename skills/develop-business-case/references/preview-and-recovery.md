# Ordered preview and recovery contract

Use this reference for every UC-2 mutation sequence.

## Ordered plan

Show the complete intended order, for example:

```text
1. Register one evidence item.
2. Save one assumption using the returned evidenceItemId.
3. Save one scenario using the returned assumptionId and successor case head.
```

The plan is not batch authorization. Preview and confirm step 1 only. After its receipt, substitute returned IDs into the complete step-2 preview and confirm again. Repeat for step 3. Never call a generic dispatcher and never execute an unconfirmed remainder.

Every executable preview states the tool name, complete normalized input, exact current case and subject heads, change reason, effect, and that a command UUID will be created only after confirmation. A correction invalidates the preview and its confirmation.

After confirmation, the tool call must contain every preview input field with exactly the previewed value and no additional field except the new `commandId`. Do not omit a previewed optional value or silently normalize it between preview and execution.

## Receipt discipline

- Evidence registration does not create a case revision. Preserve its evidence and source-artifact IDs.
- Assumption and scenario saves/removals each create exactly one successor case revision.
- Advance the working `expectedCaseRevisionId` only from a successful structured receipt.
- Retain stable subject IDs separately from immutable subject revision IDs.
- Never claim a step succeeded from prose, a partial result, or a transport close.

## Failure boundary

At the first failure:

1. make no second mutation call in the turn: do not repeat the failed command and do not execute a later mutation;
2. preserve and report prior committed receipts;
3. identify the stable error and the failed step;
4. do not generate command IDs for the uncommitted remainder; and
5. refresh and re-preview only the affected remainder before any later confirmation.

This stop applies even when the ordered plan was previously described. A plan is not permission to continue after an error.

## Recovery matrix

- Ambiguous or lost response: retry once with the same command ID and byte-identical input. `replayed: true` ends recovery successfully.
- Case or subject revision conflict: read the authorized current exact state; compare it with the confirmed input; show the entire changed preview; obtain fresh confirmation; use a new command ID.
- Dependency conflict: show only returned authorized dependency IDs/kinds and an explicit remediation order. Do not cascade.
- Incompatible value or non-driver: preserve the proposal, explain the exact mismatch, and ask for a compatible value or separately confirmed driver revision.
- Invalid input: identify the field; never repair it with an invented fact or implicit conversion.
- Rate limited or rate-limit unavailable: stop without retrying in the same turn; preserve the exact failed preview for a later user-confirmed attempt. A `retry: safe` disposition does not override this turn boundary.
- Command reuse: stop and re-preview with a new ID only after the user confirms the reconstructed intent.
- Authentication, organization, membership, authorization, or access failure: stop, do not retry unchanged, and do not reveal hidden case existence.
- Persistence unavailable or timeout before a definitive receipt: use the one unchanged retry rule, then stop if still ambiguous.

For a stale refresh, returned head details are remediation data, not overwrite permission. Never auto-merge concurrent changes. The refreshed preview must disclose what changed and include the new exact head before asking again.
