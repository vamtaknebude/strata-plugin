---
name: understand-business-case
description: Read and explain exact authorized Strata business-case state. Use when a user asks about current framing, alternatives, assumptions, drivers, scenarios, evidence lineage, revision history, previous-version comparisons, Assumption Reviews, Decisions, Outcome Reviews, Decision Learnings, or exact Learning references.
---

# Understand Business Case

Answer the user's question with the smallest authorized Strata read sequence that can establish it. Treat every revision as immutable and every omitted `caseRevisionId` as a one-time request to resolve current state. For Assumption Review, Decision, Outcome Review, and Decision Learning questions, use the exact protected read guidance in [the exact read, pagination, lineage, comparison, restricted-data, and recovery contract](references/exact-reads-history-and-lineage.md).

Use Strata MCP as the only source of case facts. Do not inspect the working directory, repository, process environment, or public web, and do not use a shell to search for case data. Do not call `list_mcp_resources` or `list_mcp_resource_templates`; the exact permitted Strata tools are named below. After the host loads this installed skill, continue with those Strata tools and the user's messages only.

When the Strata tools are unavailable, Strata is not connected in this client. Stop before any other step and ask the user to connect it to `https://strata-utrtwerwt.sprava.ai/api/mcp`: in the Claude desktop app, open Customize, Connectors; in the Claude Code CLI, run `/mcp`; in another client, its own MCP sign-in.

## Organization-order invariant

Before the first tenant-bound case call, determine whether the host or conversation has already established the active Strata organization. If it has not, the first Strata tool call must be `strata_connection_context`. Never call `strata_list_cases`, `strata_read_case`, `strata_get_case`, a history tool, or a comparison tool and then resolve connection context. Call connection context at most once in a normal journey.

## Bounded case-list invariant

Every `strata_list_cases` call must use exactly `{ "limit": 25 }`. Never omit `limit`, use another value, or send an empty input object. This rule applies to every case lookup, including scenario-to-evidence traces.

## Exact lineage-summary invariant

For every scenario-to-evidence answer, copy the adjustment operation and signed value into the final `summary` exactly as returned. A paraphrase such as “15 percent decrease” does not replace a returned signed value such as `-15`. Also include the returned scenario label, assumption label, evidence role, evidence title or citation, and source URI.

## Read workflow

1. Identify the exact question before calling a tool. Apply the organization-order invariant above. If the request is not a direct protected-record read, identify the requested case before calling a case tool. Also use `strata_connection_context` when the user asks which company or capabilities are active, or when context is necessary to explain an access problem.
2. Before resolving a case, check whether the user supplied a visible Assumption Review, Decision, Outcome Review, Outcome Review Revision, or Decision Learning ID. For that request, call the matching protected read directly. Do not resolve a case first, and do not infer a Case from the record ID.
3. Resolve the case for case projections, history, comparison, and scenario-to-evidence answers:
   - If the user supplied `investmentCaseId` and `decisionCycleId`, retain them.
   - Otherwise call `strata_list_cases` with `{ "limit": 25 }`. Match only returned titles or case keys.
   - If multiple returned cases could match, show bounded choices with title and case key and ask the user to select one. Never guess.
   - If the required match is not on the page and `nextCursor` exists, follow that cursor only when another page can answer the user's request.
4. Select one projection with [the projection selection rules](references/select-projection.md).
5. Call `strata_read_case` for a focused case projection. For an Assumption Review, Decision, Outcome Review, or Decision Learning read, call the named protected read tool from this list and apply the exact read guidance in the reference. Do not add `caseRevisionId` to those protected-read inputs. Record exact case revision fields only when a protected read returns them. When `strata_get_case` is necessary, record its returned `caseRevisionId` and pin any follow-up case read to it.
6. Pin every follow-up `strata_read_case` call to the recorded `caseRevisionId`. For history, preserve `investmentCaseId`, `decisionCycleId`, and any returned cursor. For comparison, use exact `fromCaseRevisionId` and `toCaseRevisionId` values selected from returned fields. For protected-read pagination, preserve the same record reference and top-level limit. When the read includes Disposition history, also preserve `dispositionHistory.findingId` and its nested limit, and map each returned `nextCursor` to the corresponding input `cursor`. Never send `caseRevisionId` to `strata_read_decision`, `strata_read_outcome_review`, `strata_read_learning`, or `strata_read_assumption_review`. Never merge results from different revisions into an unlabeled answer.
7. Answer only from returned fields. For case-projection answers, state the case key and exact revision number or ID. Treat “where are we?” as a broad current-state overview. In the final structured `summary`, include the returned objective and every returned alternative label exactly as returned; do not replace them with a count or generic prose. Omit one of those fields only when the user explicitly asks for a narrower answer. For direct protected-read answers, report only returned identifiers and returned fields. Do not resolve a case only to satisfy a case key or revision statement for a direct protected read. Label a non-current read as historic or superseded. Distinguish recorded facts, user-supplied interpretation, and missing or masked data.
8. Stop when the selected projection answers the question. For “where are we?” or another current-state question, do not list revision history or read another projection unless the user also asks about a prior time, changes, authorship, or history.

## History and comparison

Read [the history and comparison rules](references/history-and-comparison.md) for revision history or comparison questions.

## Evidence and restricted data

Read [the evidence and restricted-data rules](references/evidence-and-restricted-data.md) for evidence, lineage, or masked items.

## Recovery

Read [the recovery rules](references/recovery.md) when a Strata call fails or a title is ambiguous.

When the host supports local skill references, use [the exact read, pagination, lineage, comparison, restricted-data, and recovery contract](references/exact-reads-history-and-lineage.md) for additional input examples and result-field details.
