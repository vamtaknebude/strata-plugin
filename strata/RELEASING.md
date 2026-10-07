# Release the Strata Agent Plugin

The private Strata monorepo is the canonical source. The release command generates
the separate `vamtaknebude/strata-plugin` distribution repository. Do not edit
generated distribution files directly.

The current release channel is a staging preview. Every package version must use a
`-staging.<number>` prerelease suffix while the hosted MCP resource uses the WorkOS
Staging issuer. GitHub must publish each matching release as a prerelease.

## Prepare and publish a release

Install locked dependencies and Gitleaks 8.30.1. Run the no-output preflight from
a clean, committed Strata checkout:

```bash
bun install --frozen-lockfile
gitleaks version
bun run agent-plugin:release -- --output ../strata-plugin-check --dry-run
```

Generate a package for inspection at a path that does not exist:

```bash
bun run agent-plugin:release -- --output ../strata-plugin-inspection
```

The command exports the exact allowlist from `<source-commit>:plugins/strata`. It
validates portable manifests, client manifests, marketplace catalogs, file paths,
symlinks, metadata, digest, source provenance, the pinned Codex CLI, and the pinned
Claude Code CLI. Gitleaks scans the generated directory with its default rules and
redacted output. The inspection directory contains a package tree, not a trusted
publication checkout. The release places the Strata package in `strata/`, the
Uprali package in `uprali-demo/`, the ChatGPT package in
`uprali-demo-chatgpt/`, and the Codex and Claude catalogs at the repository
root.

Inspect every generated file and `release-metadata.json`. Confirm that the source
commit belongs to `main`. Publish the release only after explicit maintainer
approval:

```bash
bun run agent-plugin:release -- --publish
```

Publish mode creates a fresh temporary clone of the distribution repository. It
disables inherited Git configuration, templates, replacement objects, and hooks.
It constructs the release commit and immutable `v<version>` tag, runs `gitleaks git
--log-opts=--all` over the prospective history, verifies OAuth metadata, configures
available GitHub secret-scanning controls and release ref rules, atomically pushes
`main` and the tag, and creates or verifies the matching GitHub prerelease. The
maintainer also confirms that protected-resource metadata names the documented
WorkOS Staging issuer.

Publish mode does not change repository visibility. GitHub credentials remain in
the release environment. The command never copies credentials into package files,
logs, metadata, or repository configuration.

Before public visibility, edit the existing `v0.4.0` GitHub release. Mark it as a
prerelease, change its title to `Superseded staging package v0.4.0`, and add a note
that directs users to `v0.5.0-staging.1`. Do not move or replace the immutable
`v0.4.0` tag.

## Perform the final public-visibility step

The maintainer performs this step separately after the staging release passes the
Claude Code and Codex installation checks. Review GitHub's [repository visibility
effects](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/managing-repository-settings/setting-repository-visibility)
before changing the repository from private to public.

After GitHub reports public visibility, enable private vulnerability reporting in
**Settings → Code security → Private vulnerability reporting**. GitHub does not
offer that control for a private repository. Confirm that the private report link
in [SECURITY.md](./SECURITY.md) opens for a signed-out user.

Confirm that GitHub still reports secret scanning and push protection as enabled.
Confirm that the `Strata release main` and `Strata release tags` rulesets remain
active. Restore any control that the visibility change disabled before announcing
the repository.

Use signed-out or unauthenticated GitHub access to repeat the clean marketplace
installation journeys in [README.md](./README.md). Confirm that the installed
version equals the GitHub prerelease and that OAuth reaches WorkOS Staging.
