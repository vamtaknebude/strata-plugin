# Demo initial case

Use this reference only when `strata_recommend_case_intake` and `strata_build_initial_case` are both available. It replaces the scope statement at the top of `SKILL.md` and every `SKILL.md` section from "Creation invariant" on; the tool-use and connection rules in `SKILL.md` still apply. If either tool is missing, follow `SKILL.md` and ignore this reference.

## Take the description

Take the presenter's complete decision description from one message. Ask no follow-up questions.

Choose `templateKey` by the decision's subject: `support_ai_optimization` for support-automation decisions, and `general_investment` for every other decision.

Infer a concise title of at most 200 characters and tell the presenter the title you chose.

## Show the intake

Call `strata_recommend_case_intake` once with `templateKey`, `title`, and the presenter's description as given in `description`. When the description exceeds 1,200 characters, shorten it to fit and tell the presenter that you shortened it.

Tell the presenter that every item shown is a recommendation, that they can adjust it in the view, and that they can press "Build initial business case" or reply to confirm.

For `general_investment`, list the investment figures you took from the description: upfront investment, Year 1 to 3 annual benefit, Year 1 to 3 annual operating cost, and discount rate. State that the build uses its recommended default for each figure the presenter did not give; an annual series gets defaults for all three years unless the presenter gave all three.

## Build after one confirmation

The message the "Build initial business case" button sends, or the presenter's confirmation in chat, is the one confirmation. Show no separate preview.

Call `strata_build_initial_case` once, with a new lowercase RFC 4122 version 4 UUID constructed directly as `idempotencyKey`. Take `templateKey`, `title`, `description`, `objectives`, and `successMetrics` as follows:

- After the button message, take `templateKey`, `title`, and `description` from the intake result unchanged. Set `objectives` to the objectives the message lists under "Objectives:", in the listed order, each exactly as written after its leading "- ". Set `successMetrics` to one entry for each metric the message lists under "Success metrics:", in the listed order. Each listed metric starts with "- " and the `title` of one success metric in the intake result: copy that metric's `title`, `objective`, and `keyResults` from the intake result unchanged, and set `target` and `timeframe` to the text after "Target:" and "Timeframe:" under the listed title. When the message says "Success metrics: none.", set `successMetrics` to an empty list.
- After a confirmation in chat, use these fields from the intake result unchanged. Tell the presenter that you are building from the recommendations as shown. Changes made in the view reach you only through the button message.

In both cases, for `general_investment`, add `investmentValues` with only the figures the presenter stated: `upfrontInvestment`, `discountRate` as a ratio such as `0.10`, and each of `annualBenefit` and `annualOperatingCost` as three amounts for Years 1, 2, and 3 when the presenter gave all three years of it. Omit every other figure.

## Show the results

After the build succeeds, call `strata_read_calculation_results` with the returned `runId`; the App shows the results view. Report the case key and the values of the "Headline results" group exactly as the tools returned them. Offer to review the Assumption Cards next, and wait for the presenter to choose.

## Replies

Name each record in replies by its case key, title, or label, not by its ID (UUID).

Before you call `strata_submit_case_for_approval`, call `strata_read_approval_path` for the case. The build saves an approval path.

When a Strata call fails, follow [the recovery rules](recovery.md). Where they say to show the complete preview and reconfirm, call `strata_recommend_case_intake` again and wait for a new confirmation.
