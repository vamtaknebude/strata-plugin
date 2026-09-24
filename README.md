# Strata Agent Plugin

> **Status: Staging preview.** Version `0.5.0-staging.2` connects to the hosted
> Strata MCP server through WorkOS Staging. Strata authorizes this release only
> for operators whom a Strata maintainer has approved. Strata does not authorize
> customer workspace access or customer production use.

Strata helps teams create, develop, and understand evidence-aware business
cases. The plugin installs three shared workflow skills and a connection
definition that contains no credentials. A Strata maintainer must provision and
authorize each operator in WorkOS Staging. Strata offers no self-service staging
enrollment.

Strata has completed end-to-end operator acceptance with Codex. The package passes
Claude Code installation validation, but Strata has not approved the complete
Claude Code OAuth and MCP workflow. Hermes Agent remains a compatibility preview.
This repository contains no ChatGPT submission package.

## What the plugin installs

The package contains three skill directories, portable metadata, native Claude
Code and Codex metadata, and the hosted MCP URL. Every package format uses the
same skill files.

The hosted Strata server controls business-case data, authorization, tools,
immutable history, and MCP Apps. The package contains no API key, bearer token,
cookie, password, or authorization code. Each client completes browser-based
Strata OAuth and stores its own grant. Never paste a credential into an assistant
conversation. Installing the plugin authorizes nothing: connecting the client to
the hosted server is a separate required step in every client.

## Claude Code compatibility validation

Only an authorized Strata operator should run this procedure. It validates the
package installation path, not an approved end-to-end Claude Code workflow.

Add the marketplace and install the plugin:

```bash
claude plugin marketplace add vamtaknebude/strata-plugin
claude plugin install strata@strata
```

Start Claude Code, run `/reload-plugins`, and confirm that Claude Code lists these
skills under the `strata` namespace:

- `/strata:start-business-case`
- `/strata:develop-business-case`
- `/strata:understand-business-case`

In the Claude Code CLI, run `/mcp`, select the bundled `strata` server, and
complete browser OAuth. Do not paste the callback code or any token into chat.
`claude mcp list` names that server `plugin:strata:strata` and reports
`Needs authentication` until the grant exists. The Claude desktop app has no
`/mcp` panel; use [Claude desktop app
connection](#claude-desktop-app-connection) instead. Verify a read before using
writes:

```text
Use /strata:understand-business-case to show my Strata connection context.
```

Update the marketplace and plugin:

```bash
claude plugin marketplace update strata
claude plugin update strata@strata
```

Remove the plugin and marketplace:

```bash
claude plugin uninstall strata@strata
claude plugin marketplace remove strata
```

Use **Clear authentication** for `strata` in `/mcp` if you also want to remove
the OAuth grant.

## Claude desktop app connection

The Claude desktop app manages MCP servers through Connectors, and a
plugin-bundled server is not a connector, so installing the plugin never offers a
sign-in for it. Install the plugin for its skills, then add the same endpoint as
a custom connector.

On Pro and Max, open Customize, then Connectors, then the `+` control, then **Add
custom connector**, and enter this MCP URL:

```text
https://strata-utrtwerwt.sprava.ai/api/mcp
```

Leave the Advanced OAuth client ID and secret empty. The Strata authorization
server advertises a `registration_endpoint`, so dynamic client registration
supplies them. Complete browser OAuth, and do not paste the callback code or any
token into chat.

On Team and Enterprise, only an Owner adds the connector, in Organization
settings, Connectors. Each member then authenticates individually from Customize,
Connectors by selecting **Connect**.

Verify the connection with the same read shown above before using writes. Until
the connection is authorized every Strata tool is absent, and each skill stops
and names this step instead of failing at its first tool call.

Remove the connector where it was added — Customize, Connectors, or
Organization settings, Connectors on Team and Enterprise — when you remove the
plugin.

## Codex installation

Only an authorized Strata operator should run this procedure.

Add the marketplace and install the plugin:

```bash
codex plugin marketplace add vamtaknebude/strata-plugin --ref main
codex plugin list --marketplace strata --available --json
codex plugin add strata@strata
```

Start a new Codex task and confirm that Codex lists these skills:

- `$strata:start-business-case`
- `$strata:develop-business-case`
- `$strata:understand-business-case`

Complete browser OAuth and verify the remote endpoint:

```bash
codex mcp get strata --json
codex mcp login strata
```

The endpoint must equal `https://strata-utrtwerwt.sprava.ai/api/mcp`. Start a new
task and ask `$strata:understand-business-case` to show the Strata connection
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

Hermes Agent can install the root portable Agent Plugins v1 package. Install the
package without enabling it, inspect the installed name, and enable that name:

```bash
hermes plugins install vamtaknebude/strata-plugin --no-enable
hermes plugins list
hermes plugins enable strata
```

Hermes translates the `streamable-http` entry in `mcp.json` into its remote MCP
runtime. Agent Plugins v1 cannot declare Strata's OAuth policy. Add the endpoint
through Hermes' native OAuth MCP configuration:

```bash
hermes mcp add strata --url https://strata-utrtwerwt.sprava.ai/api/mcp --auth oauth
hermes mcp login strata
hermes mcp test strata
```

Complete OAuth in the browser. Use `skills_list` and `skill_view` in a new Hermes
session to confirm the three installed skills. Ask for the Strata connection
context before using writes. Do not use write guidance until the read succeeds
and the installed Hermes version exposes the expected write tools.

Update or remove the preview installation:

```bash
hermes plugins update strata
hermes plugins disable strata
hermes plugins remove strata
hermes mcp remove strata
```

## Versions and support

The staging release uses a `-staging.<number>` semantic-version suffix and a
matching GitHub prerelease. Immutable `v<version>` tags identify published plugin
versions.

Report installation and behavior defects in the repository [issue
tracker](https://github.com/vamtaknebude/strata-plugin/issues). Report suspected
vulnerabilities through the private process in [SECURITY.md](./SECURITY.md).

[RELEASING.md](./RELEASING.md) defines the release process and source boundary.
[DISTRIBUTION.md](./DISTRIBUTION.md) defines the permitted use of this package.
