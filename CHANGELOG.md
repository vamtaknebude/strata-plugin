# Changelog

## 0.5.0-staging.9

The `strata`, `uprali-demo`, and `uprali-demo-chatgpt` packages carry the same
skill changes.

- When `strata_read_approval_path` is available and the user asks for a case's
  Decision or Outcome Review without a visible ID, `develop-business-case` and
  `understand-business-case` resolve the case and call
  `strata_read_approval_path` with its `investmentCaseId`. They pass the
  returned `decisionId` to `strata_read_decision` and the returned
  `outcomeReviewReference` to `strata_read_outcome_review`. When either value
  is `null`, they state that the case has no Decision or no Outcome Review
  (#1733).
- The Assumption Delegation reference of `develop-business-case` now covers
  Adoption. The skill finds adoptable work with
  `strata_list_assumption_delegations` and `view` `awaiting_adoption`, reads
  the delegation with `strata_read_assumption_delegation` before every
  attempt, presents its `adoptionPreview`, and calls
  `strata_adopt_assumption_delegation_submission` once after confirmation. It
  omits the adopt command when the preview has a blocker. On `ADOPTION_STALE`
  or `CASE_REVISION_CONFLICT` it reads fresh state and asks for a new
  confirmation. On `SUBMISSION_ADOPTED` or `COMMAND_REUSED` it stops without a
  second mutation (#1238).

## 0.5.0-staging.8

The `strata`, `uprali-demo`, and `uprali-demo-chatgpt` packages carry the same
skill changes.

- `start-business-case` now offers "Build initial business case" for every
  template, including `general_investment` when the presenter stated
  investment figures. For `general_investment`, it sends the stated figures in
  `investmentValues` in the `strata_recommend_case_intake` call, so the button
  build uses them. It then lists the figures from the result's
  `investmentFigures` and says which the presenter stated and which are
  recommended defaults. The skill no longer asks the presenter to confirm in
  chat when they stated figures. After a button build, it no longer says that
  every figure used its recommended default.
- When no App build reached the model context, `start-business-case` no longer
  asks whether the presenter pressed the button. It calls `strata_list_cases`
  and takes the newest case with the intake title. If that case exists, it
  continues with it. If no such case exists and the message confirms the
  intake, it builds the case in chat. Otherwise it answers the message and
  waits for a button press or a confirmation.
- Before they state an Assumption Card's status, `develop-business-case` and
  `understand-business-case` call `strata_list_assumption_cards` in the same
  turn. Before they state approval status or a step record, they call
  `strata_read_approval_path` in the same turn. This also applies after a press
  of an App button.
- When the user refers to the new case and the conversation has no build
  result, `develop-business-case` and `understand-business-case` take the
  newest case from `strata_list_cases`. When the conversation showed an intake,
  the case must also have the intake title. If no such case exists,
  `develop-business-case` says that no new case exists yet and offers to start
  one.
- When `strata_compare_scenario_to_base` is available and the user asks to
  compare a named scenario with Base, `understand-business-case` takes `runId`
  and `option.optionRevisionId` from the case's latest
  `strata_read_calculation_results` result. If there is none, it resolves the
  case and calls `strata_read_calculation_results` first. It then calls
  `strata_compare_scenario_to_base` once, with `scenarioLabel` set to the
  user's scenario name, and answers with its Pairwise Results, such as NPV and
  payback.
- Steps 5 and 6 of the `understand-business-case` Read workflow, which read the
  selected projection or protected record and pin follow-up reads, moved from
  `SKILL.md` to the "Read and pin exact revisions" section of
  `references/exact-reads-history-and-lineage.md`. Their rules are unchanged.

## 0.5.0-staging.7

The `strata`, `uprali-demo`, and `uprali-demo-chatgpt` packages carry the same
skill changes. The workspace App's "Build initial business case", "Confirm
card", "Confirm all cards", "Approve step", and "Send change request" buttons
now make their change in the App. The App reports each result to the agent
through its model context and sends no chat message.

- After a press of "Build initial business case", `start-business-case` does
  not call `strata_build_initial_case`. It reads the build's case key,
  `investmentCaseId`, and `runId` from the App's model context and continues
  with the results. When the presenter's next message arrives and no build
  reached the model context, it asks whether they pressed the button, unless
  the message already says so. If they did, it calls `strata_list_cases`, takes
  the newest case with the intake title, and reads its results with
  `strata_read_calculation_results`.
- A confirmation in chat builds from the recommendations as shown in the intake
  result. Edits made in the view apply only when the presenter presses the
  button.
- For `general_investment`, the button build uses the recommended default for
  every investment figure, and `start-business-case` says so after the build.
  When the presenter stated investment figures, the skill asks them to confirm
  in chat instead of pressing the button. The chat build sends those figures in
  `investmentValues`.
- "Confirm card" and "Confirm all cards" confirm the cards in the App at their
  listed revisions, one at a time, and stop at the first failure.
  `develop-business-case` reports the confirmed cards from the App's model
  context and sends no `strata_confirm_assumption_card` for them. When the user
  says they pressed one of these buttons and no confirmation reached the model
  context, or the press covered more cards than the context names, it calls
  `strata_list_assumption_cards` and reports each card's status.
- "Approve step" and "Send change request" record the step record in the App.
  `develop-business-case` reports the step record from the App's model context
  and sends no `strata_record_approval_step` for it. When the user says they
  pressed one of these buttons and no step record reached the model context, it
  calls `strata_read_approval_path` and reports the latest step records.

## 0.5.0-staging.6

The `strata`, `uprali-demo`, and `uprali-demo-chatgpt` packages carry the same
skill changes.

- The "Build initial business case" button in the intake view now sends a
  readable message instead of the build arguments as JSON. After that message,
  `start-business-case` takes the template, title, and description from the
  intake result. It takes the objectives from the message's "Objectives:" list.
  For each metric in the message's "Success metrics:" list, it copies the
  metric's title, objective, and key results from the intake result and takes
  its target and timeframe from the message.
- The `assumption-delegation` reference of `develop-business-case` adds
  `strata_adopt_assumption_delegation_submission` to its allowed tools. With it,
  the agent can adopt a delegation Submission as a new or replacement unlinked
  Assumption.

## 0.5.0-staging.5

The `strata`, `uprali-demo`, and `uprali-demo-chatgpt` packages carry the same
skill changes.

- Each skill's `SKILL.md` is now smaller than 8,000 bytes, the size at which Codex
  truncates a selected skill. Sections moved without change into new reference
  files: `normalized-draft` and `recovery` in `start-business-case`,
  `resolve-current-state` and `stop-and-recover` in `develop-business-case`, and
  `select-projection`, `history-and-comparison`, `evidence-and-restricted-data`,
  and `recovery` in `understand-business-case`.
- When the user asks to confirm several Assumption Cards at once,
  `develop-business-case` shows one preview of the named cards. After one
  approval, it sends one `strata_confirm_assumption_card` per named card, lists
  the cards again, and reports each card's status.
- When `strata_recommend_case_intake` and `strata_build_initial_case` are both
  available, `start-business-case` follows the new `demo-initial-case` reference.
  It takes the decision description from one message and asks no follow-up
  questions. It chooses a template, shows the recommended intake, builds the case
  after one confirmation, and opens the results view for the new calculation Run.
- When `strata_build_initial_case` is available, `develop-business-case` names
  records in its replies by case key, title, or label instead of ID. When a
  `capturedAt` or `observedAt` value has only a date, it sends that date at
  `T00:00:00Z`. Before it offers to submit a case for approval, it reads the
  case's approval path and first saves an approval path when the case has none.
- When `strata_read_calculation_results` is available, `develop-business-case`
  calls it for a Run whose results the user asks to see, and the App shows the
  results view.

## 0.5.0-staging.4

The `strata`, `uprali-demo`, and `uprali-demo-chatgpt` packages carry the same
skill changes.

- `develop-business-case` now calculates a case when
  `strata_create_calculation_run` is available. Earlier releases never
  calculated. The agent states only values that a Run returned. A Decision relies
  on calculated results only when it names a successful Run for the same Case
  Revision.
- When the user asks for a Run's calculation model as a file and
  `strata_export_calculation_model` is available, `develop-business-case`
  exports it as a CSV or XLSX file.
- Add the `approvals` reference: save an approval path, submit a case for
  approval, and record approval steps. An approval path never blocks or changes a
  Decision.
- Add the `assumption-cards` reference: group Assumptions into Assumption Cards
  and confirm a card.
- Add the `assumption-delegation` reference: delegate an Assumption to an
  Organization member, revise the request, save and submit a Response, return a
  Submission for correction, cancel the request, and list or read delegations.
- Add the `performance` reference: designate the revenue, ROI, and payback
  Outcome Commitments of a decided case, record its health, and read the
  performance registry.
- The approvals, Assumption Cards, and performance references apply only when all
  of their tools are available. Otherwise the agent tells the user that this
  Strata connection cannot record that data.
- `start-business-case` offers to record success metrics after it creates a case.
  The user can record or change them later. `develop-business-case` offers
  success metrics as starting points for the Outcome Commitments of an approve or
  pilot Decision.

## 0.5.0-staging.3

- The Uprali demo packages (`uprali-demo` and `uprali-demo-chatgpt`) now name
  their MCP server `uprali-demo`.

## 0.5.0-staging.2

- Add the Uprali demo plugin (`uprali-demo`) to the Claude Code marketplace. It
  installs the same three workflow skills and connects to the Uprali demo MCP
  server.
- Add the Uprali demo ChatGPT local plugin package (`uprali-demo-chatgpt`). It
  carries the same three workflow skills.

## 0.5.0-staging.1

- Label the plugin and both marketplaces as a staging preview.
- Declare the hosted MCP resource's current WorkOS Staging environment.
- Permit approved Strata operators to install and evaluate the proprietary
  package from a publicly readable repository.
- Document the final repository-visibility and private security-reporting checks.

## 0.4.0

- Publish the Strata Agent Plugin from a dedicated private distribution repository.
- Add native Claude Code and Codex marketplace metadata over one shared skill set.
- Document the Hermes Agent compatibility-preview path and browser OAuth setup.
