# Intake input contract

Use this reference to construct the exact `strata_record_case_intake` routing and creation inputs and to preview them. The tool declaration remains authoritative for mechanical bounds.

## Contents

- [Brief and change reason](#brief-and-change-reason)
- [Weak-input diagnostics](#weak-input-diagnostics)
- [What creation persists](#what-creation-persists)
- [Exact preview template](#exact-preview-template)
- [Exact tool input](#exact-tool-input)
- [Retry identity](#retry-identity)
- [Success metrics after creation](#success-metrics-after-creation)

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

## What creation persists

The creation call writes, in one transaction (`executeIntakeStartTransaction`, `src/server/case-graph/case-repository.ts:2132`; `insertShellCaseRevision`, `src/server/case-graph/case-repository.ts:1980`):

- the case identity: a case key, a decision cycle, and the authenticated user as owner;
- the first revision, with `title` from `brief.title`, `objective` from `brief.description`, no alternatives, and the configuration pinned for the confirmed template;
- fixed default narrative text from `INTAKE_NARRATIVE_DEFAULTS` (`src/server/case-drafting/intake.ts:213`): decision question "Which option should the organization choose?", materiality statement "This decision has a material impact on the organization.", counterfactual "Continue with the current approach.", and no constraints;
- one intake-answers row. Creation sends `answers: {}`, so the row holds no member answers, plus any company-profile figures that `applyProfilePrefill` maps into key inputs (`src/server/case-drafting/intake.ts:336`, reported as `prefilled`);
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

Creation persists
- one case (case key, decision cycle) and first revision: title and objective from the brief, no alternatives;
- the template's pinned configuration;
- fixed placeholder text for decision question, materiality, and counterfactual, and no constraints (revisable after creation);
- empty intake answers, plus any mapped company-profile figures;
- the template-routing choice, audit events, and receipt.

Idempotency key: generated after confirmation; reused only for an unchanged retry.
```

If a correction changes one field, do not show only that field. Re-render the entire preview. A correction to the brief requires a new routing call before the preview, because the proposal came from the earlier description.

## Exact tool input

Both calls use `strata_record_case_intake`. The routing call runs before the preview. The creation call runs after the user confirms the preview and the proposed template.

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

## Success metrics after creation

Apply this section only when `strata_save_success_metrics` is available. When it is not, the case creation ends the tool use.

After the creation call succeeds, offer to record the case's success metrics. Each metric has a `title`, an `objective`, one or more `keyResults`, a `target`, and a `timeframe`, taken from the user's words. Preview the complete ordered list and save it only after the user confirms it. For a new case, send `expectedSuccessMetricsRevisionNumber: null`, the returned `investmentCaseId`, the confirmed `metrics`, and a new `idempotencyKey`. When the user declines, record nothing.

The user may also record or change success metrics later. Read the current list with `strata_read_success_metrics`, preview the complete replacement list, and after confirmation save it with the returned `successMetricsRevisionNumber` as `expectedSuccessMetricsRevisionNumber`. The saved list replaces the whole earlier list, so include every metric the user keeps.
