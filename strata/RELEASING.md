# Release the Strata Agent Plugin

The private Strata monorepo owns the plugin content. The release version is an
input to the release command. It is written into the generated distribution,
marketplace catalog, release metadata, and immutable Git tag. It is never a
source-code change.

Source manifests use the fixed development version `0.0.0-staging.0`. Client
manifests are generated from the canonical portable manifest. Tests use fixed
fixtures. Do not update those values or current-state documentation for a release.
GitHub releases and `release-metadata.json` identify published versions.

## Publish

Use a clean, committed checkout on `main` with locked dependencies and Gitleaks
8.30.1 installed. Set `PLUGIN_VERSION` to the intended `-staging.<number>` version,
then run:

```bash
bun run agent-plugin:release -- --version "${PLUGIN_VERSION:?Set PLUGIN_VERSION}" --publish
```

A version-only release needs no source commit, pull request, test suite, native
client installation test, or code review. Changes to plugin content and release
code follow the normal development checks before they merge.

The command exports the committed allowlist, substitutes the requested version
only in manifest version fields, validates package structure and metadata, scans
the package and prospective distribution history, verifies OAuth discovery,
pushes the distribution branch and immutable tag atomically, and creates the
matching GitHub prerelease. It preserves existing repository security settings.
Administrative controls are configured only when creating a distribution
repository. Publication never changes repository visibility.

## Inspect without publishing

A dry run writes no output. An inspection path must not already exist:

```bash
bun run agent-plugin:release -- --version "${PLUGIN_VERSION:?Set PLUGIN_VERSION}" --output ../strata-plugin-check --dry-run
bun run agent-plugin:release -- --version "${PLUGIN_VERSION:?Set PLUGIN_VERSION}" --output ../strata-plugin-inspection
```

The inspection package contains `strata/`, `uprali-demo/`,
`uprali-demo-chatgpt/`, both root catalogs, and `release-metadata.json`. The
metadata identifies the exact source commit and package digest. It contains no
credentials.
