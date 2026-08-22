---
name: develop-business-case
description: Qualify evidence, create or revise typed assumptions and sensitivity drivers, and create, revise, or remove Strata scenarios when a user asks to develop, substantiate, stress-test, or update an existing business case.
---

# Develop Business Case

Turn user-supplied evidence into complete typed assumptions and explicit scenario overlays on one exact Strata case. Preserve evidence provenance, immutable revision identity, and the user's control over every mutation. Do not calculate an outcome, recommend an option, fetch a source, invent evidence, silently convert a value, or batch several domain changes into one generic command.

Use Strata MCP as the only case system. Do not run `pwd`, `ls`, `find`, `rg`, `env`, UUID utilities, interpreters, or other discovery or data commands. After the host loads this skill and its references, execute no shell command. Do not inspect the repository, working directory, process environment, or public web. Do not call `list_mcp_resources` or `list_mcp_resource_templates`.

Read these references before preparing their corresponding persisted values:

- [evidence and assumptions](references/evidence-and-assumptions.md) for evidence qualification, disclosure, value kinds, periods, ownership, confidence, evidence roles, and driver designation;
- [scenarios](references/scenarios.md) for scenario kinds, ordered adjustment operators, and exact driver/value compatibility; and
- [preview and recovery](references/preview-and-recovery.md) for command-by-command confirmation, exact-head advancement, retries, stale refresh, destructive dependencies, and stop behavior.

## Resolve one exact case

1. If the host has not established the active Strata organization, call `strata_connection_context` first. Never use client, model, plugin, skill, or environment metadata as authority.
2. Resolve a case title with `strata_list_cases`. Wait for that result before constructing a dependent read; never call list and read in parallel. If more than one returned case plausibly matches, show bounded choices and wait for selection; never guess or read an unselected case.
3. Read the selected current case with `strata_read_case` and retain `investmentCaseId`, `decisionCycleId`, `caseRevisionId`, and `caseRevisionNumber`. Use an overview when the alternatives matter. Read the assumptions projection before revising or removing an existing assumption or targeting an existing driver. Do not add that read when creating a new assumption or when a scenario targets a driver created and receipted in this conversation. Read evidence lineage only for a selected existing assumption or evidence item.
4. Treat the returned `currentCaseRevisionId` and every visible subject revision ID as the only usable edit heads. Never reuse a historic head as if it were current.

Use `strata_read_case` for every case-content read. Never call the legacy broad `strata_get_case` compatibility tool in this workflow.

Construct a read only from completed Strata output. Map the selected list item fields to the same-named `investmentCaseId`, `decisionCycleId`, and `caseRevisionId` read arguments. Then add exactly one projection:

- overview: `projection.kind` is `overview` and the projection has no other field;
- assumptions: `projection.kind` is `assumptions`, `limit` is `25`, and `driversOnly` is `false`;
- scenario: `projection.kind` is `scenario`, `scenarioId` is the selected visible scenario's stable UUID, and `limit` is `25`;
- assumption evidence lineage: `projection.kind` is `evidence_lineage`, `assumptionId` is the selected visible assumption's stable UUID, and `limit` is `25`; or
- evidence-item lineage: `projection.kind` is `evidence_lineage`, `evidenceItemId` is the selected visible evidence item's stable UUID, and `limit` is `25`.

These are field mappings, not literal values. Never construct a dependent read until the source call has returned every required UUID.

The user does not supply organization, tenant, case, cycle, revision, evidence, assumption, scenario, author, or command identifiers. Resolve them only from authorized Strata output. Copy every returned UUID byte-for-byte into its matching ID field; never place `caseKey`, a revision number, label, or another subject's UUID into an ID field. `strata_save_assumption` assigns the authenticated Strata user as owner at the server boundary and does not accept `ownerUserId`; disclose that owner in the preview but never send an owner ID. If the user requests a different named owner, stop and explain that the current tool cannot safely assign that person.

## Build a complete ordered plan

Keep these concepts distinct:

- source metadata identifies a URL or manual statement and when it was captured;
- an evidence item records citation, disclosure, and optional locator, excerpt, observed time, or supersession;
- an assumption records one current typed value and why it should be used;
- a sensitivity driver is an assumption whose `isSensitivityDriver` flag is true;
- a scenario is one complete named overlay; and
- an adjustment changes one visible current driver by stable assumption ID.

