# Changelog

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
