<!-- markdownlint-disable -->

# Hardening Report: brunojppb--turbo-cache-server/4.0.12

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **brunojppb--turbo-cache-server/4.0.12** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All `uses:` references across all workflow files are pinned to mutable version tags instead of immutable 40-character SHA digests, making the workflows vulnerable to supply-chain attacks if any referenced action is compromised or its tag is moved.

Failing references include:
- build-binaries.yml: actions/checkout@v7, docker/setup-buildx-action@v4, docker/login-action@v4, actions/upload-artifact@v7 (×2)
- build.yml: actions/download-artifact@v8, actions/upload-artifact@v7
- ci.yml: actions/checkout@v7 (×2), actions-rust-lang/setup-rust-toolchain@v1 (×2), actions-rust-lang/rustfmt@v1
- release.yml: actions/checkout@v7, actions/download-artifact@v8, docker/setup-buildx-action@v4.2.0, docker/login-action@v4 (×2)

Locations:

- `.github/workflows/build-binaries.yml:19`
- `.github/workflows/build-binaries.yml:22`
- `.github/workflows/build-binaries.yml:33`
- `.github/workflows/build-binaries.yml:72`
- `.github/workflows/build-binaries.yml:75`
- `.github/workflows/build-binaries.yml:88`
- `.github/workflows/build-binaries.yml:101`
- `.github/workflows/build.yml:13`
- `.github/workflows/build.yml:20`
- `.github/workflows/ci.yml:11`
- `.github/workflows/ci.yml:13`
- `.github/workflows/ci.yml:14`
- `.github/workflows/ci.yml:22`
- `.github/workflows/ci.yml:23`
- `.github/workflows/release.yml:28`
- `.github/workflows/release.yml:35`
- `.github/workflows/release.yml:44`
- `.github/workflows/release.yml:49`
- `.github/workflows/release.yml:54`

### broad-permissions (severity: medium)

release.yml sets `permissions: "write-all"` at the top level, granting every job in the workflow overly broad write access to all repository scopes. This should be replaced with specific minimal permissions (e.g., `contents: write`, `packages: write`) scoped to only what each job actually needs.

Locations:

- `.github/workflows/release.yml:13`

### missing-permissions (severity: medium)

The following workflow files have no top-level `permissions:` key and no per-job `permissions:` keys, meaning they run with the default (potentially broad) repository permissions:
- build-binaries.yml: no permissions block at any level
- build.yml: no permissions block at any level
- ci.yml: no permissions block at any level

Each workflow should declare minimal explicit permissions (e.g., `permissions: read-all` or specific scopes).

Locations:

- `.github/workflows/build-binaries.yml:1`
- `.github/workflows/build.yml:1`
- `.github/workflows/ci.yml:1`

### script-injection (severity: high)

Multiple `run:` blocks directly interpolate GitHub Actions expressions into shell command strings, violating rule (a). This allows an attacker who controls the input values to inject arbitrary shell commands.

1. build-binaries.yml — `sed` run block interpolates `${{ inputs.version }}` directly into a shell string:
   `sed -i "s/^version = \".*\"/version = \"${{ inputs.version }}\"/" Cargo.toml`
   (appears in both build-linux and build-macos jobs)

2. build-binaries.yml — docker run block interpolates `${{ inputs.version }}` and `${{ inputs.cache-ref }}` directly:
   `docker buildx build ... --tag brunojppb/turbo-cache-server-build:${{ inputs.version }} ... --cache-to type=registry,ref=brunojppb/turbo-cache-server-build:${{ inputs.cache-ref }} ...`
   and `docker pull brunojppb/turbo-cache-server-build:${{ inputs.version }}`
   and `docker create ... brunojppb/turbo-cache-server-build:${{ inputs.version }}`

3. release.yml — `sed` run block interpolates `${{ github.event.inputs.semver }}` directly:
   `sed -i "s/^version = \".*\"/version = \"${{ github.event.inputs.semver }}\"/" Cargo.toml`

4. release.yml — `echo` run block interpolates `${{ github.repository }}` directly into a shell string written to a file:
   `DOWNLOAD_URL="https://github.com/${{ github.repository }}/releases/download/${CACHE_SERVER_VERSION}"`
   `echo "**Full Changelog**: https://github.com/${{ github.repository }}/compare/..."`

Locations:

- `.github/workflows/build-binaries.yml:27`
- `.github/workflows/build-binaries.yml:40`
- `.github/workflows/build-binaries.yml:44`
- `.github/workflows/build-binaries.yml:45`
- `.github/workflows/build-binaries.yml:82`
- `.github/workflows/release.yml:32`
- `.github/workflows/release.yml:88`
- `.github/workflows/release.yml:104`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, broad-permissions, missing-permissions, script-injection

**Notes:**

Fixed all four findings across build-binaries.yml, build.yml, ci.yml, and release.yml:

1. unpinned-uses: Pinned all 8 unique action references to their full 40-char SHA digests (actions/checkout, docker/setup-buildx-action, docker/login-action, actions/upload-artifact, actions/download-artifact, actions-rust-lang/setup-rust-toolchain, actions-rust-lang/rustfmt). Original tags preserved as inline comments.

2. broad-permissions: Replaced `permissions: "write-all"` in release.yml with `contents: write` and `packages: write` (minimum needed for creating GitHub releases and pushing to ghcr.io).

3. missing-permissions: Added `permissions: contents: read` to build-binaries.yml, build.yml, and ci.yml.

4. script-injection: Moved all ${{ }} expressions out of run: shell strings into env: blocks. In build-binaries.yml: inputs.version → INPUT_VERSION, inputs.cache-ref → INPUT_CACHE_REF. In release.yml: github.event.inputs.semver → INPUT_SEMVER, github.repository → GITHUB_REPOSITORY. All shell references use ${VAR} syntax.

### Iteration 2

**Fixes applied:** hardcoded-credentials, script-injection

**Notes:**

1. ci.yml: Replaced hardcoded literal credential values `some-access-key` and `some-secret-key` with `${{ secrets.S3_ACCESS_KEY }}` and `${{ secrets.S3_SECRET_KEY }}` secret references. 2. release.yml: Added double-quotes around `${CACHE_SERVER_VERSION}` in the 'Commit new binary' step (`git tag -a "${CACHE_SERVER_VERSION}"`) and the 'Push new tag' step (`git push origin "${CACHE_SERVER_VERSION}"`), preventing shell word-splitting and glob expansion of the workflow-dispatch semver input.

