<!-- markdownlint-disable -->

# Hardening Report: brunojppb--turbo-cache-server/4.0.14

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **brunojppb--turbo-cache-server/4.0.14** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks interpolate ${{ inputs.* }} and ${{ github.event.inputs.* }} expressions directly into shell command strings (rule a), allowing an attacker-controlled value to be executed as shell code. Additionally, several env vars holding workflow-controlled data are expanded unquoted (rule b).

In build-binaries.yml:
- `sed -i "s/^version = \".*\"/version = \"${{ inputs.version }}\"/" Cargo.toml` (line ~30, rule a)
- `--tag brunojppb/turbo-cache-server-build:${{ inputs.version }}` inside docker buildx build run block (rule a)
- `--cache-to type=registry,ref=brunojppb/turbo-cache-server-build:${{ inputs.cache-ref }}` (rule a)
- `docker pull brunojppb/turbo-cache-server-build:${{ inputs.version }}` (rule a)
- `docker create ... brunojppb/turbo-cache-server-build:${{ inputs.version }}` (rule a)

In release.yml:
- `sed -i "s/^version = \".*\"/version = \"${{ github.event.inputs.semver }}\"/" Cargo.toml` (rule a)
- `--tag ghcr.io/brunojppb/turbo-cache-server:$CACHE_SERVER_VERSION` — unquoted expansion of inputs.semver (rule b)
- `git tag -a ${CACHE_SERVER_VERSION}` — unquoted expansion of inputs.semver (rule b)
- `git push origin $CACHE_SERVER_VERSION` — unquoted expansion of inputs.semver (rule b)
- `echo "**Full Changelog**: https://github.com/${{ github.repository }}/compare/..."` — direct ${{ github.repository }} in run: (rule a)
- `DOWNLOAD_URL="https://github.com/${{ github.repository }}/releases/download/..."` — direct ${{ github.repository }} in run: (rule a)

Locations:

- `.github/workflows/build-binaries.yml:30`
- `.github/workflows/build-binaries.yml:43`
- `.github/workflows/build-binaries.yml:44`
- `.github/workflows/build-binaries.yml:49`
- `.github/workflows/build-binaries.yml:50`
- `.github/workflows/release.yml:34`
- `.github/workflows/release.yml:64`
- `.github/workflows/release.yml:73`
- `.github/workflows/release.yml:78`
- `.github/workflows/release.yml:107`
- `.github/workflows/release.yml:120`

### unpinned-uses (severity: high)

All uses: references across all workflow files use mutable version tags (e.g. @v7, @v4, @v1, @v8, @v4.2.0) instead of immutable 40-character SHA commit hashes. This exposes the workflows to supply-chain attacks if any referenced action is compromised or its tag is moved.

Failing references include:
- actions/checkout@v7 (build-binaries.yml, build.yml, ci.yml, release.yml)
- docker/setup-buildx-action@v4 (build-binaries.yml, release.yml)
- docker/login-action@v4 (build-binaries.yml, release.yml)
- actions/upload-artifact@v7 (build-binaries.yml, build.yml)
- actions-rust-lang/setup-rust-toolchain@v1 (ci.yml)
- actions-rust-lang/rustfmt@v1 (ci.yml)
- actions/download-artifact@v8 (build.yml, release.yml)
- docker/setup-buildx-action@v4.2.0 (release.yml)

Locations:

- `.github/workflows/build-binaries.yml:24`
- `.github/workflows/build-binaries.yml:27`
- `.github/workflows/build-binaries.yml:37`
- `.github/workflows/build-binaries.yml:62`
- `.github/workflows/build-binaries.yml:65`
- `.github/workflows/build-binaries.yml:75`
- `.github/workflows/build-binaries.yml:85`
- `.github/workflows/build.yml:12`
- `.github/workflows/build.yml:18`
- `.github/workflows/ci.yml:12`
- `.github/workflows/ci.yml:14`
- `.github/workflows/ci.yml:15`
- `.github/workflows/ci.yml:22`
- `.github/workflows/ci.yml:23`
- `.github/workflows/release.yml:30`
- `.github/workflows/release.yml:40`
- `.github/workflows/release.yml:47`
- `.github/workflows/release.yml:51`
- `.github/workflows/release.yml:56`

