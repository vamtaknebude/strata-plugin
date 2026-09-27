# Demo initial case

Use this reference only when `strata_recommend_case_intake` and `strata_build_initial_case` are both available. It replaces the scope statement at the top of `SKILL.md` and every `SKILL.md` section from "Creation invariant" on; the tool-use and connection rules in `SKILL.md` still apply. If either tool is missing, follow `SKILL.md` and ignore this reference.

## Take the description

Take the presenter's complete decision description from one message. Ask no follow-up questions.

Choose `templateKey` by the decision's subject: `support_ai_optimization` for support-automation decisions, and `general_investment` for every other decision.

Infer a concise title of at most 200 characters and tell the presenter the title you chose.

## Show the intake

Call `strata_recommend_case_intake` once with `templateKey`, `title`, and the presenter's description as given in `description`. When the description exceeds 1,200 characters, shorten it to fit and tell the presenter that you shortened it.

Tell the presenter that every item shown is a recommendation. Then tell them how to confirm:

- For `support_ai_optimization`, and for `general_investment` when the presenter stated no investment figures: they can adjust the items in the view and press "Build initial business case", which builds the case in the App, or reply to confirm the items as shown.
- For `general_investment` when the presenter stated investment figures: they reply in chat to confirm. The build uses the items as shown, because the button builds with the recommended default for every figure.

For `general_investment`, list the investment figures you took from the description: upfront investment, Year 1 to 3 annual benefit, Year 1 to 3 annual operating cost, and discount rate. State that the build uses its recommended default for each figure the presenter did not give; an annual series gets defaults for all three years unless the presenter gave all three.

## Build after one confirmation

A press of "Build initial business case", or the presenter's confirmation in chat, is the one confirmation. Show no separate preview.

A button press sends no chat message. The App builds the case with the presenter's selections and edits, shows its results view, and puts the build in its model context with the case key, `investmentCaseId`, and `runId`. The model context keeps that build until the App's next action; report it once. After a press, the case exists: continue at "Show the results" with that `runId`, and leave `strata_build_initial_case` uncalled. For `general_investment`, tell the presenter that the button build used the recommended default for every investment figure.

When the presenter's next message arrives and no App build reached you in model context, ask whether they pressed the button, unless their message says so. After a press, call `strata_list_cases`, take the newest case whose title is the intake title, and call `strata_read_calculation_results` with its `investmentCaseId`.

After a confirmation in chat, call `strata_build_initial_case` once, with a new lowercase RFC 4122 version 4 UUID constructed directly as `idempotencyKey`. Take `templateKey`, `title`, `description`, `objectives`, and `successMetrics` from the intake result unchanged. Tell the presenter that you are building from the recommendations as shown. For `general_investment`, add `investmentValues` with only the figures the presenter stated: `upfrontInvestment`, `discountRate` as a ratio such as `0.10`, and each of `annualBenefit` and `annualOperatingCost` as three amounts for Years 1, 2, and 3 when the presenter gave all three years of it. Omit every other figure.

## Show the results

After the build succeeds, call `strata_read_calculation_results` with the build's `runId`; the App shows the results view. Report the case key and the values of the "Headline results" group exactly as the tools returned them. Offer to review the Assumption Cards next, and wait for the presenter to choose.

## Replies

Name each record in replies by its case key, title, or label, not by its ID (UUID).

Before you call `strata_submit_case_for_approval`, call `strata_read_approval_path` for the case. The build saves an approval path.

When a Uprali call fails, follow [the recovery rules](recovery.md). Where they say to show the complete preview and reconfirm, call `strata_recommend_case_intake` again and wait for a new confirmation.
