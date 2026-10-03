# Explain exact retained derivations

Use this reference when the user asks how a calculated value was produced, for
example "how was this calculated". Answer only from the `derivation`
projection of `strata_read_calculation_run`. Never invent inputs, operations,
values, or trace content the tool did not return.

## Select the derivation

- Always resolve the case first: call `strata_list_cases` with `{ "limit": 25 }`,
  match the case title, then call `strata_read_case` for the current overview.
  Read the derivation only after the overview.
- Then read the derivation in one call. Do not read the summary, evaluation,
  comparison, or verification projection first.
- Call `strata_read_calculation_run` with `projection: "derivation"`.
- For a Model Item, supply `runId`, `optionRevisionId`, `scenarioRevisionId`,
  and `modelItemKey`, for example `{ "runId": "<uuid>",
"optionRevisionId": "<uuid>", "scenarioRevisionId": "<uuid>",
"projection": "derivation", "modelItemKey": "margin_share" }`.
- For a Pairwise result, supply `runId`, `optionRevisionId`,
  `scenarioRevisionId`, and `pairwiseResultKey`. The server resolves the
  Counterfactual from the retained Run.
- Take `runId` and the revision IDs from returned results, such as the
  `strata_read_case` case reference or a prior calculation read. Never invent
  them.

## Distinguish the five outcomes

State the returned status word first, for example "status: match".

- `match` with a scalar or series terminal: explain the consumed inputs,
  operation IDs, pre-rounding intermediate values, and the terminal value in
  dependency order. Quote the operation IDs verbatim.
- `match` with a failed terminal: quote the diagnostic code, explanation, and
  source key, and state that the item has no success value.
- `mismatch`: state that recomputation disagrees with the retained outcome and
  that no trace exists.
- `unavailable`: state which exact source is unavailable and that no trace
  exists.
- `execution_failed`: state the failure code and source key and that no trace
  exists.

On `TRACE_LIMIT_EXCEEDED` or `OUTPUT_TOO_LARGE`, state the bound outcome and
stop. No partial trace exists. Never quote, summarize, or reconstruct trace
content after a limit.

## Copy exact values

- Copy decimal and rational text verbatim, including numerator and denominator.
  State each rational as "<numerator> divided by <denominator>", for example
  "900000 divided by 7". Never simplify a value in a way that changes its
  meaning.
- Copy source identities verbatim: `modelItemKey`, `assumptionKey`,
  `pairwiseResultKey`, and revision IDs.
- Follow node order as returned. A conditional consumes only its taken branch:
  never mention values from unconsumed nodes. A failed-source terminal
  dependency appears once.

## Stay read-only

Never recommend an option, repair a model, verify a Run, or write anything.
