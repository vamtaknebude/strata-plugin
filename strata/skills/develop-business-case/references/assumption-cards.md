# Assumption Cards

Use this reference only when `strata_save_assumption_card`, `strata_confirm_assumption_card`, and `strata_list_assumption_cards` are all available. If any of the three tools is missing, do not offer Assumption Cards. If the user asks for them, say that this Uprali connection cannot record them, and continue with the rest of the request.

## Concepts

- An Assumption Card is a named group of Assumptions on one case, laid out by the case's pinned template. Its title, description, headline metric (label, plain-language formula, optional Model Item key), and ordered Assumption field keys come from the layout, not from user input.
- The responsible member is information only and may be unassigned. It does not restrict who may save or confirm the card.
- The card status is derived from the Assumptions on the card. `confirmed` means the card has at least one field, every field holds a current revision, and every one of those revisions counts as confirmed: a revision counts as confirmed when a card confirmation recorded it or when the adoption of a delegated Submission produced it. `modified` means a confirmation event exists and the card is not confirmed. Otherwise the card is `pending`; a never-confirmed card with some confirmed Assumptions shows `pending` with its confirmed and total counts.

## Read before each write

Call `strata_list_assumption_cards` with the exact `investmentCaseId` before each card write. Each card carries its `cardKey`, title, description, headline metric, responsible member (`userId`, nullable `displayName`, `email`) or none, status, confirmed and total counts, and one entry per layout field in field order: `fieldKey`, `fieldLabel`, `present`, `assumptionId`, `currentRevisionId`, `currentRevisionNumber`, `confirmed`, and `value`, with a not-yet-present entry for a field with no current revision.

## Save a card

Each save sets, changes, or clears only the responsible member. Send `investmentCaseId`, `cardKey`, `responsibleMemberUserId` (`null` clears), `expectedResponsibleMemberUserId` (the last-read member, `null` when the card had none), and a fresh `idempotencyKey`. Resolve a named member with `strata_list_organization_members`; the member must be an active member of the tenant. Show the card, the current member, and the new member; wait for approval before sending.

## Confirm a card

Confirm a card only when the user asks to confirm it. Show the card's title, status, and each entry's field key with its current revision number and confirmed flag, and state that confirming records those Assumption revisions. Wait for approval before sending one `strata_confirm_assumption_card` with `investmentCaseId`, `cardKey`, the complete `expectedAssumptionRevisions` pairs from that list (at least one pair), and its own fresh `idempotencyKey`.

When the user asks to confirm several cards at once, show one preview of the named cards. After the user's one approval, send one `strata_confirm_assumption_card` per named card. Then list the cards again and report each named card's status.

A press of "Confirm card" or "Confirm all cards" in the workspace App is the user's approval, and the App confirms those cards itself through the host one at a time at their listed revisions, stopping at the first failure. It refuses a card with no present inputs. The App's model context names the case key, the card titles, the `investmentCaseId`, and the `cardKeys` it confirmed, and keeps that last action with every later selection until the App's next action replaces it. The cards it names are confirmed: report them and send no `strata_confirm_assumption_card` for them.

## Adoption outcome

Adoption of a delegated Submission is a second path to `confirmed`, alongside direct `strata_confirm_assumption_card`: each adoption records the new Assumption Revision it produced, and the card shows `confirmed` only when the current revision of every Assumption on it counts as confirmed. A later revision that supersedes an adopted one keeps the card from `confirmed`.

## Scope and same-turn reads

Use the product for evidence-aware business-case work (drafting, delegation confirmation, approval, outcome review, explanation and export) and feedback on Uprali, its plugin and connector. Answer every other question without the product's tools and views. Company profile facts, the tenant's own company facts used for business-case work, are in scope. Read in the same turn before any statement about case, card, delegation, or approval status. Before stating any card's status, call `strata_list_assumption_cards` for the case in the same turn and report the status it returns, including after a press of an App button.

## Retry

Use a fresh `idempotencyKey` for each write the user approves. Retry an ambiguous result only with the same key and unchanged input. On `STALE_STATE`, list the cards again, show the changed preview, wait for the user's approval, and use a new key: `stale-revisions` for a moved Assumption revision set, `stale-responsible-member` for a moved member. On `CARD_INPUT_REJECTED`, report the named field keys and reasons (`missing`, `open_delegation`, `unevidenced`) and make no further write until the inputs are fixed.
