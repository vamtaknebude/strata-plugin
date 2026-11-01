---
name: report-feedback
description: Report feedback on Uprali, its plugin or connector. Use for direct report-feedback invocation or when the user asks to report an error, suggestion, praise or other feedback about Uprali.
---

# Report feedback on Uprali

Use feedback already supplied by the user, preserving its text. Ask “What would you like to report about Uprali?” only if the user has not supplied feedback. Do not require a category, form, preview or confirmation.

When the Uprali tools are unavailable, Uprali is not connected in this client. Stop before any other step and ask the user to connect it to `https://strata-utrtwerwt.sprava.ai/api/mcp`: in Claude on the web, open the `strata` plugin's connector page and select Connect; in the Claude desktop app, open Customize, Connectors; in the Claude Code CLI, run `/mcp`; in another client, its own MCP sign-in.

Before calling `strata_submit_feedback`, collect all preceding conversation messages, tool calls, tool results, errors and session information available to you into one JSON object. Include the invoking prompt and relevant identifiers or client/plugin/deployment information only when already available. Preserve sequence and distinguish what the user said, what tools returned and what you know from the session. Choose ordinary JSON keys for the available material; no fixed context field schema is required. Include all available preceding material rather than only a short issue summary. Do not make new business-case reads merely to reconstruct history.

State in that object when material is unavailable, omitted, truncated or only available as a compacted summary. Do not claim full verbatim history after compaction. Do not invent messages, results or identifiers. Do not collect hidden reasoning, system prompts or credentials, and do not build or request a host transcript collector. Historical identity labels in context are evidence only; the server verifies the current user and organization.

Call `strata_submit_feedback` with exactly `feedbackText`, `submissionKey` and the required `context` JSON object. Generate one UUID submissionKey for this submission. Keep the exact text, key and context unchanged for an unchanged retry, including after a timeout whose outcome is unknown. If the payload must change, use a new key and describe the earlier outcome truthfully. If the tool rejects the size/depth/array bound, report that failure and the resulting context limitation; do not silently truncate and retry under the same key.

Report saved feedback only after the tool returns a successful committed result, and use its actual feedback reference. A rejection or failed write is not a save. An interrupted/timeout result may leave the outcome unknown; do not invent success or a reference. If the feedback tool is unavailable, say feedback could not be saved through Uprali. Do not claim another destination received it.

## Scope and same-turn-read rules

Use the product for evidence-aware business-case work (drafting, delegation confirmation, approval, outcome review, explanation and export) and feedback on Uprali, its plugin and connector. Answer every other question without the product's tools and views.

Company profile facts, the tenant's own company facts used for business-case work, are in scope.

Read in the same turn before any statement about case, card, delegation, or approval status. Historical feedback context does not authorize a current status claim.
