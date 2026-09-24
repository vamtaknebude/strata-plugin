# Coherent initial case contract

Use this reference to construct the exact `strata_start_case` input and to diagnose weak conversational input. The tool declaration remains authoritative for mechanical bounds. This reference defines the semantic distinctions that the declaration cannot teach by itself.

## Contents

- [Narrative field mapping](#narrative-field-mapping)
- [Alternative contract](#alternative-contract)
- [Weak-input diagnostics](#weak-input-diagnostics)
- [Exact preview template](#exact-preview-template)
- [Exact tool input](#exact-tool-input)
- [Retry identity](#retry-identity)

## Narrative field mapping

`title` is a short noun phrase identifying the case. It is not the decision question.

`decisionQuestion` states the actual choice and its subject. A useful form is “Should we build, buy, or defer the forecasting platform before the 2027 planning cycle?” Reject a topic such as “Forecasting” and an objective such as “Improve forecasting” as incomplete questions.

`objective` states the desired outcome and any user-supplied success criteria. Preserve concrete measures, thresholds, or service levels exactly when supplied. Do not invent a metric or collapse several conflicting objectives into one; ask which governs.

`materialityStatement` records the problem or opportunity and why the choice matters. It must not merely say that the decision is important. Preserve the affected cost, risk, service, customer, capability, or timing fact without asserting unsupported causation.

`counterfactual` records what happens if no proposed change is made. The user must state the expected no-change outcome before preview. A description of current operations does not establish that future outcome. Do not derive it from the materiality statement or an alternative. It is not automatically the same as a “defer” alternative: the counterfactual describes the baseline, while a defer alternative is an explicit selectable course with its own scope and rationale.

`deadline` is an optional exact calendar date in `YYYY-MM-DD`. Record it only when the user gives or confirms a date. A renewal, fiscal period, quarter, or relative phrase that does not resolve to one date is a focused clarification.

`constraints` is the complete ordered list of cross-cutting hard bounds and material in/out-of-scope facts. It may include a supplied budget ceiling, regulatory requirement, service-level floor, geographic boundary, required stakeholder impact, or immovable date. An aspiration belongs in the objective, not the constraint list.

The authenticated user is the initial server-derived owner and author. A sponsor or stakeholder named in the brief is not substituted for that user. When a stakeholder fact changes the decision, preserve it in the relevant objective, materiality, constraint, alternative scope, or strategic effect.

## Alternative contract

Create 2–25 alternatives in the user's intended order. Each alternative contains:

- `label`: a concise name unique within the preview;
- `optionType`: exactly `build`, `buy`, `custom`, `defer`, `pilot`, or `stop`;
- `scope`: what that course includes, excludes, delivers, or retains;
- `rationale`: why this course merits comparison, using a fact or tradeoff rather than a restated objective;
- `dependencies`: the complete user-supplied prerequisites, or `[]` only when none were asserted;
- `strategicEffects`: the complete user-supplied durable effects, or `[]` only when none were asserted; and
- `implementationStart` plus `implementationEnd` only as an exact forward-ordered date pair.

Alternatives are materially distinct when they differ in mechanism, supplier/ownership model, commitment, timing, scope, or another consequential choice dimension. These are not distinct:

- “Build platform” and “Build platform faster” with the same scope and mechanism;
- “Improve accuracy” and “Increase forecast quality,” which are objectives rather than courses of action;
- labels that differ while scope and rationale are copies; or
- a base alternative split into phases when the phases are not independently selectable.

Do not manufacture a “do nothing” option. Add a defer or stop alternative only when the user wants it considered. Always record the counterfactual separately.

## Weak-input diagnostics

Ask one focused question when any of these conditions holds:

- the question names a topic but no choice;
- the objective is absent, circular, or conflicts with a stated success criterion;
- materiality states no business consequence;
- the user has not stated the expected no-action outcome, even when current operations are described;
- a time-sensitive phrase lacks an exact date;
- fewer than two courses of action remain after removing duplicates;
- an alternative lacks a scope or rationale;
- one implementation date is present without the other, or start is not before end;
- a dependency, effect, stakeholder impact, scope boundary, or uncertainty is referenced but materially ambiguous; or
- the change reason is blank or merely repeats the title.

Placeholders such as “to be determined,” “usual constraints,” “standard option,” “as appropriate,” and “etc.” are not facts. Ask what the placeholder must mean when it affects persisted content. Do not interrogate the user about optional detail that has no material effect on the root case.

## Exact preview template

Render the complete values, not only this outline:

```text
Initial owner/author: authenticated Uprali user
Change reason: <exact text>

Framing
- title: <exact text>
- decisionQuestion: <exact text>
- objective: <exact text>
- materialityStatement: <exact text>
- counterfactual: <exact text>
- deadline: <YYYY-MM-DD or not recorded>
- constraints, in order: <complete list or []>

Alternatives, in order
1. <label>
   - optionType: <enum>
   - scope: <exact text>
   - rationale: <exact text>
   - dependencies: <complete list or []>
   - strategicEffects: <complete list or []>
   - implementationStart / implementationEnd: <both dates or not recorded>

Atomic effect: confirmation creates one immutable root revision containing this
framing and all alternatives, plus the server-derived case/cycle/owner, audit,
outbox, and command receipt records.
Idempotency key: generated after confirmation; reused only for an unchanged retry.
```

If a correction changes one field, do not show only that field. Re-render this entire preview so the next confirmation is tied to one exact normalized command.

## Exact tool input

After confirmation, call `strata_start_case` with this shape:

```json
{
  "idempotencyKey": "<new UUID>",
  "changeReason": "<confirmed reason>",
  "narrative": {
    "title": "<confirmed title>",
    "decisionQuestion": "<confirmed question>",
    "objective": "<confirmed objective>",
    "materialityStatement": "<confirmed materiality>",
    "counterfactual": "<confirmed baseline>",
    "deadline": "<confirmed YYYY-MM-DD, omitted when not recorded>",
    "constraints": []
  },
  "alternatives": [
    {
      "label": "<confirmed label>",
      "optionType": "build",
      "scope": "<confirmed scope>",
      "rationale": "<confirmed rationale>",
      "dependencies": [],
      "strategicEffects": []
    }
  ]
}
```

The example shows shape, not a complete valid command: a real command needs at least two complete alternatives. Array position records alternative order. `ordinal` appears in the server receipt only; never send it on an alternative. Omit both implementation keys when timing is not recorded. Never send `null`, a natural-language date, only one date, or a server-derived identity/tenant field.

## Retry identity

The retry identity is the canonical confirmed tool input plus `idempotencyKey`.

- An unchanged retry after timeout or a lost response uses the same UUID, fields, array order, array items, and omitted properties.
- A correction, reordered alternative, changed inference, or validation repair changes intent and requires another complete preview, confirmation, and UUID.
- `replayed: true` means the original transaction already succeeded. Report its receipt and stop retrying.
- `COMMAND_REUSED` means the same UUID reached the server with different input. Do not guess which version won.

For a stale later edit, `currentCaseHead` or `currentSubjectHead` is remediation data, not permission to overwrite. Read the returned/current exact revision with `strata_read_case`, compare it with the user's intended change, show a new preview, reconfirm, and use a new UUID.
