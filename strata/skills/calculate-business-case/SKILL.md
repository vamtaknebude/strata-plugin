---
name: calculate-business-case
description: Check calculation readiness for one exact Strata Case Revision and run the confirmed complete calculation command. Use when a user asks to calculate, run, or refresh the numbers for a supplied exact Case Revision.
---

# Calculate Business Case

Run one exact Strata Case Revision through a single confirmed calculation command. Preserve revision identity and user control. Perform no calculation in the skill and recommend no Option. Do not widen the requested Option or Scenario product.

Use only Strata MCP, user messages, and host-loaded references. Do not inspect the repository, environment, filesystem, web, discovery, UUID, or resource-listing tools.

When the Strata tools are unavailable, Strata is not connected in this client. Stop before any other step and ask the user to connect it to `https://strata-utrtwerwt.sprava.ai/api/mcp`: in the Claude desktop app, open Customize, Connectors; in the Claude Code CLI, run `/mcp`; in another client, its own MCP sign-in.

Readiness and Run requests belong here. Requests to add, change, remove, or complete Models, evidence, assumptions, sensitivity drivers, scenarios, Assumption Reviews, Decisions, Outcome Reviews, Decision Learning, or exact Learning references belong to `develop-business-case`. Requests to explain current framing, alternatives, assumptions, drivers, scenarios, evidence lineage, revision history, comparisons, reviews, Decisions, Outcome Reviews, Decision Learnings, or exact Learning references belong to `understand-business-case`.

## Resolve exact selectors

Call `strata_connection_context` first only when the host or conversation has not established the active Strata organization. Do not verify explicit organization and user context.

Retain the Investment Case, Decision Cycle, and exact Case Revision IDs supplied by the user or authorized reads. Ask for missing or ambiguous selectors. Never replace an exact historical revision with current state.

## Read readiness without confirmation

Call `strata_read_calculation_model` with those selectors and `projection: readiness` without requesting confirmation. Explain the returned `result.status: ready` as eligibility for a complete Run, not a guarantee of successful evaluation. A blocked result or tool error ends this workflow with the observed status. Do not repair or retry.

## Propose the complete Run command

Construct a fresh lowercase RFC 4122 version 4 UUID directly as `idempotencyKey`. Its grammar is `xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx`, where `y` is `8`, `9`, `a`, or `b`. Never use a tool to generate it or reuse a key for a different command.

Present `strata_create_calculation_run` and one compact, complete JSON command with exactly these fields in this order: `investmentCaseId`, `decisionCycleId`, `caseRevisionId`, `idempotencyKey`. Explain that the command calculates the complete revision. Do not add subset selectors, release provenance, or tenant parameters.

Retain the exact compact proposal and wait for explicit confirmation. A general request to calculate is not confirmation of a proposal that did not yet exist. A refusal or changed instruction authorizes no write. A correction invalidates the pending proposal.

## Invoke the confirmed command once

Invoke `strata_create_calculation_run` once using the confirmed four fields and key unchanged, in the presented field order. Do not regenerate a key after confirmation.

Preserve the returned receipt and report creation from it. Claim successful calculation only when an authoritative `strata_read_calculation_run` summary reports success. End without retry, another Run, or downstream Decision work.

## Recover from a stale failure

When the confirmed creation returns a stale source or membership error (`DEPENDENCY_CONFLICT`, `CASE_REVISION_CONFLICT`, `MODEL_REVISION_CONFLICT`, or membership `ACCESS_DENIED`), report the observed error. Perform a new `strata_read_calculation_model` readiness read for the same three selector IDs. Present a new compact proposal carrying the same IDs and requested product with a fresh UUIDv4 key, and wait for a new explicit confirmation. Never silently reuse the old key or substitute current state for an exact historical revision. A blocked refreshed readiness ends the workflow with the observed status.

## Recover from an ambiguous response

When the creation response is lost or unreadable, replay exactly once using the byte-identical confirmed four-field command and key. Report creation from the returned receipt and confirm the retained Run with `strata_read_calculation_run`. Never construct a new command, regenerate the key, or start a second Run.

## Explain a blocked readiness

When the readiness read returns `result.status: blocked`, the workflow ends there: report that no Run was created. Name every entry of the returned `blockers` array in its returned order, each as its `code`, its `subject` kind and id (plus `dependency` kind and id when present), and its returned `diagnostic` restated in business terms without adding causes, values, or fixes. Never repair, never retry, never proceed to a proposal. Blocked readiness prevents Run creation; say so.

## Explain a configuration or limit failure

When the confirmed creation returns `CONFIGURATION_UNAVAILABLE` or `CALCULATION_LIMIT_EXCEEDED` (or any non-recoverable creation error outside the stale set), report the observed code in business terms and stop. Never present a changed proposal, never regenerate the key, never retry under a new key. These failures create no Run; say so with the unchanged durable count.

## Report an unsuccessful Run as retained

When the receipt reports `status: created` and the authoritative `strata_read_calculation_run` summary reports `status: unsuccessful` for the same `runId`, report the `runId`, the exact Case, Cycle, and Revision identities, and that the Run is retained and inspectable. Never retry, never start a second Run, never explain the failure cause beyond the returned summary.

## Distinguish a request that created no Run

When creation returns an error envelope (`isError: true`, e.g. `CALCULATION_NOT_READY`), report that no Run was created and the durable count is unchanged. State explicitly which of the three occurred: blocked readiness (no proposal was confirmed), a failed request (no Run exists), or a retained unsuccessful Run (a `runId` exists).
