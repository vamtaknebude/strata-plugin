# Scenario contract

Use this reference before previewing `strata_save_scenario` or a scenario removal.

## Contents

- [Scenario completeness](#scenario-completeness)
- [Operator compatibility](#operator-compatibility)
- [Preview](#preview)
- [Scenario input shapes](#scenario-input-shapes)
- [Removal](#removal)
- [Readiness](#readiness)

## Scenario completeness

A scenario has one kind (`base`, `bear`, `bull`, or `custom`), a nonblank label, rationale, and the complete ordered adjustment set. A Base scenario has no adjustments. Do not assume a scenario is Bear or Bull from its label when the user's direction is ambiguous.

Resolve each adjustment against the assumptions projection at the exact current case head. The selected assumption must be visible, current, and `isSensitivityDriver: true`. One scenario may adjust a driver only once.

For a scalar Scenario, use the stable `assumptionId` as the scalar command requires. For a series Scenario, read the current Scenario, the source assumptions, and the authoring Model coverage before composing the command. Use each source's stable `assumptionId`, exact `assumptionRevisionId`, and the returned `authoringModelRevisionId`. Keep every `relativePeriodKey` relative. Do not derive calendar dates.

## Operator compatibility

- Scalar `percentage_delta` accepts a decimal-ratio percentage change (`"0.05"` is a 5% increase; `"5.0"` would be 500%) and only a `numeric`, `numeric_range`, `money`, or `money_range` driver. Against a `money` or `money_range` driver it requires `roundingMode`: one of `up`, `down`, `halfUp`, `halfDown`, `halfEven`, `halfOdd`, `halfTowardsZero`, or `halfAwayFromZero` from the tool schema, for example `{ "assumptionId": "<id>", "operation": "percentage_delta", "roundingMode": "halfUp", "value": "0.05" }`. Against a `numeric` or `numeric_range` driver it must omit `roundingMode`.
- A scalar `numeric_delta` carries a delta in the driver's authored shape: a numeric value with its unit, or a `money` value `{ "kind": "money", "amount": "<decimal in major unit>", "currencyCode": "<supported code>" }` whose code exactly matches the driver's code (a `money_range` driver also takes a scalar `money` delta with its code).
- A series `replace` contains the complete compatible source series. It uses the source value kind and every covered Relative Period in source order.
- A series `numeric_delta` contains one typed delta for every covered Relative Period. Its series value declares the source `valueKind` (`"numeric"` or `"money"`); a numeric point retains the source `unitKey`, and a money series carries one `currencyCode` beside its points, for example `{ "valueKind": "money", "currencyCode": "EUR", "points": [{ "relativePeriodKey": "<covered Relative Period>", "value": { "kind": "money", "amount": "12.50" } }] }` (a per-point code is rejected).
- A series `percentage_delta` contains one decimal ratio for every covered Relative Period and requires a `numeric` driver; `numeric_range` and monetary series drivers are incompatible with `percentage_delta`.

Send only a currency code the tool schema publishes: the schema carries the supported-code enum, and a code outside that set is rejected. Do not list codes from memory.

Adjustments carry the authored money form when sending to `strata_save_scenario`; reads return the realized form with `amountCents` (or `lowerAmountCents`/`upperAmountCents`). Never send a realized field.

Refuse Not Applicable, masked, missing, or incompatible source facts. Do not convert units, currencies, value kinds, ranges, dates, or percentages. Never silently revise `isSensitivityDriver` to make an adjustment pass.

## Preview

```text
Next command: strata_save_scenario
Case head: <exact current revision>
Subject: <create, or scenario ID + exact current scenario revision>
Change reason: <exact text>
Kind / label / rationale: <exact values>
Adjustments, in order:
1. <stable assumption ID; current assumption revision shown for review>
   - operation: <percentage_delta, numeric_delta, or replace>
   - scalar value, or series: <source assumption revision; authoring Model revision; first and last Relative Period; every ordered point; compatible unit/currency when applicable>
Effect: creates one successor case revision containing this complete scenario overlay.
```

For a revision, show the stable Scenario ID and exact current Scenario revision in the preview. The eventual `strata_save_scenario` input must match the approved preview byte-for-byte except for one new `idempotencyKey`.

If a save reports a stale Case or Scenario head, make no retry in that turn. Read the current Scenario and source assumptions again. Rebuild and show a changed complete preview using the fresh heads and identities. Obtain a new explicit confirmation. Make one new save with a new idempotency key.

After success, retain the stable scenario ID, exact scenario revision, and successor case head. Retain the receipt. Offer an exact scenario read; do not apply the adjustments or calculate a result.

## Scenario input shapes

Scenario fields are never top-level tool arguments. The block is a non-executable shape template; never send its angle-bracket placeholder text. The create preview must use the exact `scenario` wrapper below and omit `idempotencyKey` until confirmation:

```json
{
  "investmentCaseId": "<investmentCaseId>",
  "decisionCycleId": "<decisionCycleId>",
  "expectedCaseRevisionId": "<current caseRevisionId>",
  "changeReason": "<change reason>",
  "scenario": {
    "kind": "bull",
    "label": "<label>",
    "rationale": "<rationale>",
    "adjustments": [
      {
        "assumptionId": "<stable driver assumption ID>",
        "operation": "percentage_delta",
        "value": "<decimal ratio: 0.05 is a 5% increase>"
      }
    ]
  }
}
```

Use the scalar shape only for scalar Scenario work. A series Scenario uses the existing series command contract. Its `adjustments` list contains one of the following complete shapes for each source driver:

```json
{
  "assumptionId": "<stable source assumption ID>",
  "assumptionRevisionId": "<exact source assumption revision ID>",
  "operation": "replace",
  "value": {
    "authoringModelRevisionId": "<returned authoring Model revision ID>",
    "firstRelativePeriodKey": "<first covered Relative Period>",
    "lastRelativePeriodKey": "<last covered Relative Period>",
    "kind": "series",
    "points": [
      {
        "relativePeriodKey": "<covered Relative Period>",
        "value": "<source-compatible value>"
      },
      {
        "relativePeriodKey": "<covered Relative Period>",
        "notApplicable": { "reason": "<source reason>" }
      }
    ]
  }
}
```

For `replace`, the series value and each point have the source value kind. Every point has `relativePeriodKey` and exactly one of `value` or `notApplicable`. For `numeric_delta`, use `operation: "numeric_delta"`, include the source `valueKind` (`"numeric"` or `"money"`), and provide points shaped as `{ "relativePeriodKey": "<covered Relative Period>", "value": "<compatible typed value>" }`, where a money series carries one `currencyCode` beside its points and each point value omits it. For `percentage_delta`, use `operation: "percentage_delta"` and provide points shaped as `{ "relativePeriodKey": "<covered Relative Period>", "value": "<decimal ratio>" }`. The first and last keys and every point must exactly cover the returned source series coverage.

A revision uses the same complete `scenario` object and adds these two top-level fields together:

```json
{
  "scenarioId": "<stable scenario ID>",
  "expectedScenarioRevisionId": "<current scenario revision ID>"
}
```

Those two revision fields extend the create shape; they do not replace or enter the `scenario` object.

## Removal

Before `strata_remove_scenario`, read the current scenario and show its stable ID, exact scenario revision, current case head, label, kind, and adjustment count. State that removal creates a successor case revision and does not remove assumptions or evidence. Obtain explicit confirmation and never cascade.

## Readiness

After the scenario and framing writes, read `strata_read_calculation_model` once with `projection: "readiness"` for the exact Investment Case, Decision Cycle, and the framing receipt's successor Case Revision. Report its `ready` or `blocked` result. On `blocked`, quote every returned blocker `code` exactly and invent no code. A blocked read ends the journey: issue no further call, repair nothing automatically, create no Run, and recommend no Option.
