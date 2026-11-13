# Export routing

When the user asks for a Run's model as a file, call `strata_export_calculation_model` once with exactly one of the exact `runId` or the case's `investmentCaseId`. The export returns the model as an XLSX workbook.

When the result has `status: "no_successful_run"`, state that the case has no successful Run. When the result names a latest created Run, name its `runId` and status, and stop without creating a Run.

When the result returns a resource link, give it to the user as a link named with the returned `fileName` and tell the user to open the link to download the workbook. When the host gives no way to open the link, attach the workbook to the chat as a file named with the returned `fileName` and tell the user to download the attached file. When the host gives no way to create a file, say that the file cannot be attached in chat. Do not write the file, recompute it, or summarize values beyond what the export returned.
