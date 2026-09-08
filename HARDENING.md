<!-- markdownlint-disable -->

# Hardening Report: brunojppb--turbo-cache-server/4.0.18

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **brunojppb--turbo-cache-server/4.0.18** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags instead of pinned full-length SHA commit hashes, making them vulnerable to supply-chain attacks. Failing references include: actions/checkout@v7, docker/setup-buildx-action@v4, docker/login-action@v4, actions/upload-artifact@v7, actions/download-artifact@v8, actions-rust-lang/setup-rust-toolchain@v2, actions-rust-lang/rustfmt@v1, docker/setup-buildx-action@v4.3.0.

Locations:

- `.github/workflows/build-binaries.yml:24`
- `.github/workflows/build-binaries.yml:27`
- `.github/workflows/build-binaries.yml:36`
- `.github/workflows/build-binaries.yml:57`
- `.github/workflows/build-binaries.yml:67`
- `.github/workflows/build-binaries.yml:74`
- `.github/workflows/build-binaries.yml:84`
- `.github/workflows/build.yml:15`
- `.github/workflows/build.yml:22`
- `.github/workflows/ci.yml:12`
- `.github/workflows/ci.yml:14`
- `.github/workflows/ci.yml:15`
- `.github/workflows/ci.yml:22`
- `.github/workflows/ci.yml:23`
- `.github/workflows/release.yml:36`
- `.github/workflows/release.yml:46`
- `.github/workflows/release.yml:52`
- `.github/workflows/release.yml:57`
- `.github/workflows/release.yml:63`

### script-injection (severity: high)

Multiple run: blocks directly interpolate GitHub Actions expressions (${{ ... }}) inside shell command strings, enabling script injection. Sub-rule (a) violations: (1) build-binaries.yml 'Update version on Cargo.toml' steps interpolate `${{ inputs.version }}` directly in a sed command; (2) build-binaries.yml 'build binary' and 'create temp containers' steps interpolate `${{ inputs.version }}` and `${{ inputs.cache-ref }}` directly in docker commands; (3) release.yml 'Update version on Cargo.toml' interpolates `${{ github.event.inputs.semver }}` in a sed command; (4) release.yml 'Create GitHub Release' interpolates `${{ github.repository }}` directly in shell echo commands. Sub-rule (b) violations: (5) release.yml 'Build and push Docker image' uses unquoted `$CACHE_SERVER_VERSION` in docker tag arguments; (6) release.yml 'Commit new binary' uses unquoted `${CACHE_SERVER_VERSION}` in git tag/commit commands; (7) release.yml 'Push new tag' uses unquoted `$CACHE_SERVER_VERSION` in git push.

Locations:

- `.github/workflows/build-binaries.yml:33`
- `.github/workflows/build-binaries.yml:47`
- `.github/workflows/build-binaries.yml:51`
- `.github/workflows/build-binaries.yml:52`
- `.github/workflows/build-binaries.yml:79`
- `.github/workflows/release.yml:40`
- `.github/workflows/release.yml:72`
- `.github/workflows/release.yml:80`
- `.github/workflows/release.yml:86`
- `.github/workflows/release.yml:91`
- `.github/workflows/release.yml:113`
- `.github/workflows/release.yml:124`

### github-env-injection (severity: high)

The 'Read Rust version from rust-toolchain.toml' steps write a value derived from `steps.rust.outputs.version` (a workflow-controllable step output) to $GITHUB_OUTPUT without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). The pattern `echo "version=$(...)" >> "$GITHUB_OUTPUT"` embeds the output of a shell command substitution directly into the environment file. Additionally, in build-binaries.yml, `${{ inputs.version }}` is interpolated directly into a sed command whose output could influence subsequent steps.

Locations:

- `.github/workflows/build-binaries.yml:38`
- `.github/workflows/build-binaries.yml:73`
- `.github/workflows/release.yml:68`

### broad-permissions (severity: medium)

release.yml sets top-level `permissions: "write-all"`, granting overly broad write access to all GitHub API scopes. This should be replaced with specific minimal permissions (e.g., contents: write, packages: write) required by the workflow.

Locations:

- `.github/workflows/release.yml:13`

### missing-permissions (severity: medium)

Three workflow files have no top-level `permissions:` key and no job-level `permissions:` blocks on any job. Without explicit permissions, workflows inherit the repository's default token permissions (which may be read/write). Affected files: build-binaries.yml, build.yml, ci.yml.

Locations:

- `.github/workflows/build-binaries.yml:1`
- `.github/workflows/build.yml:1`
- `.github/workflows/ci.yml:1`

### hardcoded-credentials (severity: high)

ci.yml contains literal hardcoded credential values for S3_ACCESS_KEY (`some-access-key`) and S3_SECRET_KEY (`some-secret-key`) set as environment variables in the test job. Even if these are placeholder/test values, hardcoding credential-named keys with non-expression literal values violates the hardcoded-credentials policy and sets a dangerous precedent. These should be stored as GitHub Actions secrets and referenced via `${{ secrets.S3_ACCESS_KEY }}`.

Locations:

- `.github/workflows/ci.yml:31`
- `.github/workflows/ci.yml:32`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, broad-permissions, missing-permissions, hardcoded-credentials

**Notes:**

Fixed all 6 findings across 4 workflow files:

1. **unpinned-uses**: Pinned all action references to full commit SHAs:
   - actions/checkout@v7 → @3d3c42e5aac5ba805825da76410c181273ba90b1
   - docker/setup-buildx-action@v4/v4.3.0 → @37fe631027851001ddb9b187196cc803df7f5f0e
   - docker/login-action@v4 → @dbcb813823bdd20940b903addbd779551569679f
   - actions/upload-artifact@v7 → @043fb46d1a93c77aae656e7c1c64a875d1fc6a0a
   - actions/download-artifact@v8 → @3e5f45b2cfb9172054b4087a40e8e0b5a5461e7c
   - actions-rust-lang/setup-rust-toolchain@v2 → @ecabd13d1c56bd1345c230e542e9144811ad706f
   - actions-rust-lang/rustfmt@v1 → @4066006ec54a31931b9b1fddfd38f2fdf2d27143

2. **script-injection**: Moved all ${{ inputs.version }}, ${{ inputs.cache-ref }}, ${{ github.event.inputs.semver }}, and ${{ github.repository }} expressions out of run: shell strings into env: blocks. Variables are now referenced as ${INPUT_VERSION}, ${INPUT_CACHE_REF}, ${INPUT_SEMVER}, ${GH_REPO} etc. Also fixed unquoted variable expansions in docker tag/git commands by wrapping in double quotes.

3. **github-env-injection**: Fixed all 3 'Read Rust version' steps in build-binaries.yml (both jobs) and release.yml to sanitize the value before writing to $GITHUB_OUTPUT using `printf '%s' "$raw" | tr -d '\n\r'`.

4. **broad-permissions**: Replaced `permissions: "write-all"` in release.yml with specific minimal permissions: `contents: write` and `packages: write`.

5. **missing-permissions**: Added `permissions: contents: read` to build-binaries.yml, build.yml, and ci.yml.

6. **hardcoded-credentials**: Replaced hardcoded `some-access-key` and `some-secret-key` literal values in ci.yml with `${{ secrets.S3_ACCESS_KEY }}` and `${{ secrets.S3_SECRET_KEY }}`.

