# Strata Agent Plugin

Strata helps teams create, develop, and understand evidence-aware business cases.
This public package installs three shared workflow skills and one credential-free
connection to the hosted Strata MCP server. A Strata account is required.

Claude Code and Codex are supported clients. Hermes Agent is a compatibility
preview: its portable adapter loads the shared skills and Streamable HTTP MCP
entry, while Strata's browser-OAuth policy is configured separately through
Hermes' native MCP command.

## What is installed

The local package contains the three `skills/` directories, portable metadata,
native Claude Code and Codex metadata, and the production MCP URL. The skills are
one shared set; there are no client-specific copies.

Business-case data, authorization, tools, immutable history, and MCP Apps remain on
the hosted Strata server. No API key, bearer token, cookie, password, or
authorization code is included in the package. Each client completes browser-based
Strata OAuth and stores its own grant. Never paste a credential into an assistant
conversation.

## Claude Code

Use Claude Code's native marketplace and plugin formats:

```bash
claude plugin marketplace add vamtaknebude/strata-plugin
claude plugin install strata@strata
```

Start Claude Code, run `/reload-plugins` (or start a new session), and confirm these
skills appear under the `strata` namespace:

- `/strata:start-business-case`
- `/strata:develop-business-case`
- `/strata:understand-business-case`

Run `/mcp`, select the bundled `strata` server, and complete the browser OAuth flow.
Do not paste the callback code or any token into chat. Verify a read before using
writes:

```text
Use /strata:understand-business-case to show my Strata connection context.
```

Update and reload:

```bash
claude plugin marketplace update strata
claude plugin update strata@strata
```

Remove the plugin and marketplace:

```bash
claude plugin uninstall strata@strata
claude plugin marketplace remove strata
```

Use **Clear authentication** for `strata` in `/mcp` when the OAuth grant should also
be removed.

## Codex

Add the public repository marketplace and install the plugin:

```bash
codex plugin marketplace add vamtaknebude/strata-plugin --ref main
codex plugin list --marketplace strata --available --json
codex plugin add strata@strata
```

Start a new Codex task and confirm all three skills are installed:

- `$strata:start-business-case`
- `$strata:develop-business-case`
- `$strata:understand-business-case`

Complete browser OAuth and verify the remote endpoint:

```bash
codex mcp get strata --json
codex mcp login strata
```

The endpoint must be `https://strata-utrtwerwt.sprava.ai/api/mcp`. Start another
new task and ask `$strata:understand-business-case` to show the Strata connection
context before using writes.

Update and reinstall the cached package:

```bash
codex plugin marketplace upgrade strata
codex plugin add strata@strata
```

Log out and remove the package:

```bash
codex mcp logout strata
codex plugin remove strata@strata
codex plugin marketplace remove strata
```

## Hermes Agent compatibility preview

Hermes Agent can install the root portable Agent Plugins v1 package. Install it
disabled, inspect it, then enable the exact plugin name shown by `list`:

```bash
hermes plugins install vamtaknebude/strata-plugin --no-enable
hermes plugins list
hermes plugins enable strata
```

Hermes translates the `streamable-http` entry in `mcp.json` into its remote MCP
runtime, but Agent Plugins v1 has no field that declares Strata's required OAuth
policy. Add the same endpoint through Hermes' native OAuth MCP configuration and
use that authenticated `strata` connection for verification:

```bash
hermes mcp add strata --url https://strata-utrtwerwt.sprava.ai/api/mcp --auth oauth
hermes mcp login strata
hermes mcp test strata
```

Complete OAuth in the browser. Use `skills_list` and `skill_view` in a new Hermes
session to confirm the three installed portable skills, then request the Strata
connection context as the required read check. Treat write guidance as unavailable
until that read succeeds and the installed Hermes version exposes the expected
write tools.

Update or remove the preview installation:

```bash
hermes plugins update strata
hermes plugins disable strata
hermes plugins remove strata
hermes mcp remove strata
```

## Versions and support

Immutable `v<version>` tags and matching GitHub releases identify published plugin
versions. Report ordinary defects in the public [issue
tracker](https://github.com/vamtaknebude/strata-plugin/issues). Report suspected
vulnerabilities privately as described in [SECURITY.md](./SECURITY.md).

The release process and source-of-truth boundary are documented in
[RELEASING.md](./RELEASING.md). The repository's explicit distribution notice is
in [DISTRIBUTION.md](./DISTRIBUTION.md).
