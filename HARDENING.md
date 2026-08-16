<!-- markdownlint-disable -->

# Hardening Report: brunojppb--turbo-cache-server/4.0.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **brunojppb--turbo-cache-server/4.0.6** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks directly interpolate ${{ }} expressions into shell commands, enabling script injection. In build-binaries.yml: (a) line 31 interpolates ${{ inputs.version }} directly in a sed command; (a) line 40 interpolates ${{ inputs.version }} and ${{ inputs.cache-ref }} in a docker buildx run command; (a) lines 46-48 interpolate ${{ inputs.version }} in docker pull/create commands; (a) line 73 repeats the sed injection in the build-macos job. In release.yml: (a) line 44 interpolates ${{ github.event.inputs.semver }} in a sed command; (a) line 90 interpolates ${{ github.repository }} directly in an echo command inside a run: block; (b) lines 77, 83, 95 use $CACHE_SERVER_VERSION (sourced from ${{ github.event.inputs.semver }}) unquoted in docker tag, git tag, and git push commands.

Locations:

- `.github/workflows/build-binaries.yml:31`
- `.github/workflows/build-binaries.yml:40`
- `.github/workflows/build-binaries.yml:46`
- `.github/workflows/build-binaries.yml:73`
- `.github/workflows/release.yml:44`
- `.github/workflows/release.yml:77`
- `.github/workflows/release.yml:83`
- `.github/workflows/release.yml:90`
- `.github/workflows/release.yml:95`

### unpinned-uses (severity: high)

All workflow files reference actions using mutable version tags instead of pinned 40-character SHA digests, making them vulnerable to supply-chain attacks. Failing references include: actions/checkout@v6, docker/setup-buildx-action@v4, docker/login-action@v4, actions/upload-artifact@v7, actions/download-artifact@v8, actions-rust-lang/setup-rust-toolchain@v1, actions-rust-lang/rustfmt@v1, docker/setup-buildx-action@v4.0.0.

Locations:

- `.github/workflows/build-binaries.yml:25`
- `.github/workflows/build-binaries.yml:28`
- `.github/workflows/build-binaries.yml:33`
- `.github/workflows/build-binaries.yml:60`
- `.github/workflows/build-binaries.yml:63`
- `.github/workflows/build-binaries.yml:78`
- `.github/workflows/build-binaries.yml:87`
- `.github/workflows/build.yml:13`
- `.github/workflows/build.yml:20`
- `.github/workflows/ci.yml:13`
- `.github/workflows/ci.yml:16`
- `.github/workflows/ci.yml:19`
- `.github/workflows/ci.yml:26`
- `.github/workflows/ci.yml:27`
- `.github/workflows/release.yml:41`
- `.github/workflows/release.yml:55`
- `.github/workflows/release.yml:60`
- `.github/workflows/release.yml:65`
- `.github/workflows/release.yml:70`

### broad-permissions (severity: medium)

release.yml sets top-level permissions to "write-all", granting overly broad write access across all scopes. This should be replaced with specific minimal permissions (e.g., contents: write, packages: write) required by the workflow.

Locations:

- `.github/workflows/release.yml:14`

### missing-permissions (severity: medium)

build-binaries.yml, build.yml, and ci.yml have no top-level permissions: key and no job-level permissions: keys on any of their jobs. Without explicit permissions, workflows inherit the default repository permissions, which may be overly broad.

Locations:

- `.github/workflows/build-binaries.yml:1`
- `.github/workflows/build.yml:1`
- `.github/workflows/ci.yml:1`

### hardcoded-credentials (severity: high)

ci.yml contains hardcoded literal credential values in env: blocks: S3_ACCESS_KEY is set to the literal string "some-access-key" and S3_SECRET_KEY is set to "some-secret-key". Even if these are placeholder/test values, they match the hardcoded-credentials pattern and should be replaced with ${{ secrets.* }} references.

Locations:

- `.github/workflows/ci.yml:33`
- `.github/workflows/ci.yml:34`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, broad-permissions, missing-permissions, hardcoded-credentials

**Notes:**

Fixed all 5 findings across 4 workflow files:

1. script-injection: Moved all ${{ }} expressions from run: blocks into step env: blocks in build-binaries.yml (inputs.version, inputs.cache-ref) and release.yml (github.event.inputs.semver, github.repository). All shell variable references are now properly quoted.

2. unpinned-uses: Pinned all 8 action references to full 40-char SHA digests with tag comments: actions/checkout@df4cb1c, docker/setup-buildx-action@bb05f3f (v4) and @4d04d5d (v4.0.0), docker/login-action@af1e73f, actions/upload-artifact@043fb46, actions/download-artifact@3e5f45b, actions-rust-lang/setup-rust-toolchain@166cdcf, actions-rust-lang/rustfmt@4066006.

3. broad-permissions: Replaced `permissions: "write-all"` in release.yml with `contents: write` and `packages: write`.

4. missing-permissions: Added `permissions: contents: read` to build-binaries.yml, build.yml, and ci.yml.

5. hardcoded-credentials: Replaced literal `some-access-key` and `some-secret-key` in ci.yml with `${{ secrets.S3_ACCESS_KEY }}` and `${{ secrets.S3_SECRET_KEY }}`.

