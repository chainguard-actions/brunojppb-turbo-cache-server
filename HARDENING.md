<!-- markdownlint-disable -->

# Hardening Report: brunojppb--turbo-cache-server/4.0.7

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **brunojppb--turbo-cache-server/4.0.7** was hardened automatically. 5 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference GitHub Actions using mutable tags instead of pinned SHA digests, making them vulnerable to supply-chain attacks if the tag is moved.

- build-binaries.yml: actions/checkout@v6, docker/setup-buildx-action@v4, docker/login-action@v4, actions/upload-artifact@v7
- build.yml: actions/download-artifact@v8, actions/upload-artifact@v7
- ci.yml: actions/checkout@v6, actions-rust-lang/setup-rust-toolchain@v1, actions-rust-lang/rustfmt@v1
- release.yml: actions/checkout@v6, actions/download-artifact@v8, docker/setup-buildx-action@v4.1.0, docker/login-action@v4

Locations:

- `.github/workflows/build-binaries.yml:25`
- `.github/workflows/build-binaries.yml:28`
- `.github/workflows/build-binaries.yml:36`
- `.github/workflows/build-binaries.yml:61`
- `.github/workflows/build-binaries.yml:67`
- `.github/workflows/build-binaries.yml:70`
- `.github/workflows/build-binaries.yml:88`
- `.github/workflows/build.yml:17`
- `.github/workflows/build.yml:23`
- `.github/workflows/ci.yml:11`
- `.github/workflows/ci.yml:14`
- `.github/workflows/ci.yml:17`
- `.github/workflows/release.yml:28`
- `.github/workflows/release.yml:38`
- `.github/workflows/release.yml:55`
- `.github/workflows/release.yml:62`

### script-injection (severity: high)

GitHub Actions expressions are directly interpolated inside run: shell commands (rule a), allowing an attacker to inject arbitrary shell commands via workflow inputs or event context.

build-binaries.yml:
- Line 34: `sed -i "s/^version = \".*\"/version = \"${{ inputs.version }}\"/" Cargo.toml` — inputs.version injected directly into sed command
- Line 44: `docker buildx build ... --tag brunojppb/turbo-cache-server-build:${{ inputs.version }} ... --cache-to type=registry,ref=brunojppb/turbo-cache-server-build:${{ inputs.cache-ref }} ...` — inputs.version and inputs.cache-ref injected into docker command
- Lines 47-49: `docker pull/create` commands with `${{ inputs.version }}` injected
- Line 74: `sed -i ... ${{ inputs.version }} ...` in build-macos job

release.yml:
- Line 47: `sed -i "s/^version = \".*\"/version = \"${{ github.event.inputs.semver }}\"/" Cargo.toml` — semver injected into sed
- Line 88: `echo "**Full Changelog**: https://github.com/${{ github.repository }}/compare/..."` — github.repository injected into shell

Locations:

- `.github/workflows/build-binaries.yml:34`
- `.github/workflows/build-binaries.yml:44`
- `.github/workflows/build-binaries.yml:47`
- `.github/workflows/build-binaries.yml:74`
- `.github/workflows/release.yml:47`
- `.github/workflows/release.yml:88`

### broad-permissions (severity: medium)

release.yml sets `permissions: "write-all"` at the top level, granting all jobs overly broad write access to every GitHub API scope. This should be replaced with specific minimal permissions (e.g., contents: write, packages: write) scoped to only what each job requires.

Locations:

- `.github/workflows/release.yml:13`

### missing-permissions (severity: medium)

The following workflow files have no top-level `permissions:` key and no job-level `permissions:` blocks on any job. Without explicit permissions, the default token permissions (which may be broad depending on repository settings) are used.

- build-binaries.yml: no permissions declared at top level or on build-linux/build-macos jobs
- build.yml: no permissions declared at top level or on build/collect jobs
- ci.yml: no permissions declared at top level or on lint/test jobs

Locations:

- `.github/workflows/build-binaries.yml:1`
- `.github/workflows/build.yml:1`
- `.github/workflows/ci.yml:1`

### hardcoded-credentials (severity: high)

ci.yml hardcodes literal credential values in the test job's env block: `S3_SECRET_KEY: some-secret-key`. Even though these appear to be placeholder test values, they are literal strings assigned to a secret-named field and match the hardcoded-credentials pattern. Real credentials should be stored in GitHub Secrets and referenced via `${{ secrets.S3_SECRET_KEY }}`.

Locations:

- `.github/workflows/ci.yml:30`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, broad-permissions, missing-permissions, hardcoded-credentials

**Notes:**

Fixed all 5 findings across 4 workflow files:

1. **unpinned-uses**: Pinned all action references to full commit SHAs:
   - actions/checkout@v6 → @d23441a48e516b6c34aea4fa41551a30e30af803
   - docker/setup-buildx-action@v4 → @37fe631027851001ddb9b187196cc803df7f5f0e
   - docker/login-action@v4 → @dbcb813823bdd20940b903addbd779551569679f
   - actions/upload-artifact@v7 → @043fb46d1a93c77aae656e7c1c64a875d1fc6a0a
   - actions/download-artifact@v8 → @3e5f45b2cfb9172054b4087a40e8e0b5a5461e7c
   - actions-rust-lang/setup-rust-toolchain@v1 → @166cdcfd11aee3cb47222f9ddb555ce30ddb9659
   - actions-rust-lang/rustfmt@v1 → @4066006ec54a31931b9b1fddfd38f2fdf2d27143
   - docker/setup-buildx-action@v4.1.0 → @d7f5e7f509e45cec5c76c4d5afdd7de93d0b3df5

2. **script-injection**: Moved all ${{ inputs.version }}, ${{ inputs.cache-ref }}, ${{ github.event.inputs.semver }}, and ${{ github.repository }} expressions from run: shell strings into env: blocks, referencing them as plain shell variables.

3. **broad-permissions**: Replaced `permissions: "write-all"` in release.yml with `contents: write` and `packages: write`.

4. **missing-permissions**: Added `permissions: contents: read` to build-binaries.yml, build.yml, and ci.yml.

5. **hardcoded-credentials**: Replaced literal `some-secret-key` and `some-access-key` values in ci.yml with `${{ secrets.S3_SECRET_KEY }}` and `${{ secrets.S3_ACCESS_KEY }}` references.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed two unquoted variable expansions of CACHE_SERVER_VERSION in .github/workflows/release.yml: (1) In the 'Commit new binary' step, changed `git tag -a ${CACHE_SERVER_VERSION}` to `git tag -a "${CACHE_SERVER_VERSION}"`. (2) In the 'Push new tag' step, changed `git push origin $CACHE_SERVER_VERSION` to `git push origin "${CACHE_SERVER_VERSION}"`. Both variables were already safely assigned via the step's env block, so only quoting was needed to prevent word splitting and shell metacharacter injection from a crafted semver input.

