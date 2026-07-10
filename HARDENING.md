<!-- markdownlint-disable -->

# Hardening Report: brunojppb--turbo-cache-server/4.0.10

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **brunojppb--turbo-cache-server/4.0.10** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files use action references pinned to mutable version tags instead of immutable 40-character commit SHAs, making them vulnerable to supply-chain attacks. Affected references include: actions/checkout@v7, docker/setup-buildx-action@v4, docker/login-action@v4, actions/upload-artifact@v7, actions/download-artifact@v8, actions-rust-lang/setup-rust-toolchain@v1, actions-rust-lang/rustfmt@v1, docker/setup-buildx-action@v4.2.0.

Locations:

- `.github/workflows/build-binaries.yml:26`
- `.github/workflows/build-binaries.yml:29`
- `.github/workflows/build-binaries.yml:37`
- `.github/workflows/build-binaries.yml:68`
- `.github/workflows/build-binaries.yml:71`
- `.github/workflows/build.yml:14`
- `.github/workflows/build.yml:19`
- `.github/workflows/ci.yml:13`
- `.github/workflows/ci.yml:14`
- `.github/workflows/ci.yml:15`
- `.github/workflows/release.yml:43`
- `.github/workflows/release.yml:52`
- `.github/workflows/release.yml:64`
- `.github/workflows/release.yml:67`
- `.github/workflows/release.yml:73`

### broad-permissions (severity: medium)

release.yml sets top-level permissions to 'write-all', granting overly broad write access to all GitHub API scopes. This should be replaced with specific minimal permissions required by each job.

Locations:

- `.github/workflows/release.yml:14`

### missing-permissions (severity: medium)

Three workflow files have no top-level permissions block and no job-level permissions blocks. Without explicit permissions, workflows inherit the repository's default token permissions, which may be broader than necessary.

Locations:

- `.github/workflows/build-binaries.yml:1`
- `.github/workflows/build.yml:1`
- `.github/workflows/ci.yml:1`

### script-injection (severity: high)

Multiple run: blocks directly interpolate ${{ }} expressions into shell commands (rule a), and several use unquoted shell variable expansions of user-controlled data (rule b).

(a) Direct expression interpolation in run: blocks:
- build-binaries.yml line 34: `sed -i "s/^version = \".*\"/version = \"${{ inputs.version }}\"/" Cargo.toml` — ${{ inputs.version }} injected directly into shell
- build-binaries.yml line 52: `docker buildx build ... --tag brunojppb/turbo-cache-server-build:${{ inputs.version }} ... --cache-to type=registry,ref=brunojppb/turbo-cache-server-build:${{ inputs.cache-ref }}` — both inputs injected directly
- build-binaries.yml lines 56-58: `docker pull/create ... brunojppb/turbo-cache-server-build:${{ inputs.version }}` — input injected directly
- build-binaries.yml line 76 (macOS job): same sed pattern with ${{ inputs.version }}
- release.yml line 49: `sed -i "s/^version = \".*\"/version = \"${{ github.event.inputs.semver }}\"/" Cargo.toml`
- release.yml line 118: `echo "**Full Changelog**: https://github.com/${{ github.repository }}/compare/...` in run: block
- release.yml line 126: `DOWNLOAD_URL="https://github.com/${{ github.repository }}/releases/download/...` in run: block

(b) Unquoted shell variable expansions of user-controlled data:
- release.yml line 88: `--tag ghcr.io/brunojppb/turbo-cache-server:$CACHE_SERVER_VERSION` (CACHE_SERVER_VERSION holds ${{ github.event.inputs.semver }})
- release.yml line 100: `git tag -a ${CACHE_SERVER_VERSION} -m "Release ${CACHE_SERVER_VERSION}"` — unquoted
- release.yml line 105: `git push origin $CACHE_SERVER_VERSION` — unquoted

Locations:

- `.github/workflows/build-binaries.yml:34`
- `.github/workflows/build-binaries.yml:52`
- `.github/workflows/build-binaries.yml:56`
- `.github/workflows/build-binaries.yml:76`
- `.github/workflows/release.yml:49`
- `.github/workflows/release.yml:88`
- `.github/workflows/release.yml:100`
- `.github/workflows/release.yml:105`
- `.github/workflows/release.yml:118`
- `.github/workflows/release.yml:126`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, broad-permissions, missing-permissions, script-injection

**Notes:**

Fixed all four findings across four workflow files:

1. **unpinned-uses**: Pinned all action references to full 40-character commit SHAs with tag comments:
   - actions/checkout@v7 → @9c091bb21b7c1c1d1991bb908d89e4e9dddfe3e0 # v7
   - docker/setup-buildx-action@v4 / @v4.2.0 → @bb05f3f5519dd87d3ba754cc423b652a5edd6d2c # v4 / # v4.2.0
   - docker/login-action@v4 → @af1e73f918a031802d376d3c8bbc3fe56130a9b0 # v4
   - actions/upload-artifact@v7 → @043fb46d1a93c77aae656e7c1c64a875d1fc6a0a # v7
   - actions/download-artifact@v8 → @3e5f45b2cfb9172054b4087a40e8e0b5a5461e7c # v8
   - actions-rust-lang/setup-rust-toolchain@v1 → @166cdcfd11aee3cb47222f9ddb555ce30ddb9659 # v1
   - actions-rust-lang/rustfmt@v1 → @4066006ec54a31931b9b1fddfd38f2fdf2d27143 # v1

2. **broad-permissions**: Replaced top-level `permissions: write-all` in release.yml with `permissions: contents: read` at top level, and job-level `permissions: contents: write, packages: write` on the release job that actually needs those permissions.

3. **missing-permissions**: Added `permissions: contents: read` top-level blocks to build-binaries.yml, build.yml, and ci.yml.

4. **script-injection**: Moved all `${{ }}` expressions out of run: blocks into env: blocks and referenced them as plain shell variables. Also quoted all shell variable expansions of user-controlled data (e.g. `"${CACHE_SERVER_VERSION}"`, `"${INPUT_VERSION}"`). Replaced `${{ github.repository }}` in release.yml with `GITHUB_REPOSITORY` env var.

