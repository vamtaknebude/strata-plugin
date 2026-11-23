# Recovery

- `INPUT_INVALID`: correct only from user facts, refresh the creation panel via `strata_read_case_intake` in MCP App hosts or show the complete plain-language preview and reconfirm in chat when no panel is visible, and use a new UUID behind the scenes.
- `PERSISTENCE_UNAVAILABLE`, `CAPABILITY_TIMEOUT`, lost response, or ambiguous transport: retry once with byte-for-byte identical normalized input and the same UUID. Accept `replayed: true` as the existing receipt.
- `COMMAND_REUSED`: stop, reconstruct the intended brief, make a new routing call if the description changed, refresh the creation panel via `strata_read_case_intake` in MCP App hosts or show the complete plain-language preview and reconfirm in chat when no panel is visible, and use a new UUID behind the scenes.
- Routing failure (`CAPABILITY_DEPENDENCY_FAILED`): the routing call is retryable with the same UUID and identical input. A second failure ends intake with no case.
- Authentication or access failure: do not retry unchanged or disclose another organization's data.
- Stale state during a later edit: read the current exact revision, show the changed plain-language edit preview, reconfirm, and use a new UUID behind the scenes. Do not create another case.
- Cancellation or hesitation before a success receipt: stop with no further tool call and respond in plain business language without technical routing details.

Never expose idempotency keys or internal retry rules to the member. Never claim the brief is objectively true or convert prose quality into authorization. The server independently enforces deterministic validation and tenancy even when this skill is absent.
