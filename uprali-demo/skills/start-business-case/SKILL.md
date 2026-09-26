---
name: start-business-case
description: Elicit, preview, confirm, and create a coherent initial Uprali business case from a complete or incomplete decision brief.
---

# Start Business Case

Create one complete Uprali root revision with a coherent narrative and 2–25 ordered, materially distinct alternatives. Do not create drafts, evidence, assumptions, scenarios, recommendations, or decisions. Approvals need approval tools.

Use only the user's messages and named Uprali tools. Do not use shell, filesystem, environment, web, resource-listing, or UUID tools.

When the Uprali tools are unavailable, the Uprali demo is not connected in this client. Stop before any other step and ask the user to connect it to `https://strata-utrtwerwt.sprava.ai/api/uprali-demo/mcp`: in Claude web or the Claude desktop app, open Customize, Connectors. The demo needs no account.

When `strata_recommend_case_intake` and `strata_build_initial_case` are both available, follow [the demo initial-case flow](references/demo-initial-case.md) instead of the scope statement above and the rest of this file.

Creation invariant: after `strata_start_case` succeeds, stop tool use except the reference's success-metrics step when its tools exist. Copy every receipt field byte-for-byte from `structured_content.data`. Never generate, replace, reconstruct, or summarize a receipt identifier. Never call `strata_start_case` again to retrieve or replay a successful receipt.

## Organization and ownership

Call `strata_connection_context` first only when the host or conversation has not established the active organization. Do not verify explicit organization and user context.

The server derives the authenticated user as owner and author. Do not ask for or send identity, tenancy, case, revision, configuration, or user-supplied idempotency keys. A sponsor or stakeholder is not the initial owner.

## Build the normalized draft

Read [the normalized draft rules](references/normalized-draft.md) before you build or change the draft.

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

Read [the recovery rules](references/recovery.md) when a Uprali call fails or the user stops.
