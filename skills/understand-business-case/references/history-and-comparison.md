# History and comparison

- Use `strata_list_case_revisions` when the user asks for revision history or needs a revision selected by recorded operation, author, or time. Preserve its cursor for further history pages.
- Interpret “previous” as the selected revision's exact `parentCaseRevisionId`, never the second item in a date-sorted list.
- Use `strata_compare_case_revisions` only with two exact revision IDs and `limit: 25`. Set `fromCaseRevisionId` to the older/base revision and `toCaseRevisionId` to the newer/target revision.
- For “what changed since the previous revision,” use the selected current overview's `parentCaseRevisionId` immediately as `fromCaseRevisionId` and its `caseRevisionId` as `toCaseRevisionId`. The complete normal sequence is bounded case list, current overview, then exact comparison. Do not call `strata_list_case_revisions`, because history order is not the parent relation. Do not read the parent overview unless the user explicitly asks for a previous value that the comparison does not return.
- Report every returned comparison item. For a narrative item, name every entry in `changedFields`, even when several fields changed in one item. Pair each changed field with its current value from the already returned overview when that field is present there; for example, report the current objective text and current deadline, not only “objective and deadline changed.” For subject items, preserve `added`, `changed`, and `removed` exactly. Fetch another exact projection only if the user's question requires a subject's actual value that the comparison and current overview do not return.
- Do not describe a change as causal. Revision lineage establishes author, recorded time, and change reason, not business causation or objective truth.

## Scenario compared with Base

When `strata_compare_scenario_to_base` is available and the user asks to compare a named scenario with Base, skip Read workflow steps 3 to 8. Take `runId` and `option.optionRevisionId` from this case's latest `strata_read_calculation_results` result; if there is none, resolve the case and call that tool with its `investmentCaseId`. Then call `strata_compare_scenario_to_base` once with `scenarioLabel` set to the user's scenario name and `limit: 25`, and answer with its Pairwise Results, such as NPV and payback.
