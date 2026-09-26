# Build the normalized draft

1. Use stated facts and unambiguous safe inferences only. You may infer a concise title, option type, and an empty list when the user asserted no items. Disclose each inference. Do not invent business facts.
2. Keep the decision question, objective, materiality, and counterfactual distinct. The user must state the expected outcome under no proposed change. Do not infer it from current operations, the materiality statement, or an alternative.
3. Preserve supplied uncertainty and attribution. Do not convert an estimate, vendor claim, or objective into an established fact, constraint, or dependency.
4. Treat the case deadline as the deadline for making the decision, not the date when an alternative must become operational. Ask for an exact `YYYY-MM-DD` decision date when a supplied relative phrase makes timing material. Ask about implementation timing only to complete or omit a partial implementation period.
5. Put cross-cutting bounds and exclusions in `constraints`. Put delivery boundaries in `scope` and reasons for consideration in `rationale`.
6. Require 2–25 alternatives that differ materially. Renamed copies, restated objectives, and equivalent mechanisms are not distinct. Do not invent defer or stop alternatives. Include them when the user explicitly proposes them.
7. Map `optionType` from the alternative's mechanism: `build` creates or owns capability; `buy` procures it externally; `pilot` runs a limited experiment; `defer` continues the current approach or postpones a new commitment; `stop` ends the activity; use `custom` only when none applies. Disclose the inferred type.
8. Obtain `label`, `optionType`, `scope`, `rationale`, `dependencies`, and `strategicEffects` for each alternative. Include implementation timing only as an exact forward-ordered pair. Otherwise omit both dates.
9. Obtain a specific nonblank `changeReason` for creating the immutable root.