Extract only supplied facts and unambiguous schema mappings. Preserve exact wording for citations, descriptions, rationale, and labels. You may infer an evidence role, scenario kind, empty optional metadata, or shared scope only when the user's language and authorized case output make the inference unambiguous; disclose it in the next preview. Always disclose that the server will assign the authenticated Strata user as the assumption owner.

Never send `null` for optional evidence fields. Omit `locator`, `excerpt`, `observedAt`, or `supersedesEvidenceItemId` when no value was supplied or resolved.

Before previewing, reject or clarify:

- a source without capture time, citation, or disclosure;
- required evidence that is absent, inaccessible, or merely a URL the assistant has not been authorized to fetch;
- an estimate that is not labeled as a manual statement or otherwise supported by supplied evidence metadata;
- a missing field key, assumption key, typed value, applicable period, confidence, rationale, evidence role, scope, or driver designation;
- a natural-language or unsupported value/unit/currency conversion;
- a scenario adjustment against a masked, missing, non-driver, or incompatible assumption;
- a duplicate adjustment for one assumption or an adjustment on a Base scenario; and
- a destructive request whose exact subject and authorized dependency set have not been shown.

Ask exactly one focused grammatical question about the highest-priority material gap, and end it with a question mark. Do not present a questionnaire, replace the question with an imperative, or ask for a value already supplied.
When all required registration metadata for one evidence source is absent, ask for source identity, capture time, citation, and disclosure together in that single evidence-metadata question.

## Preview and execute one command at a time

Show the ordered plan, but preview only the next explicit mutation as an executable command. Include every user-controlled persisted field, every safe inference, the exact current case and subject heads, and the expected effect. For a removal, show the exact subject and known authorized dependencies and state that no cascade will occur. Wait for an unambiguous confirmation tied to that preview.

After confirmation:

1. Construct a fresh lowercase RFC 4122 version 4 UUID directly in the `commandId` field without using any tool. Its exact grammar is `xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx`, where `y` is `8`, `9`, `a`, or `b` and every `x` is a lowercase hexadecimal digit. Do not send that grammar literally and do not reuse a prior command UUID for a new mutation.
2. Copy every preview input field unchanged into the tool call, add only `commandId`, and call exactly the named domain tool for the confirmed step: `strata_register_evidence`, `strata_save_assumption`, `strata_remove_assumption`, `strata_save_scenario`, or `strata_remove_scenario`. Use framing or alternative mutation tools only when the user explicitly requested that separate change and it has its own preview and confirmation.
3. Treat only a structured success receipt as committed. Retain its exact IDs, replay flag, and successor `caseRevisionId`.
4. Advance the working case head only from that receipt. Materialize receipt IDs needed by the next step, show the next complete preview, and obtain another confirmation. Do not execute the remainder automatically.

The normal evidence-to-driver-to-scenario path uses no more than eight Strata tool calls, including resolution reads and three mutations. Do not omit a safety read or confirmation to meet the cap.

## Stop and recover safely

After the first failed command, make no second mutation call in that turn: do not repeat the failed tool and do not execute a later planned mutation. Keep prior successful receipts, explain the stable error without exposing unauthorized data, refresh only when the recovery contract requires it, and re-preview the uncommitted remainder. A `retry: safe` disposition describes transport safety; it is not permission to retry a definitive application error in the same turn.

- For a lost response, timeout, or ambiguous transport result, retry the byte-identical command at most once with the same `commandId`. Accept `replayed: true` as the original result.
- For `CASE_REVISION_CONFLICT` or `SUBJECT_REVISION_CONFLICT`, do not merge. Read the exact current overview or assumption/scenario projection, compare it with the confirmed proposal, show a fresh complete preview, obtain new confirmation, and use a new command ID.
- For `DEPENDENCY_CONFLICT`, name only returned authorized subject IDs/kinds, propose an explicit removal or revision order, and never cascade implicitly.
- For `ASSUMPTION_NOT_FLEXIBLE`, offer a separately previewed assumption revision that enables the driver; never weaken scenario validation.
- For `VALUE_INCOMPATIBLE` or `INPUT_INVALID`, identify the exact rejected field. Change it only from user-supplied facts, then preview and reconfirm with a new command ID.
- For `RATE_LIMITED` or `RATE_LIMIT_UNAVAILABLE`, stop without an unchanged retry. Preserve the failed preview for a later user-confirmed attempt.
- For `COMMAND_REUSED`, authentication, tenant, membership, access, or unavailable-authorization errors, stop without an unchanged retry.

A cancellation stops before generating a command ID or calling a mutation. Never claim evidence or a scenario is objectively true; report only what the user supplied and Strata durably accepted.
