# Intake input and preview contract

Use this reference for internal `strata_record_case_intake` and `strata_read_case_intake` inputs, host-specific previews, plain-language explanations, and post-creation reporting. The tool declarations remain authoritative for mechanical bounds.

## Contents

- [Brief and internal change reason](#brief-and-internal-change-reason)
- [Weak-input diagnostics](#weak-input-diagnostics)
- [Intake selection and lifecycle](#intake-selection-and-lifecycle)
- [Plain-language preview](#plain-language-preview)
- [MCP App panel preview and edits](#mcp-app-panel-preview-and-edits)
- [Behind-the-scenes creation behavior](#behind-the-scenes-creation-behavior)
- [Exact tool input](#exact-tool-input)
- [Retry identity](#retry-identity)
- [Report the creation receipt](#report-the-creation-receipt)

## Brief and internal change reason

`brief.title` is a short noun phrase identifying the case (1–200 characters).

`brief.description` is the decision brief in the member's words (1–2000 characters). Routing reads it to propose a template, and creation saves it on the initial revision. Preserve concrete measures, thresholds, service levels, and volunteered extra details (reasons for starting the case, constraints, deadlines, or alternatives) within 2000 characters without inventing facts or metrics. Leave unanswered framing details unset until post-creation editing.

`changeReason` is a nonblank internal string (1–1000 trimmed characters) required by the tool input schema. Never ask the member for a change reason or display it in any preview or report. Supply a clear nonblank internal value behind the scenes in routing and creation calls (for example `"Propose template for initial business case intake"` for routing and `"Create initial business case from confirmed brief"` for creation).

The authenticated Uprali user is the server-derived owner and author. Keep identity, tenancy, and internal schema labels strictly behind the scenes; never substitute a named sponsor or stakeholder, and never display owner/author, change-reason, schema-field, raw identifier, confidence, or retry labels to the member.

## Weak-input diagnostics

Ask one focused question when any of these conditions holds:

- `brief.description` names a topic without a decision;
- `brief.title` is absent and the description supports no concise inferred title; or
- `brief.description` exceeds 2000 characters (including volunteered details) or `brief.title` exceeds 200 characters.

Placeholders such as "to be determined" and "etc." are not facts. Do not ask separate questions for optional details such as framing statements, deadlines, constraints, or alternatives; tell the member in plain language that extra details can be added after the case is created.

## Intake selection and lifecycle

After `kind: "proposal"`, call `strata_read_case_intake` with the chosen `templateKey`, the original `brief`, and the complete `routingResult`. Match `answers.options` to published `options` (`label ?? optionType`) when you need published option names. Call `strata_read_case_intake` again with the updated `templateKey`, unchanged `brief`, and complete `routingResult` if the offered template changes.

Unless the member explicitly asks before creation which template options creation selects, keep non-App pre-creation previews limited to the items in [Plain-language preview](#plain-language-preview). Do not output option dumps, selection rules, or remaining-needs lists in the default preview.

When the member explicitly asks before creation which template options creation selects, or asks after creation why no saved alternatives appear yet, answer in ordinary plain business language using the published option names from `strata_read_case_intake`:

- **Automatic option selection:** General investment (`Status quo`, `Invest`) and Multi-product (`Status quo`, `Forecast`) automatically select every published option; no repeated option selection is needed.
- **Support-AI (`Status quo`, `Buy`, `Build`):** creation from chat (`answers: {}`) leaves option selection unanswered until remaining intake questions are completed, whereas the MCP App creation panel pre-selects `Status quo` and `Buy` unless changed in the panel.
- **Draft-acceptance lifecycle:** initial intake selections become saved case alternatives—and the template's assumptions and calculation model are adopted, enabling calculation—only once a draft is reviewed and explicitly accepted. Zero saved alternatives before draft acceptance is expected and does not indicate lost options or a broken case; do not invent repair steps or ask again for an already satisfied selection.
- When answering an on-demand question while an MCP App creation panel is visible, answer in ordinary chat text (never a code block), expose no internal persistence steps, and do not disable or replace the active creation panel.

Offer supported next steps (completing remaining questions or requesting and reviewing a draft) only after creation, without starting that next task in the same turn.

## Plain-language preview

Use this format only when the host lacks MCP App support, fails to render the creation panel, or the member reports no panel is visible. Present one preview in ordinary chat text—never inside a code block—containing only:

1. a short lead naming the case that will be created in Uprali (for example, `Here is the business case I will create in Uprali:`);
2. `Name: <case name>`;
3. `Decision: <decision text>`;
4. `Type of case: <human-readable business name from routingResult.templates, without raw keys or confidence scores>`;
5. a plain sentence noting that the member can change the name and decision text after the case is created (plus, when the member volunteered reasons, constraints, deadlines, or alternatives, that extra details can be added after creation); and
6. a confirmation question such as `Shall I create it?`.

Do not include owner/author, change reason, brief schema labels, objective-mapping notes, raw template or judgment identifiers, routing confidence scores, internal persistence lists, or idempotency and retry rules.

**Corrections and template switches without a panel:**

- If the member changes the case name or picks another template offered by routing, update and re-render the complete plain-language preview without re-running routing (calling `strata_read_case_intake` behind the scenes if the template changed).
- If the member changes the decision text, re-run routing with the updated brief, call `strata_read_case_intake` on `kind: "proposal"`, and show the updated complete plain-language preview.
- Create only after explicit confirmation of the latest preview.

## MCP App panel preview and edits

Neither `strata_connection_context` nor `strata_read_case_intake` returns an App-support flag, so rely on host-level panel visibility and member feedback.

After `strata_record_case_intake` returns `kind: "proposal"`, call `strata_read_case_intake` with `templateKey`, `brief`, and the complete `routingResult` so MCP App hosts render the creation panel with **Case name** prefilled and **Create business case** active.

When the host renders the creation panel:

- Treat the creation panel as the preview. Send at most a brief plain-language pointer to review or edit the case name (or change the template) and click **Create business case** in the panel.
- Emit no duplicate chat preview and no code block, and ask for no typed chat confirmation.
- Edits to the case name made directly in the panel require no routing re-check.
- If the member asks in chat to switch to another template offered in `routingResult.templates`, refresh the panel by calling `strata_read_case_intake` with that `templateKey`, the `brief` unchanged, and the complete `routingResult`.
- If the member asks in chat to change the decision text before creating, re-evaluate template routing with `strata_record_case_intake`, then refresh the panel via `strata_read_case_intake` on `kind: "proposal"` (stop in plain language on `kind: "no-fit"`).
- When the panel reports completion, acknowledge the saved case name and case identity in plain business language without asking for another confirmation or calling `strata_record_case_intake` again.

## Behind-the-scenes creation behavior

Keep these server mechanics internal; never show a persistence list or database object breakdown to the member:

- Routing records one template routing judgment and creates no case.
- Creation atomically writes the case identity, initial revision (`brief.title` and `brief.description`, with decision question, materiality statement, counterfactual left `null`, and empty constraints until later edits), pinned template configuration, initial intake answers (auto-selecting published options for General investment and Multi-product, and applying matching company-profile facts into `prefilled`), and the `case.intake.recorded` receipt with remaining `needs`.

## Exact tool input

Both writes use `strata_record_case_intake`. Between routing and confirmation, call `strata_read_case_intake` with `templateKey`, `brief`, and the complete `routingResult`.

Routing call:

```json
{
  "idempotencyKey": "<new UUID>",
  "changeReason": "<internal nonblank reason>",
  "brief": {
    "title": "<short case name>",
    "description": "<decision brief>"
  }
}
```

Creation call, after explicit chat confirmation when no panel is active:

```json
{
  "idempotencyKey": "<new UUID>",
  "changeReason": "<internal nonblank reason>",
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

The routing call omits `answers` and `routing`. The creation call requires `answers` (send `{}`) and `routing`. Neither call accepts a case identifier, an identity field, or `null` for any property. Send the brief confirmed in the latest preview.

## Retry identity

Keep retry identity strictly behind the scenes:

- An unchanged retry after timeout or a lost response uses the same UUID, fields, array order, array items, and omitted properties.
- A correction to the brief or template changes intent and requires updating the preview (panel refresh in App hosts or complete plain-language preview and reconfirmation in non-App hosts) plus a new UUID on the next write.
- `replayed: true` means the original transaction already succeeded; report its safe receipt identifiers and stop retrying.
- `COMMAND_REUSED` means the same UUID reached the server with different input; do not guess which version won.

For a stale later edit, `currentCaseHead` or `currentSubjectHead` is remediation data, not permission to overwrite. Read the current exact revision with `strata_read_case`, compare it with the member's intended change, show a plain-language edit preview, reconfirm, and use a new UUID behind the scenes.

## Report the creation receipt

After chat-executed creation succeeds, stop tool use and give a plain-language report that:

- states only the safe saved case identity (`caseKey` and `caseRevisionNumber` from `structured_content.data.receipt`, never `authorUserId`, `changeReason`, `idempotencyKey`, or `replayed`);
- names any selected options using published option names from `strata_read_case_intake`, stating when options were selected automatically (General investment and Multi-product) or noting when option selection remains unanswered (Support-AI created from chat);
- explains that initial intake selections become saved alternatives—and template assumptions and the calculation model are adopted, enabling calculation—only once a draft is reviewed and explicitly accepted;
- reports any applied company-profile figures (`structured_content.data.prefilled`) and remaining unanswered questions (`structured_content.data.needs`) in plain language; and
- offers supported next steps (completing remaining questions or requesting and reviewing a draft) without starting that next task in this turn.
