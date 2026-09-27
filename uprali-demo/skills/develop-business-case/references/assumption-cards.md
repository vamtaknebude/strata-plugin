# Assumption Cards

Use this reference only when `strata_save_assumption_card`, `strata_confirm_assumption_card`, and `strata_list_assumption_cards` are all available. If any of the three tools is missing, do not offer Assumption Cards. If the user asks for them, say that this Uprali connection cannot record them, and continue with the rest of the request.

## Concepts

- An Assumption Card is a named group of Assumptions on one case. It has a title, a description, a responsible member, a headline metric with a label and a plain-language formula, and the keys of its Assumptions.
- The responsible member is information only. It does not restrict who may save or confirm the card.
- The card status is derived from the Assumptions on the card. `pending` means the card was never confirmed. `modified` means an Assumption on the card has a revision newer than the confirmed one, or was placed on the card after the confirmation. `confirmed` means neither. Revising an Assumption with `strata_save_assumption` makes every confirmed card that holds it `modified` until the card is confirmed again. Removing an Assumption from the card, or changing its title, description, headline metric, or responsible member, does not change its status.

## Read before each write

Call `strata_list_assumption_cards` with the exact `investmentCaseId` before each card write. To revise or confirm a card, pass its `assumptionCardId` and its `cardRevisionNumber` from that list as `expectedCardRevisionNumber`. To create a card, pass `null` for both.

## Save a card

Each save writes a complete new card revision. Take the title, headline label, formula, and responsible member from the user; if one is missing, ask for it and do not write it yourself. The description may be an empty string. Set `headlineMetric.modelItemKey` to `null` unless the user names a Model Item key.

When you revise a card, copy every field the user did not change from the card in `strata_list_assumption_cards`, including `responsibleMember.userId` as `responsibleMemberUserId`. Call `strata_list_organization_members` only when the user names a new responsible member.

The `assumptionKeys` you send replace the card's complete Assumption list. Take each key from the case's Assumptions, read with `strata_read_case` and the assumptions projection; follow the returned cursor until you find every key the user named.

Show the title, description, responsible member's name, headline label and formula, and the complete list of Assumption keys. Wait for the user's approval before you send it.

## Confirm a card

Confirm a card only when the user asks to confirm it. Show the card's title, status, `cardRevisionNumber`, and each Assumption key with its `currentRevisionNumber`, and state that confirming records those Assumption revisions. Wait for the user's approval before you send one `strata_confirm_assumption_card`.

When the user asks to confirm several cards at once, list the cards with `strata_list_assumption_cards` and show one preview of the cards the user named. For each named card, show the same fields as for a single card: title, status, `cardRevisionNumber`, and each Assumption key with its `currentRevisionNumber`. State that confirming records those Assumption revisions. After the user's one approval, send one `strata_confirm_assumption_card` per named card, with its `assumptionCardId`, its `cardRevisionNumber` from that list as `expectedCardRevisionNumber`, and its own fresh `idempotencyKey`. Then list the cards again and report each named card's status.

A press of "Confirm card" or "Confirm all cards" in the workspace App is the user's approval, and the App confirms those cards itself at their listed revisions, one at a time, stopping at the first failure. The App's model context names the case key and the `assumptionCardIds` it confirmed, and keeps that action until the App's next action; report it once. The cards it names are confirmed: report them and send no `strata_confirm_assumption_card` for them. When the user says they pressed one of these buttons and no confirmation reached you in model context, or the press covered more cards than the context names, call `strata_list_assumption_cards` and report each card's status.

## Retry

Use a fresh `idempotencyKey` for each write the user approves. Retry an ambiguous result only with the same `idempotencyKey` and unchanged input. On `STALE_STATE`, list the cards again, show the changed preview, wait for the user's approval, and use a new key.
