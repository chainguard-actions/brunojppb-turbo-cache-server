<!-- markdownlint-disable -->

# Hardening Report: brunojppb--turbo-cache-server/4.0.10

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **brunojppb--turbo-cache-server/4.0.10** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Direct ${{ }} expression interpolation inside run: shell command strings. In build-binaries.yml, ${{ inputs.version }} and ${{ inputs.cache-ref }} are interpolated directly into docker commands (e.g., `--tag brunojppb/turbo-cache-server-build:${{ inputs.version }}`), and ${{ inputs.version }} is interpolated into a sed command (`sed -i "s/^version = \".*\"/version = \"${{ inputs.version }}\"/" Cargo.toml`). In release.yml, ${{ github.event.inputs.semver }} is interpolated directly into a sed run: command, and ${{ github.repository }} is interpolated directly into an echo command inside a run: block. These template substitutions happen before the shell processes the string, enabling command injection by a malicious input value.

Locations:

- `.github/workflows/build-binaries.yml:33`
- `.github/workflows/build-binaries.yml:47`
- `.github/workflows/build-binaries.yml:52`
- `.github/workflows/build-binaries.yml:53`
- `.github/workflows/build-binaries.yml:75`
- `.github/workflows/build-binaries.yml:96`
- `.github/workflows/release.yml:36`
- `.github/workflows/release.yml:100`
- `.github/workflows/release.yml:113`

### unpinned-uses (severity: high)

All uses: references across all workflow files use mutable tag-based refs instead of pinned 40-character SHA digests, making the workflows vulnerable to supply-chain attacks if any referenced action is compromised or its tag is moved. Unpinned references include: actions/checkout@v7, docker/setup-buildx-action@v4, docker/login-action@v4, actions/upload-artifact@v7, actions/download-artifact@v8, actions-rust-lang/setup-rust-toolchain@v1, actions-rust-lang/rustfmt@v1, docker/setup-buildx-action@v4.2.0.

Locations:

- `.github/workflows/build-binaries.yml:24`
- `.github/workflows/build-binaries.yml:27`
- `.github/workflows/build-binaries.yml:30`
- `.github/workflows/build-binaries.yml:64`
- `.github/workflows/build-binaries.yml:67`
- `.github/workflows/build-binaries.yml:70`
- `.github/workflows/build-binaries.yml:85`
- `.github/workflows/build.yml:14`
- `.github/workflows/build.yml:22`
- `.github/workflows/ci.yml:12`
- `.github/workflows/ci.yml:14`
- `.github/workflows/ci.yml:15`
- `.github/workflows/ci.yml:22`
- `.github/workflows/ci.yml:23`
- `.github/workflows/release.yml:26`
- `.github/workflows/release.yml:44`
- `.github/workflows/release.yml:56`
- `.github/workflows/release.yml:61`
- `.github/workflows/release.yml:66`

### hardcoded-credentials (severity: high)

ci.yml contains literal hardcoded credential values: `S3_ACCESS_KEY: some-access-key` and `S3_SECRET_KEY: some-secret-key`. Even if these are placeholder/test values, they match the hardcoded-credentials pattern and should be replaced with GitHub Actions secret expressions (e.g., ${{ secrets.S3_ACCESS_KEY }}).

Locations:

- `.github/workflows/ci.yml:33`

### broad-permissions (severity: medium)

release.yml has a top-level `permissions: "write-all"` setting, which grants overly broad write access to all GitHub API scopes. This should be replaced with specific minimal permissions (e.g., contents: write, packages: write) required by the workflow.

Locations:

- `.github/workflows/release.yml:13`

### missing-permissions (severity: medium)

build-binaries.yml, build.yml, and ci.yml have no top-level permissions: key and no job-level permissions: blocks on any of their jobs. Without explicit permissions, workflows inherit the default repository permissions, which may be overly broad. Explicit minimal permissions should be declared.

Locations:

- `.github/workflows/build-binaries.yml:1`
- `.github/workflows/build.yml:1`
- `.github/workflows/ci.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, hardcoded-credentials, broad-permissions, missing-permissions

**Notes:**

Fixed all 5 findings across 4 workflow files:

1. **script-injection** (build-binaries.yml, release.yml): Moved all ${{ inputs.version }}, ${{ inputs.cache-ref }}, ${{ github.event.inputs.semver }}, and ${{ github.repository }} expressions out of run: shell strings into step env: blocks. Shell scripts now reference plain environment variables ($INPUT_VERSION, $INPUT_CACHE_REF, $SEMVER, $GH_REPO).

2. **unpinned-uses** (all 4 files): Pinned all action references to full 40-char SHAs:
   - actions/checkout@v7 → @9c091bb21b7c1c1d1991bb908d89e4e9dddfe3e0
   - docker/setup-buildx-action@v4/v4.2.0 → @bb05f3f5519dd87d3ba754cc423b652a5edd6d2c
   - docker/login-action@v4 → @af1e73f918a031802d376d3c8bbc3fe56130a9b0
   - actions/upload-artifact@v7 → @043fb46d1a93c77aae656e7c1c64a875d1fc6a0a
   - actions/download-artifact@v8 → @3e5f45b2cfb9172054b4087a40e8e0b5a5461e7c
   - actions-rust-lang/setup-rust-toolchain@v1 → @166cdcfd11aee3cb47222f9ddb555ce30ddb9659
   - actions-rust-lang/rustfmt@v1 → @4066006ec54a31931b9b1fddfd38f2fdf2d27143

3. **hardcoded-credentials** (ci.yml): Replaced literal 'some-access-key' and 'some-secret-key' with ${{ secrets.S3_ACCESS_KEY }} and ${{ secrets.S3_SECRET_KEY }}.

4. **broad-permissions** (release.yml): Replaced permissions: "write-all" with specific minimal permissions: contents: write, packages: write.

5. **missing-permissions** (build-binaries.yml, build.yml, ci.yml): Added permissions: contents: read to all three files.

