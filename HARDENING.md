<!-- markdownlint-disable -->

# Hardening Report: actions--setup-go/v7.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **actions--setup-go/v7.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files use mutable tag or branch refs instead of pinned full-length SHA commits. Affected references include: actions/reusable-workflows/...@main (basic-validation.yml, check-dist.yml, codeql-analysis.yml, licensed.yml, update-config-files.yml), actions/checkout@v6 (microsoft-validation.yml, versions.yml, windows-validation.yml), actions/checkout@v6 (publish-immutable-actions.yml), actions/publish-immutable-action@v0.0.4 (publish-immutable-actions.yml), actions/publish-action@v0.4.0 (release-new-action-version.yml). Any of these mutable refs could be updated to point to malicious code.

Locations:

- `.github/workflows/basic-validation.yml:13`
- `.github/workflows/check-dist.yml:14`
- `.github/workflows/codeql-analysis.yml:13`
- `.github/workflows/licensed.yml:13`
- `.github/workflows/microsoft-validation.yml:19`
- `.github/workflows/publish-immutable-actions.yml:14`
- `.github/workflows/publish-immutable-actions.yml:17`
- `.github/workflows/release-new-action-version.yml:20`
- `.github/workflows/update-config-files.yml:13`
- `.github/workflows/versions.yml:22`
- `.github/workflows/windows-validation.yml:22`

### script-injection (severity: high)

Rule (a) violation: GitHub Actions expressions are interpolated directly inside run: shell command strings. In versions.yml, `run: __tests__/verify-go.sh ${{ matrix.go }}` passes a matrix value directly as a shell argument without quoting or env-var indirection. In windows-validation.yml, `${{ matrix.go }}` and `${{ needs.find-default-go.outputs.version }}` are interpolated directly inside multi-line run: blocks (e.g., inside du, path comparisons, and conditional checks). These values flow through YAML template substitution before the shell sees them, enabling command injection if the matrix or needs output values contain shell metacharacters.

Locations:

- `.github/workflows/versions.yml:83`
- `.github/workflows/versions.yml:157`
- `.github/workflows/versions.yml:173`
- `.github/workflows/windows-validation.yml:30`
- `.github/workflows/windows-validation.yml:32`
- `.github/workflows/windows-validation.yml:40`
- `.github/workflows/windows-validation.yml:41`
- `.github/workflows/windows-validation.yml:51`
- `.github/workflows/windows-validation.yml:53`
- `.github/workflows/windows-validation.yml:57`
- `.github/workflows/windows-validation.yml:59`
- `.github/workflows/windows-validation.yml:88`
- `.github/workflows/windows-validation.yml:97`
- `.github/workflows/windows-validation.yml:103`

### missing-permissions (severity: medium)

These workflow files have no top-level `permissions:` key and no job-level `permissions:` keys on any of their jobs. Without explicit permissions, workflows inherit the repository's default token permissions (which may be broad write-all). Explicit minimal permissions should be declared.

Locations:

- `.github/workflows/basic-validation.yml:1`
- `.github/workflows/check-dist.yml:1`
- `.github/workflows/codeql-analysis.yml:1`
- `.github/workflows/licensed.yml:1`
- `.github/workflows/microsoft-validation.yml:1`
- `.github/workflows/update-config-files.yml:1`
- `.github/workflows/versions.yml:1`
- `.github/workflows/windows-validation.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three findings across 10 workflow files:

1. unpinned-uses: Pinned all mutable refs to full SHA commits:
   - actions/reusable-workflows@main → @d468c63c53c1184242904d1a3ac74fd1081f36c8 (5 files)
   - actions/checkout@v6 → @d23441a48e516b6c34aea4fa41551a30e30af803 (4 files)
   - actions/publish-immutable-action@v0.0.4 → @4bc8754ffc40f27910afb20287dbbbb675a4e978
   - actions/publish-action@v0.4.0 → @23f4c6f12633a2da8f44938b71fde9afec138fb4

2. script-injection: Moved all ${{ }} expressions out of run: blocks into env: blocks:
   - versions.yml: matrix.go → MATRIX_GO env var in local-cache, setup-versions-from-manifest, setup-versions-from-dist jobs
   - windows-validation.yml: matrix.go → MATRIX_GO and needs.find-default-go.outputs.version → DEFAULT_GO_VERSION in all affected steps

3. missing-permissions: Added 'permissions: contents: read' top-level block to basic-validation.yml, check-dist.yml, codeql-analysis.yml, licensed.yml, microsoft-validation.yml, update-config-files.yml, versions.yml, and windows-validation.yml

