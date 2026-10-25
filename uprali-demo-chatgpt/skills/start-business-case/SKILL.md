---
name: start-business-case
description: Elicit, preview, confirm, and create an initial Uprali business case (title, objective, and the confirmed template) from a short decision brief.
---

# Start Business Case

Create one Uprali case from a short brief (title and description) and a confirmed template. Creation records the first case revision under the confirmed template's pinned configuration and lists the template's unanswered needs. It records no framing detail, alternatives, drafts, evidence, assumptions, scenarios, recommendations, or decisions. Approvals need approval tools.

Use only the user's messages and named Uprali tools. Do not use shell, filesystem, environment, web, resource-listing, or UUID tools.

When the Uprali tools are unavailable, the Uprali demo is not connected in this client. Stop before any other step and ask the user to connect it to `https://strata-utrtwerwt.sprava.ai/api/uprali-demo/mcp`: in Claude web or the Claude desktop app, open Customize, Connectors. The demo needs no account.

When the Uprali demo serves `strata_recommend_case_intake` and `strata_build_initial_case` and both are available, follow [the demo initial-case flow](references/demo-initial-case.md) instead of the scope statement above and the rest of this file.

Creation invariant: after the creation call of `strata_record_case_intake` succeeds, stop tool use except the reference's success-metrics step when its tools exist. Copy every receipt field byte-for-byte from `structured_content.data.receipt`. Never generate, replace, reconstruct, or summarize a receipt identifier. Never call the creation call again to retrieve or replay a successful receipt.

## Organization and ownership

Call `strata_connection_context` first only when the host or conversation has not established the active organization. Do not verify explicit organization and user context.

When the case needs company facts or the user asks about them, call `strata_read_company_profile` first and report absent facts as missing.

The server derives the authenticated user as owner and author. Do not ask for or send identity, tenancy, case, revision, configuration, or user-supplied idempotency keys. A sponsor or stakeholder is not the initial owner.

## Build the normalized draft

Read [the normalized draft rules](references/normalized-draft.md) before you build or change the draft. The draft is the brief and the change reason.

Use the [input, preview, and tool-call contract](references/coherent-initial-case.md) for the exact tool fields.

## Ask one focused question at a time

Before routing, identify the missing or materially ambiguous brief and change-reason values. Ask exactly one question about the highest-priority gap. Use one single-line interrogative with exactly one question mark. Do not bundle questions or repeat supplied information.

Use this priority:

1. the decision the case is about (the brief description);
2. a short case name (the brief title), when the user's words do not already supply one;
3. the change reason.

Re-evaluate after each answer. Do not route while a required value is missing or ambiguous. Ask nothing else: the intake calls accept no other case content.

## Route the template

After the elicitation questions are answered and before the preview, make one routing call of `strata_record_case_intake`:

- `brief`: `{ "title": "<short case name>", "description": "<decision brief>" }`, both from the user's words;
- `changeReason`: the confirmed change reason;
- `idempotencyKey`: a new UUID, constructed directly.

Omit `answers`, `routing`, and every case identifier. The routing call records a template routing judgment; it creates no case.

The result's `structured_content.data.routing` is one of:

- `kind: "proposal"`: show `templateKey`, the matching template `name` from `templates`, and `confidence` (when not null) with the preview. Keep `judgmentId` and `templateKey` for the creation call.
- `kind: "no-fit"`: tell the user that no template fits and end intake. Do not create a case.

Show the proposed template for confirmation at any confidence. If the user asks for a different template, offer only entries of `templates` from this routing result, show the chosen entry, and use its `templateKey` with the same `judgmentId` in the creation call.

## Preview what creation sends and persists

Show one complete preview after a `kind: "proposal"` routing result and immediately before confirmation. Include:

- the authenticated Uprali user as owner and author;
- `changeReason`;
- the brief `title` and `description`, word for word;
- the proposed template: `templateKey`, its `name`, and `confidence` when not null;
- the "Creation persists" block of the [exact preview template](references/coherent-initial-case.md#exact-preview-template), verbatim in substance; and
- the rule that the idempotency key is generated after confirmation and an unchanged retry reuses it.

The default text is placeholder text, not the user's facts. Keep structured and readable previews equal.

## Confirm, create, and verify

Wait for explicit confirmation of the latest complete preview. Apply only a stated correction, show the complete corrected preview, and reconfirm. Cancellation or hesitation creates no case and makes no creation call.

After confirmation:

1. Construct a valid UUID directly as `idempotencyKey` for the creation call. Do not call any tool to generate it.
2. Make exactly one creation call of `strata_record_case_intake` with the confirmed `brief`, the confirmed `changeReason`, `answers: {}`, `routing` set to `{ "templateKey": "<confirmed templateKey>", "templateRoutingJudgmentId": "<kind: proposal judgmentId from the routing call>" }`, and the new `idempotencyKey`. Send exactly these properties.
3. Accept only a structured success receipt. Verify `structured_content.data.receipt.operation` is `case.intake.recorded` and `structured_content.data.templateKey` equals the confirmed template key. Copy the returned `receipt` fields exactly. If the result lacks a valid structured success receipt, do not report creation.
4. Report the exact case key and `caseRevisionNumber`. Report every entry in `structured_content.data.prefilled` as a company-profile figure that creation recorded as a key input. Report every entry in `structured_content.data.needs` as still unanswered. Offer a next task without starting it.

## Recovery

Read [the recovery rules](references/recovery.md) when a Uprali call fails or the user stops.

## Scope and same-turn-read rules

Answer only evidence-aware business-case work through the product: drafting, delegation confirmation, approval, outcome review, and explanation and export. Answer every other question without the product's tools and views.

Company profile facts, the tenant's own company facts used for business-case work, are in scope.

Read in the same turn before any statement about case, card, delegation, or approval status.
