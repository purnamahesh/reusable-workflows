# Reusable Workflows

Reusable GitHub Actions workflows for Rust projects.

## CI Workflows

### rust-check.yaml

Runs `cargo fmt --check` and `cargo clippy` per feature.

| Input | Type | Default | Description |
|---|---|---|---|
| `toolchain` | string | `stable` | Rust toolchain |
| `features` | string | `'["default"]'` | JSON list of feature sets |
| `no-default-features` | boolean | `false` | Pass `--no-default-features` for specific features |

### rust-test.yaml

Runs `cargo test` and `cargo test --doc` across OS/toolchain/features matrix.

| Input | Type | Default | Description |
|---|---|---|---|
| `os` | string | `'["ubuntu-latest"]'` | JSON list of runner OS images |
| `toolchain` | string | `'["stable"]'` | JSON list of Rust toolchains |
| `features` | string | `'["default"]'` | JSON list of feature sets |
| `no-default-features` | boolean | `false` | Pass `--no-default-features` for specific features |
| `test-path` | string | `''` | Package passed to `-p` flag |

### rust-audit.yaml

Runs `cargo audit` and `cargo deny check`. No inputs.

## Release Workflows

### bump.yaml

Bumps version and changelog via [Knope](https://knope.tech) using conventional commits. Commits to current branch. Trigger via `/bump` slash command on a PR.

| Input | Type | Default | Description |
|---|---|---|---|
| `knope-version` | string | `0.22.3` | Knope version |

**Secrets:** `PAT` (contents:write)

### rust-pre-release.yaml

Bumps pre-release version, publishes to crates.io, commits on success, creates GitHub pre-release. Trigger via `/pre-release` on a PR.

| Input | Type | Default | Description |
|---|---|---|---|
| `pre-release-label` | string | `alpha` | Label: `alpha`, `beta`, `rc` |
| `knope-version` | string | `0.22.3` | Knope version |

**Secrets:** `PAT` (contents:write), `CARGO_REGISTRY_TOKEN`

### rust-release.yaml

Publishes crate and creates GitHub release via Knope. Trigger on merge to main.

| Input | Type | Default | Description |
|---|---|---|---|
| `knope-version` | string | `0.22.3` | Knope version |
| `publish-crate` | boolean | `false` | Publish to crates.io |

**Secrets:** `PAT` (contents:write), `CARGO_REGISTRY_TOKEN` (if publishing)

### github-release.yaml

Standalone GitHub release with optional artifact attachments. For repos not using Knope.

| Input | Type | Default | Description |
|---|---|---|---|
| `version` | string | *(required)* | Version without `v` prefix |
| `artifact` | string | `''` | Artifact name to attach |

## Utility Workflows

### command-dispatch.yaml

Handles `/command key=value` slash commands in PR comments. Checks permissions, dispatches `.github/workflows/{command}.yaml` on the PR branch.

## Features Flag Reference

| `features` value | `no-default-features` | Cargo flags |
|---|---|---|
| `"default"` | either | *(none)* |
| `"all"` | either | `--all-features` |
| `"lite"` | `false` | `--features lite` |
| `"lite"` | `true` | `--no-default-features --features lite` |

## OS Setup Scripts

Place scripts in your caller repo to install system deps per OS:

- `.github/scripts/setup-ubuntu-latest.sh`
- `.github/scripts/setup-macos-latest.sh`
- `.github/scripts/setup-windows-latest.ps1`

Runs after checkout, before toolchain install. Skipped if not present. Use `$GITHUB_ENV` to export env vars.

## Example: Full Rust CI + Release

**knope.toml** (in your repo):

```toml
[package]
versioned_files = ["Cargo.toml"]
changelog = "CHANGELOG.md"

[github]
owner = "your-org"
repo = "your-repo"

[[workflows]]
name = "prepare-release"

[[workflows.steps]]
type = "PrepareRelease"

[[workflows]]
name = "release"

[[workflows.steps]]
type = "Release"
```

**Caller workflows:**

```yaml
# .github/workflows/check.yaml
name: Check
on: [pull_request]
jobs:
  check:
    uses: purnamahesh/reusable-workflows/.github/workflows/rust-check.yaml@feature/rust-ci
    with:
      features: '["full", "lite"]'
      no-default-features: true
```

```yaml
# .github/workflows/test.yaml
name: Test
on: [pull_request]
jobs:
  test:
    uses: purnamahesh/reusable-workflows/.github/workflows/rust-test.yaml@feature/rust-ci
    with:
      os: '["ubuntu-latest", "macos-latest"]'
      features: '["full", "lite"]'
      no-default-features: true
```

```yaml
# .github/workflows/command-dispatch.yaml
name: Command Dispatch
on:
  issue_comment:
    types: [created]
jobs:
  dispatch:
    uses: purnamahesh/reusable-workflows/.github/workflows/command-dispatch.yaml@feature/rust-ci
    secrets: inherit
```

```yaml
# .github/workflows/bump.yaml -- triggered by /bump
name: Bump
on:
  workflow_dispatch:
jobs:
  bump:
    uses: purnamahesh/reusable-workflows/.github/workflows/bump.yaml@feature/rust-ci
    secrets:
      PAT: ${{ secrets.PAT }}
```

```yaml
# .github/workflows/pre-release.yaml -- triggered by /pre-release
name: Pre-Release
on:
  workflow_dispatch:
    inputs:
      pre-release-label:
        default: 'alpha'
jobs:
  pre-release:
    uses: purnamahesh/reusable-workflows/.github/workflows/rust-pre-release.yaml@feature/rust-ci
    with:
      pre-release-label: ${{ inputs.pre-release-label }}
    secrets:
      PAT: ${{ secrets.PAT }}
      CARGO_REGISTRY_TOKEN: ${{ secrets.CARGO_REGISTRY_TOKEN }}
```

```yaml
# .github/workflows/release.yaml -- triggers on merge to main
name: Release
on:
  push:
    branches: [main]
jobs:
  release:
    uses: purnamahesh/reusable-workflows/.github/workflows/rust-release.yaml@feature/rust-ci
    with:
      publish-crate: true
    secrets:
      PAT: ${{ secrets.PAT }}
      CARGO_REGISTRY_TOKEN: ${{ secrets.CARGO_REGISTRY_TOKEN }}
```
