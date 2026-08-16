<!-- markdownlint-disable -->

# Hardening Report: brunojppb--turbo-cache-server/4.0.8

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **brunojppb--turbo-cache-server/4.0.8** was hardened automatically. 5 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks in build-binaries.yml directly interpolate ${{ inputs.version }} and ${{ inputs.cache-ref }} into shell commands. These workflow_call inputs are caller-controlled and allow command injection. Affected steps: 'Update version on Cargo.toml' (sed command with ${{ inputs.version }}), 'build binary' (docker buildx with ${{ inputs.version }} and ${{ inputs.cache-ref }}), 'create temp containers' (docker commands with ${{ inputs.version }}). Sub-rule (a): direct expression interpolation in run: blocks.

Locations:

- `.github/workflows/build-binaries.yml:33`
- `.github/workflows/build-binaries.yml:41`
- `.github/workflows/build-binaries.yml:47`
- `.github/workflows/build-binaries.yml:71`

### script-injection (severity: high)

Multiple run: blocks in release.yml directly interpolate ${{ github.event.inputs.semver }} and ${{ github.repository }} into shell commands. These workflow_dispatch inputs are user-controlled and allow command injection. Affected steps: 'Update version on Cargo.toml' (sed with ${{ github.event.inputs.semver }}), 'Build and push Docker image' (docker buildx with $CACHE_SERVER_VERSION set from ${{ github.event.inputs.semver }}), 'Create GitHub Release' (gh release create with ${{ github.repository }} interpolated into a run: block). Sub-rule (a): direct expression interpolation in run: blocks.

Locations:

- `.github/workflows/release.yml:38`
- `.github/workflows/release.yml:68`
- `.github/workflows/release.yml:96`

### broad-permissions (severity: medium)

release.yml sets top-level permissions to "write-all", granting overly broad write access across all scopes (contents, packages, pull-requests, etc.). This should be replaced with the minimal specific permissions required by each job.

Locations:

- `.github/workflows/release.yml:13`

### missing-permissions (severity: medium)

build-binaries.yml, build.yml, and ci.yml have no top-level permissions: key and no job-level permissions: blocks. Without explicit permissions, workflows inherit the repository default (often write access to contents), violating the principle of least privilege.

Locations:

- `.github/workflows/build-binaries.yml:1`
- `.github/workflows/build.yml:1`
- `.github/workflows/ci.yml:1`

### unpinned-uses (severity: high)

All four workflow files reference GitHub Actions using mutable version tags instead of full 40-character SHA commit hashes, making them vulnerable to supply-chain attacks if a tag is moved. Unpinned references include: actions/checkout@v6, docker/setup-buildx-action@v4, docker/login-action@v4, actions/upload-artifact@v7, actions/download-artifact@v8, actions-rust-lang/setup-rust-toolchain@v1, actions-rust-lang/rustfmt@v1, docker/setup-buildx-action@v4.1.0.

Locations:

- `.github/workflows/build-binaries.yml:24`
- `.github/workflows/build-binaries.yml:27`
- `.github/workflows/build-binaries.yml:55`
- `.github/workflows/build-binaries.yml:62`
- `.github/workflows/build-binaries.yml:68`
- `.github/workflows/build.yml:12`
- `.github/workflows/build.yml:22`
- `.github/workflows/ci.yml:12`
- `.github/workflows/ci.yml:14`
- `.github/workflows/ci.yml:17`
- `.github/workflows/ci.yml:25`
- `.github/workflows/ci.yml:26`
- `.github/workflows/release.yml:31`
- `.github/workflows/release.yml:50`
- `.github/workflows/release.yml:57`
- `.github/workflows/release.yml:62`
- `.github/workflows/release.yml:68`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, broad-permissions, missing-permissions, unpinned-uses

**Notes:**

Fixed all findings across 4 workflow files:

1. script-injection (build-binaries.yml): Moved ${{ inputs.version }} and ${{ inputs.cache-ref }} from run: blocks into env: blocks (INPUT_VERSION, INPUT_CACHE_REF), referenced as plain shell variables.

2. script-injection (release.yml): Moved ${{ github.event.inputs.semver }} into env: blocks (INPUT_SEMVER/CACHE_SERVER_VERSION) and ${{ github.repository }} into env: block as GH_REPO (used natively by gh CLI).

3. broad-permissions (release.yml): Replaced top-level 'write-all' with permissions: {} and added minimal job-level permissions (contents: write, packages: write) only for the release job.

4. missing-permissions (build-binaries.yml, build.yml, ci.yml): Added permissions: {} at top-level and job-level for all jobs.

5. unpinned-uses: Pinned all 8 distinct action references to full 40-char SHA hashes with tag comments: actions/checkout@df4cb1c, docker/setup-buildx-action@bb05f3f (v4) and @d7f5e7f (v4.1.0), docker/login-action@af1e73f, actions/upload-artifact@043fb46, actions/download-artifact@3e5f45b, actions-rust-lang/setup-rust-toolchain@166cdcf, actions-rust-lang/rustfmt@4066006.

### Iteration 2

**Fixes applied:** hardcoded-credentials

**Notes:**

Replaced the hardcoded literal value `some-secret-key` for `S3_SECRET_KEY` in `.github/workflows/ci.yml` (line 33) with the GitHub Actions secret expression `${{ secrets.S3_SECRET_KEY }}`. The other environment variables in the same block (S3_ACCESS_KEY, S3_REGION, S3_BUCKET_NAME, S3_ENDPOINT) do not contain 'secret' in their names and appear to be non-sensitive test configuration values, so they were left as-is.

### Iteration 3

**Fixes applied:** hardcoded-credentials

**Notes:**

Replaced the hardcoded literal value `some-access-key` for `S3_ACCESS_KEY` in `.github/workflows/ci.yml` (line 34) with `${{ secrets.S3_ACCESS_KEY }}`, referencing a GitHub Actions secret instead. This is now consistent with how `S3_SECRET_KEY` is handled on the next line.

