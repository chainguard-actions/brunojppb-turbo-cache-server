<!-- markdownlint-disable -->

# Hardening Report: brunojppb--turbo-cache-server/4.0.15

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **brunojppb--turbo-cache-server/4.0.15** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple `run:` blocks in build-binaries.yml directly interpolate `${{ inputs.version }}` and `${{ inputs.cache-ref }}` expressions into shell commands. These are workflow_call inputs controlled by the calling workflow and are injected without quoting into `sed`, `docker buildx build`, `docker pull`, and `docker create` commands, enabling command injection. Offending lines include: `sed -i "...version = \"${{ inputs.version }}\"/" Cargo.toml`, `docker buildx build ... :${{ inputs.version }} ... :${{ inputs.cache-ref }}`, and `docker pull/create ... :${{ inputs.version }}`.

Locations:

- `.github/workflows/build-binaries.yml:28`
- `.github/workflows/build-binaries.yml:40`
- `.github/workflows/build-binaries.yml:47`
- `.github/workflows/build-binaries.yml:72`
- `.github/workflows/build-binaries.yml:79`

### script-injection (severity: high)

Sub-rule (a): Multiple `run:` blocks in release.yml directly interpolate `${{ github.event.inputs.semver }}` (a workflow_dispatch user-supplied input) and `${{ github.repository }}` into shell commands without quoting. Offending lines include: `sed -i "...version = \"${{ github.event.inputs.semver }}\"/" Cargo.toml`, and `echo "...https://github.com/${{ github.repository }}/compare/..." >> "$NOTES_FILE"`. These allow an attacker with dispatch access to inject arbitrary shell commands.

Locations:

- `.github/workflows/release.yml:38`
- `.github/workflows/release.yml:97`
- `.github/workflows/release.yml:115`

### unpinned-uses (severity: high)

All `uses:` references across all workflow files are pinned to mutable version tags rather than immutable 40-character SHA commit hashes. This exposes the workflows to supply-chain attacks if any referenced action is compromised or its tag is moved. Unpinned references include: `actions/checkout@v7`, `docker/setup-buildx-action@v4`, `docker/login-action@v4`, `actions/upload-artifact@v7`, `actions/download-artifact@v8`, `actions-rust-lang/setup-rust-toolchain@v1`, `actions-rust-lang/rustfmt@v1`, `docker/setup-buildx-action@v4.2.0`.

Locations:

- `.github/workflows/build-binaries.yml:24`
- `.github/workflows/build-binaries.yml:27`
- `.github/workflows/build-binaries.yml:34`
- `.github/workflows/build-binaries.yml:62`
- `.github/workflows/build-binaries.yml:65`
- `.github/workflows/build-binaries.yml:76`
- `.github/workflows/build-binaries.yml:87`
- `.github/workflows/build.yml:14`
- `.github/workflows/build.yml:21`
- `.github/workflows/ci.yml:13`
- `.github/workflows/ci.yml:15`
- `.github/workflows/ci.yml:16`
- `.github/workflows/ci.yml:24`
- `.github/workflows/ci.yml:25`
- `.github/workflows/release.yml:33`
- `.github/workflows/release.yml:43`
- `.github/workflows/release.yml:55`
- `.github/workflows/release.yml:58`
- `.github/workflows/release.yml:65`

### broad-permissions (severity: medium)

release.yml sets `permissions: "write-all"` at the top level, granting all jobs overly broad write access to every GitHub API scope. This should be replaced with the minimal specific permissions required (e.g., `contents: write`, `packages: write`).

Locations:

- `.github/workflows/release.yml:12`

### missing-permissions (severity: medium)

build-binaries.yml, build.yml, and ci.yml have no top-level `permissions:` key and no job-level `permissions:` keys on any job. Without explicit permissions, these workflows inherit the repository's default token permissions, which may be overly broad. Explicit minimal permissions should be declared.

Locations:

- `.github/workflows/build-binaries.yml:1`
- `.github/workflows/build.yml:1`
- `.github/workflows/ci.yml:1`

### hardcoded-credentials (severity: high)

ci.yml contains a hardcoded literal value assigned to `S3_SECRET_KEY: some-secret-key`. The key name contains "secret" and the value is a non-expression literal string. Even though this appears to be a placeholder for testing, hardcoded credential-named values should use GitHub Actions secrets expressions (e.g., `${{ secrets.S3_SECRET_KEY }}`) rather than literal strings.

Locations:

- `.github/workflows/ci.yml:30`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, broad-permissions, missing-permissions, hardcoded-credentials

**Notes:**

Fixed all 6 findings across 4 workflow files:

1. **script-injection** (build-binaries.yml): Moved `${{ inputs.version }}` and `${{ inputs.cache-ref }}` out of `run:` shell bodies into `env:` blocks as `INPUT_VERSION` and `INPUT_CACHE_REF`. Shell scripts now reference `${INPUT_VERSION}` and `${INPUT_CACHE_REF}` safely.

2. **script-injection** (release.yml): Moved `${{ github.event.inputs.semver }}` into `env:` blocks as `INPUT_SEMVER`/`CACHE_SERVER_VERSION`, and `${{ github.repository }}` into `GITHUB_REPOSITORY` env var. All `run:` blocks now use shell variable references only.

3. **unpinned-uses**: Pinned all action references to full 40-char SHA hashes:
   - `actions/checkout@v7` → `@3d3c42e5aac5ba805825da76410c181273ba90b1`
   - `docker/setup-buildx-action@v4` → `@bb05f3f5519dd87d3ba754cc423b652a5edd6d2c`
   - `docker/login-action@v4` → `@dbcb813823bdd20940b903addbd779551569679f`
   - `actions/upload-artifact@v7` → `@043fb46d1a93c77aae656e7c1c64a875d1fc6a0a`
   - `actions/download-artifact@v8` → `@3e5f45b2cfb9172054b4087a40e8e0b5a5461e7c`
   - `actions-rust-lang/setup-rust-toolchain@v1` → `@166cdcfd11aee3cb47222f9ddb555ce30ddb9659`
   - `actions-rust-lang/rustfmt@v1` → `@4066006ec54a31931b9b1fddfd38f2fdf2d27143`

4. **broad-permissions** (release.yml): Replaced `permissions: "write-all"` with `permissions: contents: write` and `packages: write`.

5. **missing-permissions** (build-binaries.yml, build.yml, ci.yml): Added `permissions: contents: read` to each workflow.

6. **hardcoded-credentials** (ci.yml): Replaced literal `S3_SECRET_KEY: some-secret-key` with `S3_SECRET_KEY: ${{ secrets.S3_SECRET_KEY }}`.

