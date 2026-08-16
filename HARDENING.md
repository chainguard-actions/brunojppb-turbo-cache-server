<!-- markdownlint-disable -->

# Hardening Report: brunojppb--turbo-cache-server/4.0.16

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **brunojppb--turbo-cache-server/4.0.16** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Direct ${{ }} expression interpolation inside run: shell commands. In build-binaries.yml, ${{ inputs.version }} and ${{ inputs.cache-ref }} are interpolated directly into docker and sed commands (e.g., `docker buildx build ... :${{ inputs.version }}`, `docker pull ... :${{ inputs.version }}`, `docker create ... :${{ inputs.version }}`, `sed -i "..." ${{ inputs.version }}`). In release.yml, ${{ github.event.inputs.semver }} is interpolated directly into a sed command (`sed -i "..." ${{ github.event.inputs.semver }}`), and ${{ github.repository }} is interpolated directly into echo commands inside a run: block. These are sub-rule (a) violations — any ${{ }} expression inside a run: block is a script-injection risk.

Locations:

- `.github/workflows/build-binaries.yml:31`
- `.github/workflows/build-binaries.yml:47`
- `.github/workflows/build-binaries.yml:52`
- `.github/workflows/build-binaries.yml:79`
- `.github/workflows/release.yml:44`
- `.github/workflows/release.yml:84`
- `.github/workflows/release.yml:97`

### script-injection (severity: high)

Unquoted shell variable expansion of untrusted data (sub-rule b). In release.yml, $CACHE_SERVER_VERSION (sourced from ${{ github.event.inputs.semver }}) is used unquoted in run: blocks: `--tag ghcr.io/brunojppb/turbo-cache-server:$CACHE_SERVER_VERSION` and `git push origin $CACHE_SERVER_VERSION` and `git tag -a ${CACHE_SERVER_VERSION}`. Unquoted expansions allow shell metacharacter injection.

Locations:

- `.github/workflows/release.yml:72`
- `.github/workflows/release.yml:80`
- `.github/workflows/release.yml:81`
- `.github/workflows/release.yml:86`

### broad-permissions (severity: medium)

The release.yml workflow sets top-level `permissions: "write-all"`, granting overly broad write access to all GitHub API scopes. This should be replaced with specific minimal permissions (e.g., contents: write, packages: write).

Locations:

- `.github/workflows/release.yml:14`

### missing-permissions (severity: medium)

The following workflow files have no top-level `permissions:` key and no job-level `permissions:` keys on any job, meaning they inherit the default repository permissions (which may include write access): build-binaries.yml, build.yml, and ci.yml.

Locations:

- `.github/workflows/build-binaries.yml:1`
- `.github/workflows/build.yml:1`
- `.github/workflows/ci.yml:1`

### unpinned-uses (severity: high)

Multiple `uses:` references across workflow files use mutable version tags instead of full 40-character commit SHA digests, making them vulnerable to supply-chain attacks if the referenced tag is moved. Failing references include: actions/checkout@v7, docker/setup-buildx-action@v4, docker/login-action@v4, actions/upload-artifact@v7, actions/download-artifact@v8, actions-rust-lang/setup-rust-toolchain@v1, actions-rust-lang/rustfmt@v1, docker/setup-buildx-action@v4.2.0.

Locations:

- `.github/workflows/build-binaries.yml:25`
- `.github/workflows/build-binaries.yml:28`
- `.github/workflows/build-binaries.yml:33`
- `.github/workflows/build-binaries.yml:57`
- `.github/workflows/build-binaries.yml:68`
- `.github/workflows/build-binaries.yml:75`
- `.github/workflows/build-binaries.yml:93`
- `.github/workflows/build.yml:12`
- `.github/workflows/build.yml:19`
- `.github/workflows/ci.yml:12`
- `.github/workflows/ci.yml:14`
- `.github/workflows/ci.yml:16`
- `.github/workflows/ci.yml:24`
- `.github/workflows/ci.yml:25`
- `.github/workflows/release.yml:40`
- `.github/workflows/release.yml:55`
- `.github/workflows/release.yml:60`
- `.github/workflows/release.yml:65`
- `.github/workflows/release.yml:68`

### hardcoded-credentials (severity: high)

Hardcoded credential-like values found in ci.yml: `S3_ACCESS_KEY: some-access-key` and `S3_SECRET_KEY: some-secret-key`. Even though these appear to be placeholder test values, they are literal non-expression strings assigned to names containing `access_key` and `secret_key`, matching the hardcoded credentials pattern. Real credentials should always be stored in GitHub Secrets and referenced via ${{ secrets.NAME }}.

Locations:

- `.github/workflows/ci.yml:30`
- `.github/workflows/ci.yml:31`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, broad-permissions, missing-permissions, unpinned-uses, hardcoded-credentials

**Notes:**

Fixed all findings across 4 workflow files:

1. build-binaries.yml: Pinned all 6 action references to full SHAs; moved ${{ inputs.version }} and ${{ inputs.cache-ref }} into env: blocks (INPUT_VERSION, INPUT_CACHE_REF) to eliminate script injection; added permissions: contents: read.

2. release.yml: Replaced 'permissions: write-all' with 'contents: write, packages: write'; pinned all 5 action references to full SHAs; moved ${{ github.event.inputs.semver }} into env: blocks (INPUT_SEMVER / CACHE_SERVER_VERSION) and ${{ github.repository }} into env: block (GH_REPO); quoted all $CACHE_SERVER_VERSION expansions with double quotes.

3. build.yml: Pinned actions/download-artifact@v8 and actions/upload-artifact@v7 to full SHAs; added permissions: contents: read.

4. ci.yml: Pinned actions/checkout@v7, actions-rust-lang/setup-rust-toolchain@v1, and actions-rust-lang/rustfmt@v1 to full SHAs; replaced hardcoded 'some-access-key' and 'some-secret-key' with ${{ secrets.S3_ACCESS_KEY }} and ${{ secrets.S3_SECRET_KEY }}; added permissions: contents: read.

