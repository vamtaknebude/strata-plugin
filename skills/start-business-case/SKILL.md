---
name: start-business-case
description: Elicit, check, preview, confirm, and create a coherent initial Strata business case whenever a user asks to create, start, build, frame, or draft a Strata case, including help turning an incomplete idea or decision brief into a case.
---

# Start Business Case

Guide the user from a decision idea to one complete Strata root revision. The root must contain a coherent narrative and 2–25 ordered, meaningfully distinct alternatives. Do not create an empty case, preparation record, conversational draft, evidence item, assumption, scenario, recommendation, approval, or final decision.

Use Strata MCP as the only case system. Do not run `pwd`, `ls`, `find`, `rg`, `env`, UUID utilities, or any other discovery or data command. After the host loads this skill and its linked reference, execute no shell or interpreter command for any reason, including status messages or UUID generation; send conversational updates directly. Do not inspect the repository, working directory, process environment, or public web. Do not call `list_mcp_resources` or `list_mcp_resource_templates`. Then use only the user's messages and the exact Strata tools named here.

## Organization and ownership

If the host or conversation has not established the active Strata organization, call `strata_connection_context` before any other Strata tool. Otherwise do not call it. Treat an explicit user or host statement that the active Strata organization and authenticated user are established as complete context: do not verify it with `strata_connection_context`, a list, or a read. The authenticated Strata user is the server-derived initial owner and author. Do not ask for or send organization, tenant, user, owner, author, case, cycle, revision, configuration, or command identifiers supplied by the user.

The business sponsor, accountable executive, affected stakeholders, and initial owner are different concepts. Capture stakeholder facts only when they materially affect the objective, constraints, alternative scope, rationale, or strategic effects. Never claim that a named sponsor becomes the server-owned initial owner.

Keep context, stakeholder needs, success criteria, constraints, dependencies, and uncertainties distinct before mapping them to tool fields. Put supplied success criteria in `objective`; put a stakeholder need in the field it actually qualifies; and preserve qualifying uncertainty language in `materialityStatement` or the affected alternative's `rationale`. Do not turn an uncertainty into a fact, hard constraint, or dependency. If it leaves the decision, baseline, or an alternative materially ambiguous, ask about it before previewing.

## Build the normalized draft

1. Extract only facts stated by the user or unambiguous safe inferences. Preserve wording the user supplies for a specific field unless it violates the tool schema or conflicts with another supplied field; do not editorialize it by adding punctuation, tense changes, or phrases such as “if we do nothing.” You may infer a concise title, an explicit alternative's type, and empty dependencies, constraints, or strategic effects. State every such inference in the preview. Do not invent dates, budgets, metrics, stakeholders, dependencies, effects, or business facts.
2. Keep the decision question separate from the objective. The question names the choice to make; the objective states the outcome and any supplied success criteria. Keep the problem or opportunity and why it matters in `materialityStatement`. Keep the no-action baseline in `counterfactual`.
3. Treat a deadline as optional only when time does not matter. If the user says “next quarter,” “before renewal,” or another date that cannot be normalized to `YYYY-MM-DD`, ask for the exact date.
4. Put cross-cutting hard bounds and in/out-of-scope facts in `constraints`. Put each alternative's actual delivery boundary in its `scope`. Put its reason for consideration in `rationale`; do not repeat the objective as a rationale.
5. Require 2–25 alternatives that differ in mechanism, provider, commitment, timing, or another material choice dimension. A renamed copy, a restated objective, or a “do the same thing better” variant is not distinct.
6. For every alternative, obtain or safely infer `label`, `optionType`, `scope`, `rationale`, `dependencies`, and `strategicEffects`. Include implementation timing only when both exact start and end dates are known and the start precedes the end. Otherwise omit both dates and say timing is not recorded.
7. Obtain a specific nonblank `changeReason` describing why this immutable root is being created.

Use the [coherent input, weak-input, preview, and recovery contract](references/coherent-initial-case.md) to map the conversation to the exact tool fields.

## Ask one focused question at a time

