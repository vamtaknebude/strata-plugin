# Changelog

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
