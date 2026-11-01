# Risks, findings, and business-case output

Use these rules at the matching branch of [`read-workflow.md`](read-workflow.md). Keep every read on the exact record or Case Revision the user requested.

## Read an FP&A review by ID

When the user supplies a `reviewId`, call `strata_read_fpa_review` with only `{ reviewId }` before resolving a case. Report the returned review and Run IDs, exact Case Revision, and lifecycle state. For `running`, say the review is still running. For `ready`, present every returned finding's concern, importance, and citations; an empty findings array means the completed review has no findings. For `failed`, report the returned failure reason. Do not treat a read error or failed review as an empty result.

## Read the exact-revision output

Call `strata_read_business_case_output` with exactly `investmentCaseId`, `decisionCycleId`, and `caseRevisionId`. Use the supplied tuple as-is; otherwise resolve a case through the bounded workflow and pin the call to its returned revision. Present the returned revision and every returned section in the returned order. Preserve section keys, statuses, source names, identifiers, units, labels, values, findings lifecycle and Run IDs, and provenance. Do not fetch a replacement revision or another source to fill an omitted section.

Distinguish the returned states:

- `available` carries the owner's returned data. An FP&A section can be available with `reviewStatus` `running`, `ready`, or `failed`; report that lifecycle state. A ready review with no findings is still a completed review.
- `empty` means the owner returned no values for that section and its stated reason.
- `not_requested` means the optional section was not requested.
- `not_applicable` means the section does not apply to this output; preserve the template-specific result.
- `unavailable` means its owner has not supplied the section. Name the returned source and continue presenting the available sections immediately.

Report an actual tool error as an error, not as an unavailable or empty section. Do not wait for results, approval, card, or success-metric owners, and do not invent values for an unavailable section.

## Present risks and trade-offs

Keep `what_must_be_true` separate from risks. For each returned risk, preserve its stable `riskItemId`, risk and reason text, evidence, current `provenance`, and `originProvenance`. When the current provenance is `suggested`, call it **Uprali Suggested Risk**. When the current provenance is `user_provided` and its origin is `suggested`, use the current label and identify its suggested origin.

Present returned Buy/Build trade-offs neutrally. Do not choose a winner or add a recommendation. Preserve any template-specific missing-basis result as returned.

Reading output is read-only. It does not request FP&A, run a calculation, change a risk, approve a case, or save success metrics.