Before previewing, identify missing, placeholder, contradictory, or materially ambiguous facts. Ask exactly one question about the highest-priority material gap. Write it as one single-line interrogative sentence ending with exactly one question mark. Do not bundle independent questions, present a questionnaire, or ask for a value already supplied.

Use this priority:

1. decision question;
2. objective or supplied success criteria;
3. materiality and counterfactual;
4. an exact deadline when time matters;
5. cross-cutting scope or constraints;
6. fewer than two distinct alternatives;
7. an incomplete alternative, including a one-sided implementation period;
8. change reason.

After each answer, re-evaluate the whole draft and ask only the next material question. Do not preview while any required value is missing or contradictory.

## Preview every user-controlled persisted field

Show one complete preview immediately before confirmation. It must include:

- the authenticated Strata user as initial owner and author;
- `changeReason`;
- every narrative field: `title`, `decisionQuestion`, `objective`, `materialityStatement`, `counterfactual`, optional `deadline`, and the complete ordered `constraints` list;
- every ordered alternative with `label`, `optionType`, `scope`, `rationale`, complete `dependencies`, complete `strategicEffects`, and either both implementation dates or an explicit “not recorded” marker;
- every inferred empty list or other safe inference;
- that one confirmation creates the case, decision cycle, alternatives, owner fact, audit/outbox records, receipt, and one immutable root revision atomically; and
- that a fresh command UUID will be generated only after confirmation and retained only for an unchanged retry.

The preview must not hide a field behind “and other details,” summarize several alternatives into one, or show fields that will not be sent. When a host requires structured output, put the same complete content in its `preview` object and render a concise readable summary beside it.

## Confirm, create, and verify

Wait for an unambiguous affirmative response tied to the latest complete preview. A correction invalidates the prior preview: apply only the stated correction, show the complete corrected preview again, and wait for a new confirmation. A cancellation or hesitation creates no command UUID and calls no mutation tool.

After confirmation:

1. Construct one syntactically valid UUID directly as the `commandId` argument. Do not invoke a shell, Python, Node, `uuidgen`, a UUID utility, or any other tool to generate it.
2. The normal post-confirmation non-message action sequence is exactly one `strata_start_case` call. Do not call connection context, list, read, search, or any other Strata or host tool before or after it. Send exactly the confirmed `changeReason`, `narrative`, ordered `alternatives`, and that `commandId`. Only the ambiguous/lost-response recovery below may add one byte-identical retry.
3. Treat only a structured success receipt as success. Verify `operation` is `case.created`, the returned alternative count and ordinals match the preview, and retain the exact case key, case ID, cycle ID, root revision ID/number, every ordered alternative ID and revision ID/number, and replay status.
4. Tell the user what was created and name the exact case key and root revision. Offer a natural next task without starting it silently.

A normal successful journey uses at most one optional `strata_connection_context` call and one `strata_start_case` call. Do not list or read cases before creating a new one.

## Recovery

- `INPUT_INVALID`: preserve the conversational draft, identify the rejected field, correct it only from user facts, then show the complete preview and obtain confirmation again. Changed input receives a new command UUID.
- `PERSISTENCE_UNAVAILABLE`, `CAPABILITY_TIMEOUT`, a lost response, or an ambiguous transport result: retry at most once with byte-for-byte identical normalized input and the same command UUID. Accept `replayed: true` as the existing receipt; never create a second case.
- `COMMAND_REUSED`: stop. The UUID was associated with different input. Reconstruct the intended draft, show a new complete preview, obtain confirmation, and use a new UUID.
- authentication, organization, membership, or access failure: do not retry unchanged and do not reveal whether another organization's data exists.
- any stale-head result during a later edit: never merge silently. Read the exact current overview with `strata_read_case`, show the changed edit preview, obtain a new confirmation, and use a new command UUID. Do not turn that edit into another initial case.
- cancellation at any point before a success receipt: stop with no further tool call.

Never calculate a recommendation, claim the draft is objectively true, or convert prose quality into authorization. The server independently enforces deterministic validation and tenancy even when this skill is absent.
