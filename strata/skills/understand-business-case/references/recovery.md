# Recovery

- On an ambiguous title, stop after bounded choices and wait for selection.
- On `INPUT_INVALID` or `MALFORMED_CURSOR`, correct the input from the tool schema or restart the page from the same exact revision; do not substitute another revision.
- On `NOT_FOUND`, explain which exact case or revision was unavailable and ask for a valid selection.
- On `AUTHORIZATION_UNAVAILABLE` or masked data, do not search for or infer the restricted content.
- On `PERSISTENCE_UNAVAILABLE`, `CAPABILITY_TIMEOUT`, or another retryable read failure, retry the unchanged read at most once when useful; otherwise report that Strata could not complete the read.
- Keep a normal successful journey within four Strata tool calls. Pagination may exceed that only when the user explicitly requests more returned records.
