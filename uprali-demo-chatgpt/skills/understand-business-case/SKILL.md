---
name: understand-business-case
description: Read and explain exact authorized Uprali business-case state. Use when a user asks about current framing, alternatives, assumptions, drivers, scenarios, evidence lineage, revision history, previous-version comparisons, Assumption Reviews, Decisions, Outcome Reviews, Decision Learnings, or exact Learning references.
---

# Understand Business Case

Answer with the smallest authorized Uprali read sequence that can establish it. Treat every revision as immutable and every omitted `caseRevisionId` as a one-time request to resolve current state. For Assumption Review, Decision, Outcome Review, and Decision Learning questions, use the exact protected read guidance in [the exact read, pagination, lineage, comparison, restricted-data, and recovery contract](references/exact-reads-history-and-lineage.md).

Use Uprali MCP as the only source of case facts. Do not inspect the working directory, repository, environment, or public web, and do not use a shell to search for case data. Do not call `list_mcp_resources` or `list_mcp_resource_templates`; the exact permitted Uprali tools are named below. After the host loads this installed skill, continue with those Uprali tools and the user's messages only.

When the Uprali tools are unavailable, the Uprali demo is not connected in this client. Stop before any other step and ask the user to connect it to `https://strata-utrtwerwt.sprava.ai/api/uprali-demo/mcp`: in Claude web or the Claude desktop app, open Customize, Connectors. The demo needs no account.

## Organization-order invariant

Before the first tenant-bound case call, check whether the host or conversation already established the active Uprali organization. If it has not, the first Uprali tool call must be `strata_connection_context`. Never call `strata_list_cases`, `strata_read_case`, `strata_get_case`, a history tool, or a comparison tool and then resolve connection context. Call connection context at most once in a normal journey.

## Bounded case-list invariant

Every `strata_list_cases` call must use exactly `{ "limit": 25 }`. Never omit `limit`, change it, or send an empty input object. This rule applies to every case lookup, including scenario-to-evidence traces.

## Scenario compared with Base

When `strata_compare_scenario_to_base` is available and the user asks to compare a named scenario with Base, skip Read workflow steps 3 to 8 and use [the scenario-with-Base rule](references/history-and-comparison.md#scenario-compared-with-base).

## Read workflow

1. Identify the exact question before calling a tool. Apply the organization-order invariant above. If the request is not a direct protected-record read, identify the requested case first. Also use `strata_connection_context` when the user asks which company or capabilities are active, or when context is necessary to explain an access problem.
2. Before resolving a case, check whether the user supplied a visible Assumption Review, Decision, Outcome Review, Outcome Review Revision, or Decision Learning ID. For that request, call the matching protected read directly. Do not resolve a case first, and do not infer a Case from the record ID.
3. Resolve the case for case projections, history, comparison, and scenario-to-evidence answers:
   - When the App's model context names a built case with its case key and `investmentCaseId`, that is the build result. For `strata_list_assumption_cards`, `strata_read_calculation_results`, and `strata_read_approval_path`, use that `investmentCaseId` directly and do not call `strata_list_cases`. Tools that need `decisionCycleId`, such as `strata_read_case`, keep the existing resolution below.
   - If the user supplied `investmentCaseId` and `decisionCycleId`, retain them.
   - Otherwise call `strata_list_cases` with `{ "limit": 25 }`. Match only returned titles or case keys.
   - If the user refers to the new case and this conversation has no build result, take the returned case with the newest `createdAt`; when this conversation showed an intake, the case must also have the intake title.
   - If multiple returned cases could match, show bounded choices with title and case key and ask the user to select one. Never guess.
   - If the required match is not on the page and `nextCursor` exists, follow that cursor only when another page can answer the user's request.
4. Select one projection with [the projection selection rules](references/select-projection.md). For "how was this calculated" questions, use [the derivation rules](references/explain-derivations.md).
5. Call `strata_read_case` for a focused case projection, or the step-4 protected read, with [the read rules](references/exact-reads-history-and-lineage.md#read-and-pin-exact-revisions).
6. Pin every follow-up read with [the pinning rules](references/exact-reads-history-and-lineage.md#read-and-pin-exact-revisions).
7. Answer only from returned fields. For case-projection answers, state the case key and exact revision number or ID. Treat “where are we?” as a broad current-state overview. In the final structured `summary`, include the returned objective and every returned alternative label exactly as returned; do not replace them with a count or generic prose. Omit a field only when the user explicitly asks for a narrower answer. For direct protected-read answers, report only returned identifiers and returned fields. Do not resolve a case only to satisfy a case key or revision statement for a direct protected read. Label a non-current read as historic or superseded. Distinguish recorded facts, user-supplied interpretation, and missing or masked data. Before you state Assumption Card or approval status, call `strata_list_assumption_cards` or `strata_read_approval_path` for the case in the same turn.
8. Stop when the selected projection answers the question. For “where are we?” or another current-state question, do not list revision history or read another projection unless the user also asks about a prior time, changes, authorship, or history.

## History and comparison

Read [the history and comparison rules](references/history-and-comparison.md) for revision history or comparison questions.

## Evidence and restricted data

Read [the evidence and restricted-data rules](references/evidence-and-restricted-data.md) for evidence, lineage, masked items, or the final `summary` of a scenario-to-evidence answer.

## Recovery

Read [the recovery rules](references/recovery.md) when a Uprali call fails or a title is ambiguous.

When the host supports local skill references, use [the exact read, pagination, lineage, comparison, restricted-data, and recovery contract](references/exact-reads-history-and-lineage.md) for additional input examples and result-field details.
