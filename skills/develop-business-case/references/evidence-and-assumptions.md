# Evidence and assumption contract

Use this reference before previewing `strata_register_evidence` or `strata_save_assumption`. The tool declaration owns mechanical bounds; this reference owns semantic distinctions and completeness.

## Contents

- [Evidence qualification](#evidence-qualification)
- [Evidence preview](#evidence-preview)
- [Evidence input shape](#evidence-input-shape)
- [Assumption completeness](#assumption-completeness)
- [Typed values](#typed-values)
- [Evidence roles and estimates](#evidence-roles-and-estimates)
- [Assumption preview](#assumption-preview)
- [Assumption input shapes](#assumption-input-shapes)

## Evidence qualification

An evidence registration stores inert metadata only. It does not fetch, upload, parse, authenticate to, or validate the truth of a source.

For a URL source require:

- exact HTTPS `uri`;
- exact `capturedAt` instant with UTC offset;
- nonblank `citation` identifying what the user is citing; and
- disclosure `public` or `restricted` with a supplied `restrictedGroupKey`.

For a manual statement require:

- nonblank `manualDescription` naming the statement and its origin;
- exact `capturedAt` instant;
- nonblank `citation`; and
- disclosure.

Record `locator`, `excerpt`, `observedAt`, and `supersedesEvidenceItemId` only when supplied or resolved from authorized Strata output. Do not treat an inaccessible URL, pasted title, or assistant summary as source content. Ask for the missing metadata instead of inventing it.

## Evidence preview

Render every field exactly:

```text
Next command: strata_register_evidence
Case: <authorized exact case identity>
Change reason: <exact text>
Source: <url + HTTPS URI + capturedAt, or manual_statement + description + capturedAt>
Citation: <exact text>
Disclosure: <public, or restricted + exact group key>
Locator / excerpt / observedAt / supersedes: <exact values or not recorded>
Effect: creates immutable inert evidence metadata; creates no case revision.
Command ID: generated only after confirmation; reused only for one unchanged ambiguous retry.
```

After success, retain `evidenceItemId` and `sourceArtifactId`. Link the evidence item, not the URL, to an assumption.

## Evidence input shape

The executable preview for a URL source has this exact nesting. The block is a non-executable shape template: replace every angle-bracket placeholder with authorized IDs and supplied values before previewing or calling a tool. Omit an optional evidence field rather than inventing it. The preview deliberately omits `commandId`; add that one field only after confirmation.

```json
{
  "investmentCaseId": "<investmentCaseId>",
  "changeReason": "<change reason>",
  "evidence": {
    "citation": "<citation>",
    "disclosure": { "kind": "public" },
    "locator": "<locator when supplied>",
    "observedAt": "<instant when supplied>",
    "source": {
      "kind": "url",
      "uri": "<https URI>",
      "capturedAt": "<instant>"
    }
  }
}
```

For a manual statement, replace only `evidence.source` with `{"kind":"manual_statement","manualDescription":"<supplied description>","capturedAt":"<instant>"}`. For restricted evidence, use `{"kind":"restricted","restrictedGroupKey":"<authorized group key>"}` as `evidence.disclosure`.

## Assumption completeness

One assumption preview must include:

- exact current `expectedCaseRevisionId`;
- `caseAssumptionId` and `expectedAssumptionRevisionId` together only for a revision;
- `fieldKey` selecting the configured case field;
- stable `assumptionKey`;
- `scope` as shared or one exact visible alternative ID;
- the authenticated Strata user as the server-derived accountable owner; `ownerUserId` is not a public tool input;
- one typed `value`;
- `applicablePeriod` with exact forward-ordered dates when the value is period-specific;
- `confidenceLevel` and concrete `confidenceRationale`;
- decision-use `rationale`;
- complete unique evidence links with roles; and
- explicit `isSensitivityDriver`.

Do not infer that every assumption is flexible. Driver designation is an explicit persisted fact. Do not link the same evidence item twice under different roles in one assumption.

## Typed values

Use exactly one value kind:

- `numeric`: decimal string plus `unitKey` of `count`, `currency`, or `percent`; include a three-letter uppercase `currencyCode` only for a currency value;
- `numeric_range`: lower and upper decimal strings, interval bounds, the same unit rules, and lower below upper;
- `text`: supplied text;
- `boolean`: literal true or false;
- `date`: exact `YYYY-MM-DD`;
- `date_range`: exact lower/upper dates with `[)` bounds and lower before upper.

Never convert “about one million,” fiscal quarters, local currency labels, percentages expressed as fractions, or date phrases without confirmation of the exact persisted representation. Preserve the user's unit and currency semantics; a number without its intended unit is incomplete.

## Evidence roles and estimates

Each evidence link has one role:

- `supporting`: directly supports using the value;
- `contradicting`: materially challenges the value;
- `contextual`: informs the assumption without directly proving it; or
- `superseding`: replaces earlier evidence for this assumption.

Infer a role only when the user's language is unambiguous and disclose the inference. Preserve contradicting evidence; do not omit it to make the assumption look stronger.

An estimate may be usable when the user explicitly labels it and provides manual-statement metadata, citation, disclosure, confidence rationale, and decision-use rationale. Never turn an unlabeled estimate, assistant guess, or unsupported conversion into evidence.

## Assumption preview

```text
Next command: strata_save_assumption
Case head: <exact current revision>
Subject: <create, or assumption ID + exact current subject revision>
Change reason: <exact text>
Field / stable key / scope / owner: <exact values>
Value / unit / currency / period: <complete typed representation>
Confidence / confidence rationale: <exact values>
Decision-use rationale: <exact text>
Evidence links, in order: <every evidence item ID + role>
Sensitivity driver: <true or false>
Effect: creates one successor case revision containing the complete assumption.
```

Render the owner as “authenticated Strata user (assigned by the server).” Do not put `ownerUserId` in the executable preview JSON or tool call.

A successful receipt supplies the stable assumption ID, exact assumption revision, and successor case head. Use those exact IDs in later reads and scenarios.

## Assumption input shapes

Assumption fields are never top-level tool arguments. The block is a non-executable shape template; never send its angle-bracket placeholder text. The create preview must use the exact `assumption` wrapper below and omit `commandId` until confirmation:

```json
{
  "investmentCaseId": "<investmentCaseId>",
  "decisionCycleId": "<decisionCycleId>",
  "expectedCaseRevisionId": "<current caseRevisionId>",
  "changeReason": "<change reason>",
  "assumption": {
    "fieldKey": "<configured field key>",
    "assumptionKey": "<stable key>",
    "scope": { "kind": "shared" },
    "value": {
      "kind": "numeric",
      "value": "<decimal string>",
      "unitKey": "currency",
      "currencyCode": "<ISO currency>"
    },
    "applicablePeriod": { "start": "<YYYY-MM-DD>", "end": "<YYYY-MM-DD>" },
    "confidenceLevel": "<low, medium, or high>",
    "confidenceRationale": "<rationale>",
    "rationale": "<decision-use rationale>",
    "evidence": [
      { "evidenceItemId": "<evidence item ID>", "role": "supporting" }
    ],
    "isSensitivityDriver": true
  }
}
```

Use the selected typed-value shape in place of the numeric example. For option scope, replace `scope` with `{"kind":"option","optionId":"<visible alternative ID>"}`. A revision uses the same complete `assumption` object and adds these two top-level fields together:

```json
{
  "caseAssumptionId": "<stable assumption ID>",
  "expectedAssumptionRevisionId": "<current assumption revision ID>"
}
```

Those two revision fields extend the create shape; they do not replace or enter the `assumption` object.
