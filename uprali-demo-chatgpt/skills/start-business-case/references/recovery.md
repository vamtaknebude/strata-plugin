# Recovery

- `INPUT_INVALID`: correct only from user facts, show the complete preview, reconfirm, and use a new UUID.
- `PERSISTENCE_UNAVAILABLE`, `CAPABILITY_TIMEOUT`, lost response, or ambiguous transport: retry once with byte-for-byte identical normalized input and the same UUID. Accept `replayed: true` as the existing receipt.
- `COMMAND_REUSED`: stop, reconstruct the intended brief, make a new routing call if the description changed, show a complete preview, reconfirm, and use a new UUID.
- Routing failure (`CAPABILITY_DEPENDENCY_FAILED`): the routing call is retryable with the same UUID and identical input. A second failure ends intake with no case.
- Authentication or access failure: do not retry unchanged or disclose another organization's data.
- Stale state during a later edit: read the current exact revision, show the changed edit preview, reconfirm, and use a new UUID. Do not create another case.
- Cancellation before a success receipt: stop with no further tool call.

Never claim the brief is objectively true or convert prose quality into authorization. The server independently enforces deterministic validation and tenancy even when this skill is absent.
