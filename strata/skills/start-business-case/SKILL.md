---
name: start-business-case
description: Elicit, preview, confirm, and create an initial Uprali business case (title, objective, and the confirmed template) from a short decision brief.
---

# Start Business Case

Create one Uprali case from a brief (title and decision description) and confirmed template. Creation pins configuration, records the first revision, and lists unanswered needs; it records no framing detail, saved alternatives, drafts, evidence, assumptions, scenarios, recommendations, or decisions. Use approval tools for approvals.

Use only the user's messages and named Uprali tools. Do not use shell, filesystem, environment, web, resource-listing, or UUID tools.

When the Uprali tools are unavailable, Uprali is not connected in this client. Stop before any other step and ask the user to connect it to `https://strata-utrtwerwt.sprava.ai/api/mcp`: in Claude on the web, open the `strata` plugin's connector page and select Connect; in the Claude desktop app, open Customize, Connectors; in the Claude Code CLI, run `/mcp`; in another client, its own MCP sign-in.

Creation invariant: after chat creation via `strata_record_case_intake` succeeds, stop tool use. Report only safe receipt identifiers (`caseKey`, `caseRevisionNumber`) from `structured_content.data.receipt`; never print `authorUserId`, `changeReason`, `idempotencyKey`, or `replayed`, or invent or replay a receipt. On MCP App panel completion, acknowledge the saved case name and identity without re-confirming or calling `strata_record_case_intake` again.

## Organization

Call `strata_connection_context` first only when the active organization is unset; do not verify explicit organization and user context.

When company facts are needed or asked, call `strata_read_company_profile` first and report absent facts as missing.

The server derives the authenticated user as owner and author. Never ask for owner/author, tenancy, case, revision, configuration, or idempotency fields, or display them except post-creation `caseKey` and `caseRevisionNumber`. A sponsor is not the initial owner.

## Build the normalized draft

Read [the normalized draft rules](references/normalized-draft.md) before drafting (only `title` and `description`). Use [the intake contract](references/coherent-initial-case.md) for payloads, previews, and reporting.

## Ask one focused question at a time

Before routing, ask one single-line question with one question mark for the top missing or ambiguous gap (never bundle questions or repeat supplied facts):

1. decision the case is about (`brief.description`);
2. short case name (`brief.title`), when neither supplied nor inferable (disclose inferences).

Never ask separately why the member is starting the case or for a change reason. Keep volunteered reasons, constraints, deadlines, or alternatives inside `brief.description` within 2,000 chars, preserve supplied wording without inventing facts, leave unanswered framing unset, and note in plain language that extra details can be added after creation. Re-evaluate after each answer; do not route until both brief fields are clear.

## Route the template

When the brief is clear, call `strata_record_case_intake` once to route with `brief`, a nonblank internal `changeReason` (`1–1000` trimmed chars) behind the scenes, and a directly constructed UUID `idempotencyKey` (omit `answers`, `routing`, and case IDs). Routing creates no case.

Check `structured_content.data.routing`:

- `kind: "no-fit"`: stop without creating a case and reply in plain business language that no Uprali case type fits (no technical routing details).
- `kind: "proposal"`: confirm every proposal regardless of confidence without numeric confidence, raw `templateKey`, or `judgmentId` in user text (retain `judgmentId` and `templates` internally). Immediately call `strata_read_case_intake` with `templateKey`, `brief`, and complete `routingResult` (`structured_content.data.routing`) so App hosts render **Case name** prefilled and **Create business case** active.

## Preview and confirm

Neither `strata_connection_context` nor `strata_read_case_intake` returns an App-support flag, so rely on host panel visibility and member feedback.

**MCP App host (panel visible):** Treat the panel as the preview. Send at most a brief plain-language pointer to review or edit the case name and click **Create business case** in the panel; emit no duplicate preview or code block or ask for typed confirmation. Panel name edits need no re-route. If the member asks in chat to switch to another offered template, refresh the panel via `strata_read_case_intake` with the chosen `templateKey`, unchanged `brief`, and complete `routingResult`. On an in-chat decision-text edit, re-run routing and refresh the panel on proposal via `strata_read_case_intake`. On panel completion, acknowledge the saved case name and identity without re-confirming or re-submitting.

**Non-App host (or no panel visible):** Show one ordinary chat-text preview (never a code block) per [the preview format](references/coherent-initial-case.md#plain-language-preview): short Uprali lead, `Name:`, `Decision:`, `Type of case:` (human-readable name only), plain post-creation edit note, and `Shall I create it?` (or similar). Always omit internal schema labels, keys, confidence scores, persistence lists, and retry rules; omit option/need lists unless asked. On name or offered-template edits, re-render the complete preview without re-routing (calling `strata_read_case_intake` behind the scenes if the template changed); on decision-text edits, re-run routing and re-render. Create only after explicit confirmation of the latest preview.

**Safe stops and on-demand questions:** Cancellation or hesitation creates no case and replies in plain business language without technical routing details. Answer explicit pre- or post-creation questions about template options or absent alternatives in plain language using published option names per [option lifecycle](references/coherent-initial-case.md#intake-selection-and-lifecycle), without code blocks, persistence internals, or disabling an active panel.

## Create from chat and report

After explicit confirmation of the latest non-App preview:

1. Construct a UUID `idempotencyKey` directly.
2. Call `strata_record_case_intake` once with confirmed `brief`, internal nonblank `changeReason`, `answers: {}`, `routing: { "templateKey": "<confirmed>", "templateRoutingJudgmentId": "<judgmentId>" }`, and `idempotencyKey`.
3. Verify `structured_content.data.receipt.operation` is `case.intake.recorded` and `structured_content.data.templateKey` matches the confirmed key; otherwise do not report creation.
4. Follow [receipt rules](references/coherent-initial-case.md#report-the-creation-receipt) and [option lifecycle](references/coherent-initial-case.md#intake-selection-and-lifecycle), then stop tool use.

## Recovery

Read [recovery rules](references/recovery.md) when a Uprali call fails or the user stops.

## Scope and same-turn-read rules

Use the product for evidence-aware business-case work (drafting, delegation confirmation, approval, outcome review, explanation and export) and feedback on Uprali, its plugin and connector. Answer every other question without the product's tools and views.

Company profile facts, the tenant's own company facts used for business-case work, are in scope.

Read in the same turn before any statement about case, card, delegation, or approval status.
