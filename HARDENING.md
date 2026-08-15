<!-- markdownlint-disable -->

# Hardening Report: actions--setup-go/v6.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **actions--setup-go/v6.3.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions and reusable workflows using mutable tags or branch names instead of pinned full-length SHA digests. This exposes the workflow to supply-chain attacks if the referenced tag or branch is updated with malicious code.

Failing references:
- basic-validation.yml: `actions/reusable-workflows/.github/workflows/basic-validation.yml@main`
- check-dist.yml: `actions/reusable-workflows/.github/workflows/check-dist.yml@main`
- codeql-analysis.yml: `actions/reusable-workflows/.github/workflows/codeql-analysis.yml@main`
- licensed.yml: `actions/reusable-workflows/.github/workflows/licensed.yml@main`
- update-config-files.yml: `actions/reusable-workflows/.github/workflows/update-config-files.yml@main`
- publish-immutable-actions.yml: `actions/checkout@v6`, `actions/publish-immutable-action@v0.0.4`
- release-new-action-version.yml: `actions/publish-action@v0.4.0`
- versions.yml: `actions/checkout@v6` (multiple steps)
- windows-validation.yml: `actions/checkout@v6` (multiple steps)

Locations:

- `.github/workflows/basic-validation.yml:14`
- `.github/workflows/check-dist.yml:14`
- `.github/workflows/codeql-analysis.yml:12`
- `.github/workflows/licensed.yml:13`
- `.github/workflows/update-config-files.yml:12`
- `.github/workflows/publish-immutable-actions.yml:13`
- `.github/workflows/publish-immutable-actions.yml:16`
- `.github/workflows/release-new-action-version.yml:20`
- `.github/workflows/versions.yml:23`
- `.github/workflows/windows-validation.yml:22`

### script-injection (severity: high)

Several run: blocks interpolate GitHub Actions expressions directly into shell commands (sub-rule a), allowing an attacker who controls the matrix or job output values to inject arbitrary shell commands.

In versions.yml, `${{ matrix.go }}` is interpolated directly into run: shell commands:
- `run: __tests__/verify-go.sh ${{ matrix.go }}` (local-cache job, line ~93)
- `run: __tests__/verify-go.sh ${{ matrix.go }}` (setup-versions-from-manifest job, line ~130)
- `run: __tests__/verify-go.sh ${{ matrix.go }}` (setup-versions-from-dist job, line ~148)

In windows-validation.yml, `${{ matrix.go }}` is interpolated directly into run: shell commands in path strings used with du and conditional checks (lines ~31, ~40, ~49), and `${{ needs.find-default-go.outputs.version }}` is interpolated directly into run: shell commands (lines ~80, ~87).

All these expressions are substituted by the GitHub Actions template engine before the shell ever sees the command, enabling shell metacharacter injection.

Locations:

- `.github/workflows/versions.yml:93`
- `.github/workflows/versions.yml:130`
- `.github/workflows/versions.yml:148`
- `.github/workflows/windows-validation.yml:31`
- `.github/workflows/windows-validation.yml:40`
- `.github/workflows/windows-validation.yml:49`
- `.github/workflows/windows-validation.yml:80`
- `.github/workflows/windows-validation.yml:87`

### missing-permissions (severity: medium)

The following workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs. Without explicit permissions, workflows run with the repository's default token permissions (which may be read/write), violating the principle of least privilege.

- basic-validation.yml: no permissions block at top level or job level
- check-dist.yml: no permissions block at top level or job level
- codeql-analysis.yml: no permissions block at top level or job level
- licensed.yml: no permissions block at top level or job level
- update-config-files.yml: no permissions block at top level or job level
- versions.yml: no permissions block at top level or job level
- windows-validation.yml: no permissions block at top level or job level

Locations:

- `.github/workflows/basic-validation.yml:1`
- `.github/workflows/check-dist.yml:1`
- `.github/workflows/codeql-analysis.yml:1`
- `.github/workflows/licensed.yml:1`
- `.github/workflows/update-config-files.yml:1`
- `.github/workflows/versions.yml:1`
- `.github/workflows/windows-validation.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three findings across 9 workflow files:

1. unpinned-uses: Pinned all mutable references to full SHA digests:
   - actions/reusable-workflows@main → @4735e71081024a944852f4ab9d1495b6dd2de8f2 (5 files)
   - actions/checkout@v6 → @d23441a48e516b6c34aea4fa41551a30e30af803 (versions.yml, windows-validation.yml, publish-immutable-actions.yml)
   - actions/publish-immutable-action@v0.0.4 → @4bc8754ffc40f27910afb20287dbbbb675a4e978
   - actions/publish-action@v0.4.0 → @23f4c6f12633a2da8f44938b71fde9afec138fb4

2. script-injection: Moved all ${{ matrix.go }} and ${{ needs.find-default-go.outputs.version }} expressions from run: shell commands into env: blocks, referencing them as $MATRIX_GO and $DEFAULT_GO_VERSION respectively in the shell scripts.

3. missing-permissions: Added `permissions: {}` top-level blocks to basic-validation.yml, check-dist.yml, codeql-analysis.yml, licensed.yml, update-config-files.yml, versions.yml, and windows-validation.yml.