### broad-permissions (severity: medium)

release.yml sets top-level `permissions: "write-all"`, granting every possible write permission to the GITHUB_TOKEN. This is overly broad and violates the principle of least privilege. It should be replaced with specific minimal permissions (e.g. contents: write, packages: write).

Locations:

- `.github/workflows/release.yml:13`

### missing-permissions (severity: medium)

build-binaries.yml, build.yml, and ci.yml have no top-level permissions: key and no job-level permissions: keys on any of their jobs. Without explicit permissions, the GITHUB_TOKEN inherits the repository's default permissions, which may be broader than necessary.

Locations:

- `.github/workflows/build-binaries.yml:1`
- `.github/workflows/build.yml:1`
- `.github/workflows/ci.yml:1`

### hardcoded-credentials (severity: high)

ci.yml contains literal hardcoded credential values for S3_ACCESS_KEY and S3_SECRET_KEY in the test job's env block: `S3_ACCESS_KEY: some-access-key` and `S3_SECRET_KEY: some-secret-key`. Even though these appear to be placeholder/test values, they are hardcoded literals rather than GitHub Actions secret expressions (e.g. ${{ secrets.S3_ACCESS_KEY }}), violating the hardcoded-credentials check.

Locations:

- `.github/workflows/ci.yml:29`
- `.github/workflows/ci.yml:30`

### github-env-injection (severity: high)

Several run: blocks write values derived from untrusted inputs to $GITHUB_OUTPUT without the required sanitization step (printf '%s' ... | tr -d '\n\r').

1. In build-binaries.yml (both build-linux and build-macos jobs), the 'Read Rust version' step writes a value derived from file content to $GITHUB_OUTPUT: `echo "version=$(grep -m1 '^channel' rust-toolchain.toml | cut -d'"' -f2)" >> "$GITHUB_OUTPUT"`. The output of this step (steps.rust.outputs.version) is then used unsanitized in env: blocks.

2. In release.yml, the same 'Read Rust version' step writes to $GITHUB_OUTPUT without sanitization. Additionally, the 'Update version on Cargo.toml' step writes ${{ github.event.inputs.semver }} directly into a sed command that modifies a file, and the 'Create GitHub Release' step embeds ${{ github.repository }} directly in a run: block writing to a notes file via echo >> $NOTES_FILE.

Locations:

- `.github/workflows/build-binaries.yml:38`
- `.github/workflows/build-binaries.yml:76`
- `.github/workflows/release.yml:60`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, broad-permissions, missing-permissions, hardcoded-credentials, github-env-injection

**Notes:**

Fixed all 6 findings across 4 workflow files:

1. script-injection: Moved all ${{ inputs.* }}, ${{ github.event.inputs.* }}, and ${{ github.repository }} expressions from run: blocks into env: blocks; referenced as plain env vars in shell. Quoted all unquoted env var expansions (e.g. git tag -a "${CACHE_SERVER_VERSION}", git push origin "${CACHE_SERVER_VERSION}", docker tag with quoted version).

2. unpinned-uses: Pinned all 8 action references to full 40-char SHA hashes with tag comments preserved.

3. broad-permissions: Replaced permissions: "write-all" in release.yml with contents: write and packages: write.

4. missing-permissions: Added permissions: contents: read to build-binaries.yml, build.yml, and ci.yml.

5. hardcoded-credentials: Replaced literal 'some-access-key' and 'some-secret-key' in ci.yml with ${{ secrets.S3_ACCESS_KEY }} and ${{ secrets.S3_SECRET_KEY }}.

6. github-env-injection: Sanitized all GITHUB_OUTPUT writes in 'Read Rust version' steps using printf '%s' "$raw" | tr -d '\n\r' before writing.

