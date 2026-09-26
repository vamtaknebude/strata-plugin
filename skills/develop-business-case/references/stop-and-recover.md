# Stop and recover

After the first failed command, make no second mutation call in that turn. Preserve receipts, report authorized error information, and follow the reference's recovery rules. A `retry: safe` disposition does not permit another definitive application call.

- Ambiguous transport permits one byte-identical same-key retry. Accept `replayed: true` as the original result.
- On `STALE_STATE` or a revision conflict, read fresh state, compare it with the proposal, show a changed preview, reconfirm, and use a new key.
- On `COMMAND_REUSED`, read fresh state, reconstruct the command, preview, reconfirm, and use a new key.
- On dependency, compatibility, or input errors, use only authorized facts and user corrections. Never cascade or convert.
- On `RATE_LIMITED` or `RATE_LIMIT_UNAVAILABLE`, access failures, protected missing-or-inaccessible responses, or terminal provider failure, stop without unchanged retry, alternate-ID probing, or hidden inference.

Cancellation stops before key generation or mutation. Report only what the user supplied and Strata durably accepted.
