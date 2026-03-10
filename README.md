# Reusable Workflows

Reusable GitHub Actions workflows for Rust projects.

## Workflows

### rust-test.yaml

Runs `cargo test` and `cargo test --doc` across an OS/toolchain/features matrix.

**Inputs:**

| Input | Type | Default | Description |
|---|---|---|---|
| `os` | string | `'["ubuntu-latest"]'` | JSON list of runner OS images |
| `toolchain` | string | `'["stable"]'` | JSON list of Rust toolchains |
| `features` | string | `'["default"]'` | JSON list of feature sets (see [Features Flag Reference](#features-flag-reference)) |
| `no-default-features` | boolean | `false` | When `true`, passes `--no-default-features` for specific feature names (not applied to `"default"` or `"all"`). Use this for crates with mutually exclusive features. |
| `test-path` | string | `''` | Package or path passed to `-p` flag. Empty runs all. |

### rust-check.yaml

Runs `cargo fmt --check` and `cargo clippy` (clippy runs per feature in a matrix).

**Inputs:**

| Input | Type | Default | Description |
|---|---|---|---|
| `toolchain` | string | `stable` | Rust toolchain |
| `features` | string | `'["default"]'` | JSON list of feature sets (same semantics as rust-test) |
| `no-default-features` | boolean | `false` | Same as rust-test |

### rust-audit.yaml

Runs `cargo audit` for security vulnerabilities. No inputs required.

### command-dispatch.yaml

Dispatches slash commands from PR comments.

## OS Setup Scripts (Convention-based)

Both `rust-test` and `rust-check` support automatic OS-level setup scripts. Place scripts in your repo at:

- `.github/scripts/setup-ubuntu-latest.sh` -- runs on Ubuntu
- `.github/scripts/setup-macos-latest.sh` -- runs on macOS
- `.github/scripts/setup-windows-latest.ps1` -- runs on Windows (PowerShell)

Scripts run after checkout but before toolchain installation. If a script doesn't exist for an OS, the step is silently skipped.

Use these to install system dependencies (e.g., `apt-get install libsqlite3-dev`, `brew install sqlite`).

Scripts can export environment variables for subsequent steps using `$GITHUB_ENV`.

## Features Flag Reference

| `features` value | `no-default-features` | Cargo flags |
|---|---|---|
| `"default"` | either | *(none -- crate defaults)* |
| `"all"` | either | `--all-features` |
| `"lite"` | `false` | `--features lite` |
| `"lite"` | `true` | `--no-default-features --features lite` |

## Usage Examples

### Basic (defaults only)

```yaml
jobs:
  test:
    uses: purnamahesh/reusable-workflows/.github/workflows/rust-test.yaml@feature/rust-ci
```

### Mutually exclusive features across multiple OSes

```yaml
jobs:
  test:
    uses: purnamahesh/reusable-workflows/.github/workflows/rust-test.yaml@feature/rust-ci
    with:
      os: '["ubuntu-latest", "macos-latest"]'
      features: '["full", "lite"]'
      no-default-features: true
```

### Check with specific features

```yaml
jobs:
  check:
    uses: purnamahesh/reusable-workflows/.github/workflows/rust-check.yaml@feature/rust-ci
    with:
      features: '["full", "lite"]'
      no-default-features: true
```
