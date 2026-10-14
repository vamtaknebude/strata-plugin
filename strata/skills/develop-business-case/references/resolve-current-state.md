# Resolve exact current state

If the host has not established the active organization, call `strata_connection_context` first. Never infer organization authority from client, model, plugin, skill, or environment metadata.

For case-bound work, resolve a title with `strata_list_cases` and wait before a dependent read. If several cases match, show bounded choices and wait. Read the current case with `strata_read_case`. Retain its exact case, cycle, case revision ID and number, and subject revision IDs. Use overview when alternatives matter. Read assumptions before revising or removing an existing assumption, but not for a new assumption or a driver receipted in this conversation. Read evidence lineage only for a selected existing assumption or evidence item. Never use a historic head as current.

Use `strata_read_case` for every case-content read in this workflow. Do not use `strata_get_case`. Select one projection:

- overview: `projection.kind` is `overview` with no other projection field;
- assumptions: `projection.kind` is `assumptions`, `limit` is `25`, and `driversOnly` is `false`;
- scenario: `projection.kind` is `scenario`, `scenarioId` is the selected visible stable UUID, and `limit` is `25`;
- assumption lineage: `projection.kind` is `evidence_lineage`, `assumptionId` is the selected visible assumption UUID, and `limit` is `25`; or
- evidence lineage: `projection.kind` is `evidence_lineage`, `evidenceItemId` is the selected visible evidence item's UUID, and `limit` is `25`.

Construct dependent reads only from completed Uprali output. Copy every UUID byte-for-byte into its matching ID field. Never put a case key, revision number, label, or another subject's UUID in an ID field. For a visible protected record ID, perform the matching direct protected read before resolving a case when the requested write does not require case identifiers.

The user does not supply organization, tenant, hidden record, or author identifiers. The server records the authenticated actor. `strata_save_assumption` assigns that user as owner and accepts no `ownerUserId`. Disclose this and stop if the user requests another owner.
