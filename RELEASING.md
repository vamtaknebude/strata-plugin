# Release the Strata Agent Plugin

The private Strata monorepo is the canonical authored source. This public
distribution repository is generated and must not be edited independently.

From a clean, committed Strata checkout, install locked dependencies and run the
complete no-output preflight:

```bash
bun install --frozen-lockfile
bun run agent-plugin:release -- --output ../strata-plugin --dry-run
```

Prepare or update the local distribution repository only after the dry run passes:

```bash
bun run agent-plugin:release -- --output ../strata-plugin
```

The command validates the canonical portable package, regenerated client
manifests, both marketplace catalogs, exact allowlist, symlink containment,
credential scan, source equality, pinned Codex CLI, and pinned Claude Code CLI. It
then creates one release commit and immutable `v<version>` tag in the independent
local repository. Repeating the same source commit and version is a no-op; reusing a
version for different content fails.

Inspect the complete generated tree, Git diff, release metadata, and local tag. The
publication step is an external mutation and requires explicit maintainer approval:

```bash
bun run agent-plugin:release -- --output ../strata-plugin --publish
```

Publishing creates or verifies the public `vamtaknebude/strata-plugin` repository,
enables private vulnerability reporting, pushes `main` and the version tag, and
creates the matching GitHub release only after local validation succeeds. GitHub
credentials stay in the release environment and are never copied into the package,
logs, metadata, or repository configuration.

After publication, repeat the clean install and read-only verification journeys in
[README.md](./README.md) from environments that have no access to the private Strata
repository. Interactive OAuth checks are release evidence, not pull-request gates.
