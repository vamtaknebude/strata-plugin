---
name: develop-business-case
description: Create or revise calculation Models, evidence, assumptions, sensitivity drivers, scenarios, Assumption Reviews, Decisions, Outcome Reviews, Decision Learning, and exact Learning references for an existing Uprali business case. Use when a user asks to add, change, remove, or complete any of these records, or to substantiate or stress-test a case that already exists.
---

# Develop Business Case

Develop one exact Uprali case from supplied facts. Preserve provenance, revision identity, and user control. Calculate only via `strata_create_calculation_run` when available; state only values a Run returned. Do not recommend, fetch, invent, convert, or batch changes.

Use only Uprali MCP, user messages, and host-loaded references. Do not inspect the repository, environment, filesystem, web, discovery, UUID, or resource-listing tools.

When the Uprali tools are unavailable, the Uprali demo is not connected in this client. Stop before any other step and ask the user to connect it to `https://strata-utrtwerwt.sprava.ai/api/uprali-demo/mcp`: in Claude web or the Claude desktop app, open Customize, Connectors. The demo needs no account.

When `strata_build_initial_case` is available, name records in replies by case key, title, or label, not by ID (UUID).

Read every applicable reference before preparing persisted values:

- [evidence](references/evidence-and-assumptions.md) for evidence and Assumptions.
- [scenarios](references/scenarios.md) for Scenarios.
- [decision review and learning](references/decision-review-and-learning.md) for Decisions, review, and learning, including calculated Run reliance.
- [approvals](references/approvals.md), [cards](references/assumption-cards.md), [delegation](references/assumption-delegation.md), [performance](references/performance.md).
- [preview and recovery](references/preview-and-recovery.md) for preview and recovery.

## Resolve exact current state

Read [the current-state resolution rules](references/resolve-current-state.md) before you prepare any command.

## Prepare one complete command

Keep referenced concepts distinct. Connected activities are optional unless requested or required next. Any authorized organization member may act. Do not invent workflow roles or separate duties.

Use only supplied facts, authorized Uprali output, and unambiguous schema mappings. Preserve supplied wording, uncertainty, and attribution. Infer only the limited values the applicable reference permits, and disclose every inference in the preview. Never send `null` for optional evidence fields. Omit `locator`, `excerpt`, `observedAt`, or `supersedesEvidenceItemId` when no value was supplied or resolved.

Clarify missing fields, unsupported conversions, partial Evidence, incompatible adjustments, unavailable IDs, incomplete replacement sets, or unresolved dependencies. Ask one focused grammatical question about the highest-priority gap. When all registration metadata is absent, ask for source identity, capture time, citation, and disclosure together.

For removal, read the exact subject and dependencies, disclose that no cascade occurs, and use only `strata_remove_assumption` or `strata_remove_scenario` after confirmation.

## Models

Use Model, Assumption, and Model-read operations. Do not create Scenario. Read [evidence and Assumptions](references/evidence-and-assumptions.md) and [preview and recovery](references/preview-and-recovery.md) for the required staged order.

## Preview, confirm, and execute

Show the complete ordered plan. Before each write, perform its required fresh read and show one complete executable preview with exact normalized input, effect, current heads, inferences, and dependencies. Preview only the next command and wait for explicit confirmation. A correction invalidates the preview and confirmation.

After confirmation, construct a fresh lowercase RFC 4122 version 4 UUID directly in `idempotencyKey`. Its grammar is `xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx`, where `y` is `8`, `9`, `a`, or `b`. Never use a tool to generate it or reuse a key for a different command. Copy every previewed input unchanged, add only the key, and call exactly the named domain tool.

Only a structured success receipt commits. Preserve its fields and replay status, and advance heads only from it. Resolve IDs for the next preview and reconfirm; never execute an unconfirmed remainder. The normal evidence-to-driver-to-scenario path uses at most eight Uprali calls. Protected reads use bounded discriminated projections and opaque context-bound cursors; write retry rules do not apply.

Review requests and retries return queued Attempts. Use bounded polling. A timeout, refusal, malformed output, provider failure, or failed Attempt never means no material Findings. Retry a failed current Review only after explicit confirmation. Corrections append successor Dispositions or complete Outcome Review Revisions. Learning reuse names an exact revision.

## Stop and recover

Read [the stop and recovery rules](references/stop-and-recover.md) when a Uprali call fails or the user stops.
