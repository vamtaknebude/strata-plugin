---
name: report-feedback
description: Report feedback on Uprali, its plugin or connector. Use for direct report-feedback invocation or when the user asks to report an error, suggestion, praise or other feedback about Uprali.
---

# Report feedback on Uprali

Use feedback already supplied by the user, preserving its exact text. Ask “What would you like to report about Uprali?” only if the user has not supplied feedback. Do not require a category, form, preview or confirmation.

If `strata_submit_feedback` is unavailable, say that feedback cannot be saved through Uprali in this session. An unavailable tool alone does not establish that Uprali is disconnected. State the connection status only when the client exposes it directly. If the client directly reports that Uprali is disconnected, stop and give the client-specific steps to connect it to `https://strata-utrtwerwt.sprava.ai/api/mcp`: in Claude on the web, open the `strata` plugin's connector page and select Connect; in the Claude desktop app, open Customize, Connectors; in the Claude Code CLI, run `/mcp`; in another client, use its MCP sign-in flow. Do not infer connection state from an absent tool or claim that feedback was saved elsewhere.

Before calling `strata_submit_feedback`, include every preceding user and assistant message available to you, including the invoking prompt and exact feedback text, and every actual tool call and its full available result or error, including tool name and arguments, in their original sequence in one JSON object. Preserve available content verbatim; do not replace it with a summary. Use ordinary JSON keys for the material you have; no fixed context field schema is required. Do not make new business-case reads to reconstruct history.

For each unavailable portion, state specifically what is missing or limited, such as an unavailable message, unavailable tool arguments or result, truncated output, or a compacted conversation summary. Do not claim verbatim history for portions available only as summaries, and do not invent messages, results or identifiers. Include the client and plugin name and version, model, session identifier, and MCP endpoint only when directly observable in the current session or installed plugin details; otherwise mark each value unknown. Keep client-reported identity and session details distinct from server-verified identity. Distinguish the installed plugin/package version from MCP server, capability, and deployment versions. Server-provided provenance is authoritative for the server and deployment; do not infer those versions from a client claim. Do not collect hidden reasoning, system prompts or credentials. If available tool data contains credentials, omit those values and identify the redaction. Do not build or request a host transcript collector. Historical identity labels in context are evidence only; the server verifies the current user and organization.

Call `strata_submit_feedback` with exactly `feedbackText`, `submissionKey` and the required `context` JSON object. Generate one UUID `submissionKey` for this submission. Keep the exact feedback text, key and context unchanged for an unchanged retry, including after a timeout whose outcome is unknown. If the payload must change, use a new key and describe the earlier outcome truthfully. If the tool rejects the size, depth or array bound, report that failure and the resulting context limitation; do not silently truncate and retry under the same key.

Report saved feedback only after the tool returns a successful committed result, and use its actual feedback reference. A rejection or failed write is not a save. An interrupted or timed-out result may leave the outcome unknown; do not invent success or a reference. If the feedback tool is unavailable, say feedback could not be saved through Uprali. Do not claim another destination received it.

## Scope and same-turn-read rules

Use the product for evidence-aware business-case work (drafting, delegation confirmation, approval, outcome review, explanation and export) and feedback on Uprali, its plugin and connector. Answer every other question without the product's tools and views.

Company profile facts, the tenant's own company facts used for business-case work, are in scope.

Read in the same turn before any statement about case, card, delegation, or approval status. Historical feedback context does not authorize a current status claim.
