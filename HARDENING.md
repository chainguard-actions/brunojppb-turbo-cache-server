<!-- markdownlint-disable -->

# Hardening Report: brunojppb--turbo-cache-server/4.0.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **brunojppb--turbo-cache-server/4.0.4** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks directly interpolate ${{ inputs.version }}, ${{ inputs.cache-ref }}, and ${{ github.event.inputs.semver }} expressions into shell commands (sub-rule a). These values are user-controlled via workflow_dispatch or workflow_call inputs and are substituted into the shell string before the shell parses it, enabling command injection. Offending lines include: `sed -i "...version = \"${{ inputs.version }}\"..." Cargo.toml`, `docker buildx build ... :${{ inputs.version }} ... :${{ inputs.cache-ref }} ...`, `docker pull ...:${{ inputs.version }}`, `docker create ... :${{ inputs.version }}`, and `sed -i "...version = \"${{ github.event.inputs.semver }}\"..." Cargo.toml`.

Locations:

- `.github/workflows/build-binaries.yml:33`
- `.github/workflows/build-binaries.yml:42`
- `.github/workflows/build-binaries.yml:43`
- `.github/workflows/build-binaries.yml:47`
- `.github/workflows/build-binaries.yml:48`
- `.github/workflows/build-binaries.yml:72`
- `.github/workflows/release.yml:37`

### unpinned-uses (severity: high)

All workflow files reference GitHub Actions using mutable tags instead of pinned 40-character SHA commit hashes, making the workflows vulnerable to supply-chain attacks if any referenced action is compromised or its tag is moved. Unpinned references found: actions/checkout@v6, docker/setup-buildx-action@v4, docker/login-action@v4, actions/upload-artifact@v7, actions/download-artifact@v8, actions-rust-lang/setup-rust-toolchain@v1, actions-rust-lang/rustfmt@v1, docker/setup-buildx-action@v4.0.0.

Locations:

- `.github/workflows/build-binaries.yml:24`
- `.github/workflows/build-binaries.yml:27`
- `.github/workflows/build-binaries.yml:37`
- `.github/workflows/build-binaries.yml:55`
- `.github/workflows/build-binaries.yml:63`
- `.github/workflows/build-binaries.yml:67`
- `.github/workflows/build-binaries.yml:82`
- `.github/workflows/build.yml:14`
- `.github/workflows/build.yml:22`
- `.github/workflows/ci.yml:13`
- `.github/workflows/ci.yml:15`
- `.github/workflows/ci.yml:18`
- `.github/workflows/ci.yml:26`
- `.github/workflows/ci.yml:27`
- `.github/workflows/release.yml:24`
- `.github/workflows/release.yml:42`
- `.github/workflows/release.yml:49`
- `.github/workflows/release.yml:53`
- `.github/workflows/release.yml:59`

### broad-permissions (severity: medium)

release.yml sets top-level `permissions: "write-all"`, granting all available permissions to every job in the workflow. This violates the principle of least privilege and should be replaced with specific minimal permissions (e.g., contents: write, packages: write) scoped only to the jobs that need them.

Locations:

- `.github/workflows/release.yml:13`

### missing-permissions (severity: medium)

build-binaries.yml, build.yml, and ci.yml have no top-level `permissions:` key and no job-level `permissions:` blocks on any of their jobs. Without explicit permissions, workflows inherit the repository's default token permissions (which may be read/write), violating the principle of least privilege.

Locations:

- `.github/workflows/build-binaries.yml:1`
- `.github/workflows/build.yml:1`
- `.github/workflows/ci.yml:1`

### hardcoded-credentials (severity: high)

ci.yml contains a literal hardcoded value assigned to `S3_SECRET_KEY: some-secret-key`. Even though this appears to be a placeholder test value, it is a non-expression literal string assigned to a name containing 'secret', which matches the hardcoded credentials pattern and should be replaced with a GitHub Actions secret expression (e.g., `${{ secrets.S3_SECRET_KEY }}`) or a clearly documented dummy value via a secrets manager.

Locations:

- `.github/workflows/ci.yml:33`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, broad-permissions, missing-permissions, hardcoded-credentials

**Notes:**

Fixed all 5 findings across 4 workflow files:

1. script-injection: Moved all ${{ inputs.version }}, ${{ inputs.cache-ref }}, and ${{ github.event.inputs.semver }} expressions out of run: shell strings into env: blocks, referencing them as plain shell variables (${INPUT_VERSION}, ${INPUT_CACHE_REF}, ${INPUT_SEMVER}). Also moved ${{ github.repository }} to env: in release.yml.

2. unpinned-uses: Pinned all 8 action references to full 40-char SHA hashes with tag comments: actions/checkout@df4cb1c, docker/setup-buildx-action@bb05f3f (v4) and @4d04d5d (v4.0.0), docker/login-action@af1e73f, actions/upload-artifact@043fb46, actions/download-artifact@3e5f45b, actions-rust-lang/setup-rust-toolchain@166cdcf, actions-rust-lang/rustfmt@4066006.

3. broad-permissions: Replaced permissions: "write-all" in release.yml with permissions: { contents: write, packages: write }.

4. missing-permissions: Added permissions: {} to build-binaries.yml, build.yml, and ci.yml.

5. hardcoded-credentials: Replaced literal S3_SECRET_KEY: some-secret-key in ci.yml with S3_SECRET_KEY: ${{ secrets.S3_SECRET_KEY }}.

