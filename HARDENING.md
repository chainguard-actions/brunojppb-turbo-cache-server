<!-- markdownlint-disable -->

# Hardening Report: brunojppb--turbo-cache-server/4.0.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **brunojppb--turbo-cache-server/4.0.3** was hardened automatically. 5 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference GitHub Actions using mutable version tags instead of pinned full 40-character SHA commit hashes, making the workflows vulnerable to supply-chain attacks if the tag is moved.

build-binaries.yml: actions/checkout@v6, docker/setup-buildx-action@v4, docker/login-action@v4, actions/upload-artifact@v7 (×2)
ci.yml: actions/checkout@v6 (×2), actions-rust-lang/setup-rust-toolchain@v1 (×2), actions-rust-lang/rustfmt@v1
build.yml: actions/download-artifact@v8, actions/upload-artifact@v7
release.yml: actions/checkout@v6, actions/download-artifact@v8, docker/setup-buildx-action@v4.0.0, docker/login-action@v4 (×2)

Locations:

- `.github/workflows/build-binaries.yml:25`
- `.github/workflows/build-binaries.yml:28`
- `.github/workflows/build-binaries.yml:35`
- `.github/workflows/build-binaries.yml:58`
- `.github/workflows/build-binaries.yml:65`
- `.github/workflows/build-binaries.yml:68`
- `.github/workflows/build-binaries.yml:80`
- `.github/workflows/ci.yml:12`
- `.github/workflows/ci.yml:14`
- `.github/workflows/ci.yml:17`
- `.github/workflows/ci.yml:24`
- `.github/workflows/ci.yml:25`
- `.github/workflows/build.yml:14`
- `.github/workflows/build.yml:20`
- `.github/workflows/release.yml:38`
- `.github/workflows/release.yml:47`
- `.github/workflows/release.yml:54`
- `.github/workflows/release.yml:57`
- `.github/workflows/release.yml:63`

### script-injection (severity: high)

Multiple run: blocks directly interpolate GitHub Actions expressions (${{ ... }}) inside shell command strings (rule a), allowing an attacker-controlled value to be interpreted as shell syntax before the shell ever sees it.

build-binaries.yml:
- 'Update version on Cargo.toml' step (build-linux): sed -i "...version = \"${{ inputs.version }}\"/" Cargo.toml
- 'build binary' step: docker buildx build ... --tag ...build:${{ inputs.version }} ... --cache-to ...ref=...build:${{ inputs.cache-ref }} ...
- 'create temp containers' step: docker pull ...build:${{ inputs.version }}; docker create ... ...build:${{ inputs.version }} (×2)
- 'Update version on Cargo.toml' step (build-macos): sed -i "...version = \"${{ inputs.version }}\"/" Cargo.toml

release.yml:
- 'Update version on Cargo.toml' step: sed -i "...version = \"${{ github.event.inputs.semver }}\"/" Cargo.toml
- 'Create GitHub Release' step: echo "**Full Changelog**: https://github.com/${{ github.repository }}/compare/..." >> "$NOTES_FILE"

Locations:

- `.github/workflows/build-binaries.yml:32`
- `.github/workflows/build-binaries.yml:42`
- `.github/workflows/build-binaries.yml:46`
- `.github/workflows/build-binaries.yml:72`
- `.github/workflows/release.yml:45`
- `.github/workflows/release.yml:92`

### broad-permissions (severity: medium)

release.yml sets top-level permissions to "write-all", granting every job in the workflow write access to all repository scopes. This violates the principle of least privilege and should be replaced with specific minimal permissions per job.

Locations:

- `.github/workflows/release.yml:13`

### missing-permissions (severity: medium)

The following workflow files have no top-level permissions: key and no per-job permissions: keys, meaning they inherit the default repository token permissions (which may be write-all depending on repository settings).

- build-binaries.yml: no permissions key at file or job level
- build.yml: no permissions key at file or job level
- ci.yml: no permissions key at file or job level

Locations:

- `.github/workflows/build-binaries.yml:1`
- `.github/workflows/build.yml:1`
- `.github/workflows/ci.yml:1`

### hardcoded-credentials (severity: high)

ci.yml contains hardcoded literal values assigned to environment variable names containing 'secret' and 'key': S3_ACCESS_KEY is set to 'some-access-key' and S3_SECRET_KEY is set to 'some-secret-key'. Even though these appear to be placeholder/test values for a local mock S3 endpoint (localhost:9000), they are literal non-expression credential assignments and match the hardcoded-credentials pattern.

Locations:

- `.github/workflows/ci.yml:31`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, broad-permissions, missing-permissions, hardcoded-credentials

**Notes:**

Fixed all 5 findings across 4 workflow files:

1. unpinned-uses: Pinned all action references to full SHA hashes with tag comments:
   - actions/checkout@v6 → @df4cb1c069e1874edd31b4311f1884172cec0e10
   - docker/setup-buildx-action@v4 → @bb05f3f5519dd87d3ba754cc423b652a5edd6d2c
   - docker/setup-buildx-action@v4.0.0 → @4d04d5d9486b7bd6fa91e7baf45bbb4f8b9deedd
   - docker/login-action@v4 → @af1e73f918a031802d376d3c8bbc3fe56130a9b0
   - actions/upload-artifact@v7 → @043fb46d1a93c77aae656e7c1c64a875d1fc6a0a
   - actions/download-artifact@v8 → @3e5f45b2cfb9172054b4087a40e8e0b5a5461e7c
   - actions-rust-lang/setup-rust-toolchain@v1 → @166cdcfd11aee3cb47222f9ddb555ce30ddb9659
   - actions-rust-lang/rustfmt@v1 → @4066006ec54a31931b9b1fddfd38f2fdf2d27143

2. script-injection: Moved all ${{ }} expressions from run: blocks into env: blocks, referencing them as plain shell variables.

3. broad-permissions: Replaced release.yml's top-level 'write-all' with per-job minimal permissions (contents: read for sanity-check; contents: write + packages: write for release job).

4. missing-permissions: Added 'permissions: contents: read' to build-binaries.yml, build.yml, and ci.yml.

5. hardcoded-credentials: Replaced literal 'some-access-key' and 'some-secret-key' in ci.yml with ${{ secrets.S3_ACCESS_KEY }} and ${{ secrets.S3_SECRET_KEY }} references.

### Iteration 2

**Fixes applied:** missing-permissions

**Notes:**

Added `permissions: contents: read` to the `build` job in `.github/workflows/release.yml` (line 24). The `build` job calls the reusable workflow `.github/workflows/build-binaries.yml`, which itself declares `permissions: contents: read`. Without an explicit permissions block on the caller job, it would inherit the default (potentially broad) repository permissions. The fix grants only the minimum `contents: read` permission needed for the reusable workflow to check out the repository.

