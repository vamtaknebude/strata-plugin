# Performance

Use this reference only when `strata_save_case_performance` and `strata_read_performance_registry` are available. When either tool is missing, record no performance data. If the user asks about case performance or the performance registry, say that this Strata connection cannot record it, and continue with the rest of the request.

## Concepts

- A performance designation names which Outcome Commitments of a decided case measure revenue, ROI, and payback. Each of the three lists holds Commitment positions in quarter order, one per quarter.
- A Commitment position is the zero-based index of the Commitment in the `commitments` array that `strata_read_decision` returns for the case's latest Decision.
- Health is `on_track`, `at_risk`, or `off_track`, with an assessment date and a note. The user chooses the status, the assessment date, and the note. Strata does not derive the health from the planned and actual values.
- The registry shows, for one calendar quarter such as `2027-Q3`, each designated case's planned values, the actual values that completed Outcome Reviews observed, and its latest health. An actual value is `null` until a completed Outcome Review observes it.

## Read before a save

Call `strata_read_performance_registry` with any quarter before each save and find the case. Pass its `performanceRevisionNumber` as `expectedPerformanceRevisionNumber`, or `null` when the case is not listed and has no designation yet. Any quarter lists every designated case. Read every page, passing `nextCursor` as `cursor`, before you treat the case as not listed.

For a list the user changes, take positions from `strata_read_decision` for the case's latest Decision. For a list the user keeps unchanged, including every list in a health-only save, take the positions that the registry's `quarters` entries show for it, as the Save section says. When the case has no Decision, tell the user it cannot be designated until a Decision is recorded. Match each Commitment's indicator to the metric the user names. The registry places a Commitment in the calendar quarter where its evaluation period starts. Designate at most one Commitment per quarter in each list.

## Save

A save replaces the whole designation and records a new health. Send all three lists, each with at least one position. To keep a list unchanged, send the positions that the registry's `quarters` entries show for it. Show the revenue, ROI, and payback Commitments by quarter with their indicators, and the health status, assessment date, and note, and obtain confirmation first. Then send one `strata_save_case_performance`.

## Retry

Use a fresh `idempotencyKey` for each confirmed save. Retry an ambiguous result only with the same `idempotencyKey` and unchanged input. On `STALE_STATE`, read the registry again, show the changed preview, reconfirm, and use a new key.
