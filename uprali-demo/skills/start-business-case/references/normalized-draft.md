# Build the normalized draft

1. The draft has three values: the brief `title`, the brief `description`, and the `changeReason`. Intake accepts nothing else for a new case.
2. Use the user's own words. You may infer a concise `title` from the description when the user gave none; disclose the inference. Do not invent business facts.
3. `title` is a short noun phrase naming the case (at most 200 characters). `description` states the decision the case supports and any objective or success criterion the user supplied (at most 2000 characters). Preserve supplied measures, thresholds, uncertainty, and attribution exactly.
4. Obtain a specific nonblank `changeReason` (at most 1000 characters) for creating the first revision.
5. Do not collect a decision question, counterfactual, materiality statement, deadline, constraints, alternatives, or implementation dates. Creation does not send them. When the user volunteers such facts, keep them in the `description` only when they fit its length bound, and tell the user that creation records the description as the objective and nothing else.
