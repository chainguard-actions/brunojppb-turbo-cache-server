<!-- markdownlint-disable -->

# Hardening Report: brunojppb--turbo-cache-server/4.0.13

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **brunojppb--turbo-cache-server/4.0.13** was hardened automatically. 6 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks in build-binaries.yml directly interpolate ${{ inputs.version }} and ${{ inputs.cache-ref }} expressions into shell commands, enabling script injection. Affected lines include: (a) `sed -i "s/^version = \".*\"/version = \"${{ inputs.version }}\"/" Cargo.toml` (line 34, Linux job; line 81, macOS job); (b) `docker buildx build ... --tag brunojppb/turbo-cache-server-build:${{ inputs.version }} ... --cache-to type=registry,ref=brunojppb/turbo-cache-server-build:${{ inputs.cache-ref }}` (line 52); (c) `docker pull brunojppb/turbo-cache-server-build:${{ inputs.version }}` and `docker create ... brunojppb/turbo-cache-server-build:${{ inputs.version }}` (lines 56–58). These are rule (a) violations — ${{ }} expressions interpolated directly in run: scripts before the shell sees them.

Locations:

- `.github/workflows/build-binaries.yml:34`
- `.github/workflows/build-binaries.yml:52`
- `.github/workflows/build-binaries.yml:56`
- `.github/workflows/build-binaries.yml:81`

### script-injection (severity: high)

Multiple run: blocks in release.yml directly interpolate ${{ github.event.inputs.semver }} and ${{ github.repository }} expressions into shell commands, enabling script injection. Affected lines include: (a) `sed -i "s/^version = \".*\"/version = \"${{ github.event.inputs.semver }}\"/" Cargo.toml` (rule a — direct expression in run:); (b) `echo "**Full Changelog**: https://github.com/${{ github.repository }}/compare/..."` and `DOWNLOAD_URL="https://github.com/${{ github.repository }}/releases/download/..."` (rule a — direct expression in run:); (c) `docker buildx build ... --tag ghcr.io/brunojppb/turbo-cache-server:$CACHE_SERVER_VERSION` and `git push origin $CACHE_SERVER_VERSION` — $CACHE_SERVER_VERSION holds ${{ github.event.inputs.semver }} and is used unquoted in shell (rule b — unquoted expansion of workflow-controlled env var).

Locations:

- `.github/workflows/release.yml:36`
- `.github/workflows/release.yml:70`
- `.github/workflows/release.yml:78`
- `.github/workflows/release.yml:87`
- `.github/workflows/release.yml:100`

### unpinned-uses (severity: high)

All uses: references across all workflow files use mutable tag refs instead of pinned 40-character SHA digests, making the workflows vulnerable to supply-chain attacks if any referenced action is compromised or its tag is moved. Unpinned references include: actions/checkout@v7, docker/setup-buildx-action@v4, docker/login-action@v4, actions/upload-artifact@v7, actions/download-artifact@v8, actions-rust-lang/setup-rust-toolchain@v1, actions-rust-lang/rustfmt@v1, docker/setup-buildx-action@v4.2.0.

Locations:

- `.github/workflows/build-binaries.yml:26`
- `.github/workflows/build-binaries.yml:29`
- `.github/workflows/build-binaries.yml:37`
- `.github/workflows/build-binaries.yml:63`
- `.github/workflows/build-binaries.yml:74`
- `.github/workflows/build-binaries.yml:77`
- `.github/workflows/build-binaries.yml:91`
- `.github/workflows/build.yml:14`
- `.github/workflows/build.yml:20`
- `.github/workflows/ci.yml:13`
- `.github/workflows/ci.yml:15`
- `.github/workflows/ci.yml:16`
- `.github/workflows/ci.yml:24`
- `.github/workflows/ci.yml:25`
- `.github/workflows/release.yml:23`
- `.github/workflows/release.yml:49`
- `.github/workflows/release.yml:55`
- `.github/workflows/release.yml:58`
- `.github/workflows/release.yml:63`

### broad-permissions (severity: medium)

release.yml sets top-level `permissions: "write-all"`, granting every job in the workflow write access to all GitHub API scopes. This is overly broad and should be replaced with the minimal specific permissions required (e.g., contents: write, packages: write).

Locations:

- `.github/workflows/release.yml:13`

### missing-permissions (severity: medium)

build-binaries.yml, build.yml, and ci.yml have no top-level permissions: key and no job-level permissions: blocks on any job. Without explicit permissions, workflows inherit the repository's default token permissions, which may be overly broad. Each workflow should declare the minimal permissions it needs.

Locations:

- `.github/workflows/build-binaries.yml:1`
- `.github/workflows/build.yml:1`
- `.github/workflows/ci.yml:1`

### hardcoded-credentials (severity: high)

ci.yml contains a hardcoded literal value assigned to S3_SECRET_KEY (`S3_SECRET_KEY: some-secret-key`). Even though this appears to be a placeholder/test value, it matches the pattern of a hardcoded secret and should be replaced with a GitHub Actions secret expression (e.g., `${{ secrets.S3_SECRET_KEY }}`) to prevent accidental use of real credentials in this pattern.

Locations:

- `.github/workflows/ci.yml:30`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, broad-permissions, missing-permissions, hardcoded-credentials

**Notes:**

Fixed all 6 findings across 4 workflow files:

1. script-injection (build-binaries.yml): Moved ${{ inputs.version }} and ${{ inputs.cache-ref }} into env: blocks as INPUT_VERSION and INPUT_CACHE_REF. Shell scripts use ${INPUT_VERSION} and ${INPUT_CACHE_REF} with proper quoting.

2. script-injection (release.yml): Moved ${{ github.event.inputs.semver }} to SEMVER/CACHE_SERVER_VERSION env vars and ${{ github.repository }} to GITHUB_REPOSITORY env var. All shell expansions are properly double-quoted.

3. unpinned-uses: All 7 distinct action references pinned to full 40-char SHA digests with tag comments preserved (actions/checkout, docker/setup-buildx-action, docker/login-action, actions/upload-artifact, actions/download-artifact, actions-rust-lang/setup-rust-toolchain, actions-rust-lang/rustfmt).

4. broad-permissions (release.yml): Replaced `permissions: "write-all"` with `contents: write` and `packages: write`.

5. missing-permissions: Added `permissions: contents: read` to build-binaries.yml, build.yml, and ci.yml.

6. hardcoded-credentials (ci.yml): Replaced literal `some-secret-key` with `${{ secrets.S3_SECRET_KEY }}`.

### Iteration 2

**Fixes applied:** hardcoded-credentials

**Notes:**

Replaced the hardcoded literal credential `S3_ACCESS_KEY: some-access-key` with `S3_ACCESS_KEY: ${{ secrets.S3_ACCESS_KEY }}` in `.github/workflows/ci.yml` (line 34). This makes it consistent with the adjacent `S3_SECRET_KEY` which was already using a secret reference. No other findings were present.

