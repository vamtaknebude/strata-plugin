# Scenario contract

Use this reference before previewing `strata_save_scenario` or a scenario removal.

## Contents

- [Scenario completeness](#scenario-completeness)
- [Operator compatibility](#operator-compatibility)
- [Preview](#preview)
- [Scenario input shapes](#scenario-input-shapes)
- [Removal](#removal)

## Scenario completeness

A scenario has one kind (`base`, `bear`, `bull`, or `custom`), a nonblank label, rationale, and the complete ordered adjustment set. A Base scenario has no adjustments. Do not assume a scenario is Bear or Bull from its label when the user's direction is ambiguous.

Resolve each adjustment against the assumptions projection at the exact current case head. The selected assumption must be visible, current, and `isSensitivityDriver: true`. Use its stable `assumptionId`; the server binds the exact current assumption revision. One scenario may adjust a driver only once.

## Operator compatibility

- `percentage_delta` accepts a decimal-string percentage change and only a numeric or numeric-range driver.
- `numeric_delta` accepts a numeric value whose `unitKey` and optional `currencyCode` exactly match the numeric or numeric-range driver.
- `replace` accepts a value with the same value kind as the driver. Numeric replacements also require matching unit and currency.

Do not convert units, currencies, value kinds, ranges, dates, or percentages. Ask the user to supply the exact compatible representation. Never adjust a masked or missing driver and never silently revise `isSensitivityDriver` to make an adjustment pass.

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
   - value: <complete exact value and compatible unit/currency when applicable>
Effect: creates one successor case revision containing this complete scenario overlay.
```

After success, retain the stable scenario ID, exact scenario revision, and successor case head. Offer an exact scenario read; do not apply the adjustments or calculate a result.

## Scenario input shapes

Scenario fields are never top-level tool arguments. The block is a non-executable shape template; never send its angle-bracket placeholder text. The create preview must use the exact `scenario` wrapper below and omit `commandId` until confirmation:

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
        "value": "<decimal percentage string>"
      }
    ]
  }
}
```

Use the operator-compatible adjustment shape instead of the percentage example when required. A revision uses the same complete `scenario` object and adds these two top-level fields together:

```json
{
  "scenarioId": "<stable scenario ID>",
  "expectedScenarioRevisionId": "<current scenario revision ID>"
}
```

Those two revision fields extend the create shape; they do not replace or enter the `scenario` object.

## Removal

Before `strata_remove_scenario`, read the current scenario and show its stable ID, exact scenario revision, current case head, label, kind, and adjustment count. State that removal creates a successor case revision and does not remove assumptions or evidence. Obtain explicit confirmation and never cascade.
