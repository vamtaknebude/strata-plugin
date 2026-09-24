---
name: start-business-case
description: Elicit, preview, confirm, and create a coherent initial Uprali business case from a complete or incomplete decision brief.
---

# Start Business Case

Create one complete Uprali root revision with a coherent narrative and 2–25 ordered, materially distinct alternatives. Do not create drafts, evidence, assumptions, scenarios, recommendations, approvals, or decisions.

Use only the user's messages and named Uprali tools. Do not use shell, filesystem, environment, web, resource-listing, or UUID tools.

When the Uprali tools are unavailable, the Uprali demo is not connected in this client. Stop before any other step and ask the user to connect it to `https://strata-utrtwerwt.sprava.ai/api/uprali-demo/mcp`: in Claude web or the Claude desktop app, open Customize, Connectors. The demo needs no account.

Creation invariant: after `strata_start_case` returns `structured_content.status: success`, stop tool use. Copy every receipt field byte-for-byte from `structured_content.data` into any structured receipt. Never generate, replace, reconstruct, or summarize a receipt identifier. Never call `strata_start_case` again to retrieve or replay a successful receipt.

## Organization and ownership

Call `strata_connection_context` first only when the host or conversation has not established the active organization. Do not verify explicit organization and user context.

The server derives the authenticated user as owner and author. Do not ask for or send identity, tenancy, case, revision, configuration, or user-supplied idempotency keys. A sponsor or stakeholder is not the initial owner.

## Build the normalized draft

1. Use stated facts and unambiguous safe inferences only. You may infer a concise title, option type, and an empty list when the user asserted no items. Disclose each inference. Do not invent business facts.
2. Keep the decision question, objective, materiality, and counterfactual distinct. The user must state the expected outcome under no proposed change. Do not infer it from current operations, the materiality statement, or an alternative.
3. Preserve supplied uncertainty and attribution. Do not convert an estimate, vendor claim, or objective into an established fact, constraint, or dependency.
4. Treat the case deadline as the deadline for making the decision, not the date when an alternative must become operational. Ask for an exact `YYYY-MM-DD` decision date when a supplied relative phrase makes timing material. Ask about implementation timing only to complete or omit a partial implementation period.
5. Put cross-cutting bounds and exclusions in `constraints`. Put delivery boundaries in `scope` and reasons for consideration in `rationale`.
6. Require 2–25 alternatives that differ materially. Renamed copies, restated objectives, and equivalent mechanisms are not distinct. Do not invent defer or stop alternatives. Include them when the user explicitly proposes them.
7. Map `optionType` from the alternative's mechanism: `build` creates or owns capability; `buy` procures it externally; `pilot` runs a limited experiment; `defer` continues the current approach or postpones a new commitment; `stop` ends the activity; use `custom` only when none applies. Disclose the inferred type.
8. Obtain `label`, `optionType`, `scope`, `rationale`, `dependencies`, and `strategicEffects` for each alternative. Include implementation timing only as an exact forward-ordered pair. Otherwise omit both dates.
9. Obtain a specific nonblank `changeReason` for creating the immutable root.

Use the [coherent input, weak-input, preview, and recovery contract](references/coherent-initial-case.md) to map the conversation to the exact tool fields.

## Ask one focused question at a time

Before previewing, identify missing, contradictory, or materially ambiguous facts. Ask exactly one question about the highest-priority gap. Use one single-line interrogative with exactly one question mark. Do not bundle questions or repeat supplied information.

Use this priority:

1. decision question;
2. objective or supplied success criteria;
3. materiality and counterfactual;
4. exact decision deadline when time matters;
5. cross-cutting scope or constraints;
6. fewer than two alternatives;
7. incomplete alternative;
8. change reason.

Re-evaluate after each answer. Do not preview while a required value is missing, contradictory, or materially ambiguous.

## Preview every user-controlled persisted field

Show one complete preview immediately before confirmation. Include:

- the authenticated Uprali user as initial owner and author;
- `changeReason`;
- every narrative field: `title`, `decisionQuestion`, `objective`, `materialityStatement`, `counterfactual`, optional `deadline`, and the complete ordered `constraints` list;
- every ordered alternative with `label`, `optionType`, `scope`, `rationale`, complete `dependencies`, complete `strategicEffects`, and either both implementation dates or an explicit “not recorded” marker;
- every safe inference;
- the atomic creation of the case, cycle, alternatives, owner fact, audit/outbox records, receipt, and immutable root revision; and
- the rule that command UUID generation occurs only after confirmation and an unchanged retry reuses it.

Do not hide fields, combine alternatives, or preview omitted values. Keep structured and readable previews equal. A host preview may represent an absent optional field as `null`. Never send `null` for an optional deadline or implementation date; omit that field from the `strata_start_case` input.

## Confirm, create, and verify

Wait for explicit confirmation of the latest complete preview. Apply only a stated correction, show the complete corrected preview, and reconfirm. Cancellation or hesitation creates no UUID and calls no mutation.

After confirmation:

1. Construct a valid UUID directly as `idempotencyKey`. Do not call any tool to generate it.
2. The normal post-confirmation non-message sequence is exactly one `strata_start_case` call. Send the confirmed `changeReason`, `narrative`, ordered `alternatives`, and `idempotencyKey`. Array position defines alternative order; never send an `ordinal` property on an alternative.
3. Accept only a structured success receipt. Verify `operation: case.created`, alternative count, and receipt ordinal order. Copy the returned case, cycle, root revision, alternative revision, and replay fields exactly. If the result lacks a valid structured success receipt, do not report creation.
4. Report the exact case key and root revision. Offer a next task without starting it.

A normal journey uses at most one optional `strata_connection_context` call and one `strata_start_case` call. Do not list or read cases first.

## Recovery

- `INPUT_INVALID`: correct only from user facts, show the complete preview, reconfirm, and use a new UUID.
- `PERSISTENCE_UNAVAILABLE`, `CAPABILITY_TIMEOUT`, lost response, or ambiguous transport: retry once with byte-for-byte identical normalized input and the same UUID. Accept `replayed: true` as the existing receipt.
- `COMMAND_REUSED`: stop, reconstruct the intended draft, show a complete preview, reconfirm, and use a new UUID.
- Authentication or access failure: do not retry unchanged or disclose another organization's data.
- Stale state during a later edit: read the current exact revision, show the changed edit preview, reconfirm, and use a new UUID. Do not create another initial case.
- Cancellation before a success receipt: stop with no further tool call.

Never calculate a recommendation, claim the draft is objectively true, or convert prose quality into authorization. The server independently enforces deterministic validation and tenancy even when this skill is absent.
