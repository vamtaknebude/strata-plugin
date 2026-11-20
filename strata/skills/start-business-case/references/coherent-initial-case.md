# Intake input contract

Use this reference to construct the exact `strata_record_case_intake` routing and creation inputs and to preview them. The tool declaration remains authoritative for mechanical bounds.

## Contents

- [Brief and change reason](#brief-and-change-reason)
- [Weak-input diagnostics](#weak-input-diagnostics)
- [Intake selection and lifecycle](#intake-selection-and-lifecycle)
- [What creation persists](#what-creation-persists)
- [Exact preview template](#exact-preview-template)
- [Exact tool input](#exact-tool-input)
- [Retry identity](#retry-identity)

## Brief and change reason

`brief.title` is a short noun phrase identifying the case (1–200 characters). Creation stores it as the revision `title`.

`brief.description` is the decision brief in the user's words (1–2000 characters). Routing reads it to propose a template. Creation stores it as the revision `objective`. Preserve concrete measures, thresholds, and service levels exactly when supplied. Do not invent a metric.

`changeReason` is the specific reason for creating the case (1–1000 characters). It is not blank and does not merely repeat the title.

The authenticated user is the server-derived owner and author. A sponsor or stakeholder named in the brief is not substituted for that user.

## Weak-input diagnostics

Ask one focused question when any of these conditions holds:

- the description names a topic but no decision;
- the title is absent and the description supports no concise title;
- the description exceeds 2000 characters or the title exceeds 200; or
- the change reason is blank or merely repeats the title.

Placeholders such as "to be determined" and "etc." are not facts. Do not ask about optional detail: the intake calls accept no framing detail, alternatives, deadline, or constraints.

## Intake selection and lifecycle

After a proposal, call `strata_read_case_intake` with the chosen `templateKey`, the original `brief`, and the complete `routingResult`. Match `answers.options` to the published `options` rows and show the actual selected labels. Read the preview again if the chosen template changes.

General investment and Multi-product select every published option automatically. Explain that no repeated option selection is needed. Support-AI still requires an explicit selection answer; show that need as unanswered. Show remaining needs without collecting additional answers during this creation workflow.

After creation, explain that the confirmed intake selections are separate from saved alternatives. Creation has no saved alternatives; accepting a reviewed draft creates them. An overview showing zero alternatives before acceptance does not establish lost options or a broken case. Do not invent repair steps or ask for an already satisfied selection. Offer to complete remaining intake needs and then request and review a draft, without starting that next task.

## What creation persists

The creation call writes, in one transaction (`executeIntakeStartTransaction` and `insertShellCaseRevision`, `src/server/case-graph/case-repository/start-case-from-intake.ts`):

- the case identity: a case key, a decision cycle, and the authenticated user as owner;
- the first revision, with `title` from `brief.title`, `objective` from `brief.description`, no alternatives, and the configuration pinned for the confirmed template;
- missing decision question, materiality statement, and counterfactual, represented as `null`, and no constraints. Intake derives none of these framing statements from the brief and supplies no generic assertions. Later explicit narrative edits can supply them; draft generation and acceptance preserve missing framing;
- one intake-answers row. Creation sends `answers: {}`; General investment and Multi-product automatically record all supported published options, while Support-AI still requires an explicit selection answer. The row also holds any company-profile figures that `applyProfilePrefill` maps into key inputs (`src/server/case-drafting/intake.ts:336`, reported as `prefilled`);
- the template-routing choice linking the case to the routing judgment;
- audit events `case.lifecycle.created` and `case.intake.record`; and
- the command receipt `case.intake.recorded`, whose `needs` list the template's unanswered objectives, options, success metrics, and key inputs.

The routing call writes one template routing judgment and no case.

## Exact preview template

Render the complete values, not only this outline:

```text
Owner/author: authenticated Uprali user
Change reason: <exact text>

Brief
- title: <exact text>
- description (stored as the objective): <exact text>

Proposed template: <name> (<templateKey>), confidence <value or not reported>

Selected intake options: <labels from answers.options and the published options in strata_read_case_intake>
Selection rule: <automatic selection for General investment and Multi-product; otherwise follow remaining needs>
Remaining intake needs: <returned needs>

Creation persists
- one case (case key, decision cycle) and first revision: title and objective from the brief, no alternatives;
- decision question, materiality statement, and counterfactual remain missing;
- the template's pinned configuration;
- no constraints;
- recorded template option selections and any mapped company-profile figures;
- the template-routing choice, audit events, and receipt.

Intake selections become saved case alternatives when a reviewed draft is accepted.
Next task: complete remaining intake needs, then request and review a draft.

Idempotency key: generated after confirmation; reused only for an unchanged retry.
```

If a correction changes one field, do not show only that field. Re-render the entire preview. A correction to the brief requires a new routing call before the preview, because the proposal came from the earlier description.

## Exact tool input

Both writes use `strata_record_case_intake`. Between routing and confirmation, use `strata_read_case_intake` with the chosen `templateKey`, original `brief`, and complete `routingResult` to read the selected option types, published labels, and remaining needs. The routing call runs before the preview. The creation call runs after the user confirms the preview and the proposed template.

Routing call:

```json
{
  "idempotencyKey": "<new UUID>",
  "changeReason": "<confirmed reason>",
  "brief": {
    "title": "<short case name>",
    "description": "<decision brief>"
  }
}
```

Creation call, after confirmation:

```json
{
  "idempotencyKey": "<new UUID>",
  "changeReason": "<confirmed reason>",
  "brief": {
    "title": "<confirmed short case name>",
    "description": "<confirmed decision brief>"
  },
  "answers": {},
  "routing": {
    "templateKey": "<templateKey from the routing result>",
    "templateRoutingJudgmentId": "<judgmentId from the routing result>"
  }
}
```

The routing call omits `answers` and `routing`. The creation call requires `answers` (send `{}`) and `routing`. Neither call accepts a case identifier, an identity field, or `null` for any property. Send the brief the user confirmed in the preview.

## Retry identity

The retry identity is the canonical confirmed tool input plus `idempotencyKey`.

- An unchanged retry after timeout or a lost response uses the same UUID, fields, array order, array items, and omitted properties.
- A correction to the brief or change reason, or a validation repair, changes intent and requires another complete preview, confirmation, and UUID.
- `replayed: true` means the original transaction already succeeded. Report its receipt and stop retrying.
- `COMMAND_REUSED` means the same UUID reached the server with different input. Do not guess which version won.

For a stale later edit, `currentCaseHead` or `currentSubjectHead` is remediation data, not permission to overwrite. Read the returned/current exact revision with `strata_read_case`, compare it with the user's intended change, show a new preview, reconfirm, and use a new UUID.

## Report the creation receipt

Report the exact case key and `caseRevisionNumber`. Report every entry in `structured_content.data.prefilled` as a company-profile figure that creation recorded as a key input. Report every entry in `structured_content.data.needs` as still unanswered. Explain that the receipt confirms the case and intake revision were recorded. An empty `needs` list means intake is complete; the alternatives, assumptions, and template calculation Model are adopted after a draft is reviewed and explicitly accepted. Offer to complete any remaining intake needs or request and review a draft next. Preserve saved answers. End tool use after creation; do not start the offered task in this turn.
