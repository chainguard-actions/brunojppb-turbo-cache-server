<!-- markdownlint-disable -->

# Hardening Report: brunojppb--turbo-cache-server/4.0.17

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **brunojppb--turbo-cache-server/4.0.17** was hardened automatically. 4 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All `uses:` references across all workflow files use mutable version tags instead of pinned 40-character SHA digests, making the workflows vulnerable to supply-chain attacks if the referenced action tags are moved or compromised. Failing references include: `actions/checkout@v7`, `docker/setup-buildx-action@v4`, `docker/login-action@v4`, `actions/upload-artifact@v7`, `actions-rust-lang/setup-rust-toolchain@v1`, `actions-rust-lang/rustfmt@v1`, `actions/download-artifact@v8`, `docker/setup-buildx-action@v4.3.0`.

Locations:

- `.github/workflows/build-binaries.yml:25`
- `.github/workflows/build-binaries.yml:28`
- `.github/workflows/build-binaries.yml:36`
- `.github/workflows/build-binaries.yml:57`
- `.github/workflows/build-binaries.yml:64`
- `.github/workflows/build-binaries.yml:67`
- `.github/workflows/build-binaries.yml:80`
- `.github/workflows/build.yml:13`
- `.github/workflows/build.yml:22`
- `.github/workflows/ci.yml:12`
- `.github/workflows/ci.yml:14`
- `.github/workflows/ci.yml:15`
- `.github/workflows/ci.yml:22`
- `.github/workflows/ci.yml:23`
- `.github/workflows/release.yml:27`
- `.github/workflows/release.yml:43`
- `.github/workflows/release.yml:46`
- `.github/workflows/release.yml:52`

### script-injection (severity: high)

Multiple `run:` blocks directly interpolate `${{ }}` expressions into shell commands (sub-rule a), allowing an attacker who controls the input values to inject arbitrary shell commands. Specific violations:
- `build-binaries.yml` "Update version on Cargo.toml" (both build-linux and build-macos jobs): `sed -i "s/^version = \".*\"/version = \"${{ inputs.version }}\"/" Cargo.toml` — `inputs.version` interpolated directly.
- `build-binaries.yml` "build binary" step: `--tag brunojppb/turbo-cache-server-build:${{ inputs.version }} ... --cache-to type=registry,ref=brunojppb/turbo-cache-server-build:${{ inputs.cache-ref }}` — both `inputs.version` and `inputs.cache-ref` interpolated directly.
- `build-binaries.yml` "create temp containers" step: `docker pull/create` commands with `${{ inputs.version }}` interpolated directly.
- `release.yml` "Update version on Cargo.toml": `sed -i "s/^version = \".*\"/version = \"${{ github.event.inputs.semver }}\"/" Cargo.toml` — `github.event.inputs.semver` interpolated directly.
- `release.yml` "Create GitHub Release": `${{ github.repository }}` interpolated directly inside `echo` commands writing to a notes file.

Locations:

- `.github/workflows/build-binaries.yml:31`
- `.github/workflows/build-binaries.yml:44`
- `.github/workflows/build-binaries.yml:48`
- `.github/workflows/build-binaries.yml:67`
- `.github/workflows/release.yml:35`
- `.github/workflows/release.yml:75`

### broad-permissions (severity: medium)

`release.yml` sets top-level `permissions: "write-all"`, granting every job in the workflow write access to all repository scopes (contents, packages, pull-requests, etc.). This violates the principle of least privilege and should be replaced with specific minimal permissions (e.g., `contents: write` for the release job only).

Locations:

- `.github/workflows/release.yml:13`

### missing-permissions (severity: medium)

The workflow files `build-binaries.yml`, `build.yml`, and `ci.yml` have no top-level `permissions:` key and no job-level `permissions:` blocks on any of their jobs. Without an explicit permissions declaration, workflows inherit the repository's default token permissions, which may be overly broad (write access to contents and other scopes). Each workflow should declare minimal required permissions.

Locations:

- `.github/workflows/build-binaries.yml:1`
- `.github/workflows/build.yml:1`
- `.github/workflows/ci.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, broad-permissions, missing-permissions

**Notes:**

Fixed all four findings across four workflow files:

1. unpinned-uses: Pinned all 8 action references to full 40-char SHA digests with tag comments in build-binaries.yml, build.yml, ci.yml, and release.yml.

2. script-injection: Moved all ${{ }} expressions out of run: blocks into env: blocks in build-binaries.yml (inputs.version, inputs.cache-ref) and release.yml (github.event.inputs.semver, github.repository). Shell scripts now reference plain env vars.

3. broad-permissions: Replaced `permissions: "write-all"` in release.yml with specific minimal permissions: `contents: write` (for git push, tagging, and GitHub release creation) and `packages: write` (for pushing to GHCR).

4. missing-permissions: Added `permissions: {}` to build-binaries.yml, build.yml, and ci.yml since these workflows only need the default read token (or no token at all for the workflow_call).

### Iteration 2

**Fixes applied:** hardcoded-credentials, script-injection

**Notes:**

Fixed two security issues: (1) In .github/workflows/ci.yml, replaced the hardcoded plaintext value `some-secret-key` for S3_SECRET_KEY with the GitHub Actions secret reference `${{ secrets.S3_SECRET_KEY }}`. (2) In .github/workflows/release.yml, added double quotes around the user-controlled `CACHE_SERVER_VERSION` variable in both the `git tag -a "${CACHE_SERVER_VERSION}"` command and the `git push origin "$CACHE_SERVER_VERSION"` command to prevent shell metacharacter injection from workflow_dispatch input.

### Iteration 3

**Fixes applied:** hardcoded-credentials

**Notes:**

Replaced the hardcoded literal value `S3_ACCESS_KEY: some-access-key` on line 31 of `.github/workflows/ci.yml` with a secrets reference `S3_ACCESS_KEY: ${{ secrets.S3_ACCESS_KEY }}`, consistent with how the adjacent `S3_SECRET_KEY` is already handled via secrets.

