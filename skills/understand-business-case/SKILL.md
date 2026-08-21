---
name: understand-business-case
description: Read and explain an existing Strata business case from exact, authorized revisions. Use when the user asks about a case's current framing, alternatives, assumptions, drivers, scenarios, evidence lineage, revision history, or changes from the previous version.
---

# Understand Business Case

Answer the user's question with the smallest authorized Strata read sequence that can establish it. Treat every revision as immutable and every omitted `caseRevisionId` as a one-time request to resolve current state.

Use Strata MCP as the only source of case facts. Do not inspect the working directory, repository, process environment, or public web, and do not use a shell to search for case data. Do not call `list_mcp_resources` or `list_mcp_resource_templates`; the exact permitted Strata tools are named below. After the host loads this installed skill, continue with those Strata tools and the user's messages only.

## Organization-order invariant

Before the first tenant-bound case call, determine whether the host or conversation has already established the active Strata organization. If it has not, the first Strata tool call must be `strata_connection_context`. Never call `strata_list_cases`, `strata_read_case`, `strata_get_case`, a history tool, or a comparison tool and then resolve connection context. Call connection context at most once in a normal journey.

## Bounded case-list invariant

Every `strata_list_cases` call must use exactly `{ "limit": 25 }`. Never omit `limit`, use another value, or send an empty input object. This rule applies to every case lookup, including scenario-to-evidence traces.

## Exact lineage-summary invariant

For every scenario-to-evidence answer, copy the adjustment operation and signed value into the final `summary` exactly as returned. A paraphrase such as “15 percent decrease” does not replace a returned signed value such as `-15`. Also include the returned scenario label, assumption label, evidence role, evidence title or citation, and source URI.

## Read workflow

1. Identify the requested case and the exact question before calling a tool. Apply the organization-order invariant above. Also use `strata_connection_context` when the user asks which company or capabilities are active, or when context is necessary to explain an access problem.
2. Resolve the case:
   - If the user supplied `investmentCaseId` and `decisionCycleId`, retain them.
   - Otherwise call `strata_list_cases` with `{ "limit": 25 }`. Match only returned titles or case keys.
   - If multiple returned cases could match, show bounded choices with title and case key and ask the user to select one. Never guess.
   - If the required match is not on the page and `nextCursor` exists, follow that cursor only when another page can answer the user's request.
3. Select one projection:
   - `overview` for framing, alternatives, or subject counts.
   - `assumptions` for assumptions or sensitivity drivers; set `driversOnly: true` only when the user asks about drivers.
   - `scenario` for one known `scenarioId` and its ordered adjustments.
   - `evidence_lineage` for one known `assumptionId` or `evidenceItemId`.
   - Use `strata_get_case` only when one question must cross scenarios, assumptions, and evidence or when their selection IDs are not known. Its exact full graph is broader than a focused projection, so do not use it for a single-subject question.
   - For a scenario-to-evidence request that names a scenario by label and supplies no subject IDs, resolve the case with `strata_list_cases` and then call `strata_get_case` directly. Do not insert an overview or scenario projection before the full-graph read.
4. Call `strata_read_case` for a focused projection. Omit `caseRevisionId` only for the first current-state read. Record `caseReference.caseRevisionId`, `currentCaseRevisionId`, `parentCaseRevisionId`, both revision numbers, and `resolvedFromCurrent`. When `strata_get_case` is necessary, record its returned `caseRevisionId` and pin any follow-up call to it.
5. Pin every follow-up read to the recorded `caseRevisionId`. Preserve the same revision and returned cursor when paging. Never merge results from different revisions into an unlabeled answer.
6. Answer only from returned fields. State the case key and exact revision number or ID. Treat “where are we?” as a broad current-state overview. In the final structured `summary`, include the returned objective and every returned alternative label exactly as returned; do not replace them with a count or generic prose. Omit one of those fields only when the user explicitly asks for a narrower answer. Label a non-current read as historic or superseded. Distinguish recorded facts, user-supplied interpretation, and missing or masked data.
7. Stop when the selected projection answers the question. For “where are we?” or another current-state question, do not list revision history or read another projection unless the user also asks about a prior time, changes, authorship, or history.

## History and comparison

- Use `strata_list_case_revisions` when the user asks for revision history or needs a revision selected by recorded operation, author, or time. Preserve its cursor for further history pages.
- Interpret “previous” as the selected revision's exact `parentCaseRevisionId`, never the second item in a date-sorted list.
- Use `strata_compare_case_revisions` only with two exact revision IDs and `limit: 25`. Set `fromCaseRevisionId` to the older/base revision and `toCaseRevisionId` to the newer/target revision.
- For “what changed since the previous revision,” use the selected current overview's `parentCaseRevisionId` immediately as `fromCaseRevisionId` and its `caseRevisionId` as `toCaseRevisionId`. The complete normal sequence is bounded case list, current overview, then exact comparison. Do not call `strata_list_case_revisions`, because history order is not the parent relation. Do not read the parent overview unless the user explicitly asks for a previous value that the comparison does not return.
- Report every returned comparison item. For a narrative item, name every entry in `changedFields`, even when several fields changed in one item. Pair each changed field with its current value from the already returned overview when that field is present there; for example, report the current objective text and current deadline, not only “objective and deadline changed.” For subject items, preserve `added`, `changed`, and `removed` exactly. Fetch another exact projection only if the user's question requires a subject's actual value that the comparison and current overview do not return.
- Do not describe a change as causal. Revision lineage establishes author, recorded time, and change reason, not business causation or objective truth.

## Evidence and restricted data

- Trace a visible assumption through its exact `assumptionId` and `assumptionRevisionId`; trace a visible scenario adjustment through its exact assumption IDs; trace evidence through exact `evidenceItemId` and any `supersedesEvidenceItemId`.
- When the user asks for scenario-to-evidence lineage, render every link in order: scenario label and exact `scenarioId`; adjustment operator/value; assumption label plus exact `assumptionId` and `assumptionRevisionId`; then evidence role, evidence title or citation, source URI when returned, and exact `evidenceItemId`. Do not collapse the assumption or evidence-role link even when the source title seems self-explanatory.
- Preserve evidence roles exactly: `supporting`, `contradicting`, `contextual`, or `superseding`. A citation, excerpt, locator, or source URI is recorded metadata, not proof that the claim is true.
- If an item has `access: masked`, say that Strata returned a restricted item and report only its permitted mask fields or ordinal/role. Do not infer its value, owner, rationale, IDs, or change reason from surrounding records.
- Never invent a missing fact, calculate a recommendation, rank alternatives, or claim that prose is objectively good. You may identify explicit omissions, contradictions, or implausible values and ask the user how to interpret them.

## Recovery

- On an ambiguous title, stop after bounded choices and wait for selection.
- On `INPUT_INVALID` or `MALFORMED_CURSOR`, correct the input from the tool schema or restart the page from the same exact revision; do not substitute another revision.
- On `NOT_FOUND`, explain which exact case or revision was unavailable and ask for a valid selection.
- On `AUTHORIZATION_UNAVAILABLE` or masked data, do not search for or infer the restricted content.
- On `PERSISTENCE_UNAVAILABLE`, `CAPABILITY_TIMEOUT`, or another retryable read failure, retry the unchanged read at most once when useful; otherwise report that Strata could not complete the read.
- Keep a normal successful journey within four Strata tool calls. Pagination may exceed that only when the user explicitly requests more returned records.

When the host supports local skill references, use [the exact read, pagination, lineage, comparison, restricted-data, and recovery contract](references/exact-reads-history-and-lineage.md) for additional input examples and result-field details.
