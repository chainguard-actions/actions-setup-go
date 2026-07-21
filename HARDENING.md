<!-- markdownlint-disable -->

# Hardening Report: actions--setup-go/v5.6.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **actions--setup-go/v5.6.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags or branch names instead of pinned 40-character commit SHAs, making them vulnerable to supply-chain attacks. Failing references: basic-validation.yml: `actions/reusable-workflows/.github/workflows/basic-validation.yml@main`; check-dist.yml: `actions/reusable-workflows/.github/workflows/check-dist.yml@main`; codeql-analysis.yml: `actions/reusable-workflows/.github/workflows/codeql-analysis.yml@main`; licensed.yml: `actions/reusable-workflows/.github/workflows/licensed.yml@main`; update-config-files.yml: `actions/reusable-workflows/.github/workflows/update-config-files.yml@main`; publish-immutable-actions.yml: `actions/checkout@v4`, `actions/publish-immutable-action@v0.0.4`; release-new-action-version.yml: `actions/publish-action@v0.3.0`; versions.yml: `actions/checkout@v4`; windows-validation.yml: `actions/checkout@v4`.

Locations:

- `.github/workflows/basic-validation.yml:13`
- `.github/workflows/check-dist.yml:13`
- `.github/workflows/codeql-analysis.yml:12`
- `.github/workflows/licensed.yml:12`
- `.github/workflows/update-config-files.yml:12`
- `.github/workflows/publish-immutable-actions.yml:15`
- `.github/workflows/publish-immutable-actions.yml:17`
- `.github/workflows/release-new-action-version.yml:22`
- `.github/workflows/versions.yml:23`
- `.github/workflows/windows-validation.yml:22`

### script-injection (severity: high)

GitHub Actions expressions are interpolated directly inside run: shell command strings (sub-rule a), allowing template substitution to inject arbitrary shell content before the shell parses the command. In versions.yml, `${{ matrix.go }}` is passed unquoted as a positional argument to a shell script in three jobs: `run: __tests__/verify-go.sh ${{ matrix.go }}`. In windows-validation.yml, `${{ matrix.go }}` is embedded directly inside multi-line run: blocks (e.g. `du -m -s 'C:\hostedtoolcache\windows\go\${{ matrix.go }}\x64'`, `if [ $(which go) != '/c/hostedtoolcache/windows/go/${{ matrix.go }}/x64/bin/go' ]`), and `${{ needs.find-default-go.outputs.version }}` is similarly embedded in run: blocks (e.g. `du -m -s 'C:\hostedtoolcache\windows\go\${{ needs.find-default-go.outputs.version }}\x64'`). All of these are direct expression interpolations inside run: scripts.

Locations:

- `.github/workflows/versions.yml:91`
- `.github/workflows/versions.yml:148`
- `.github/workflows/versions.yml:164`
- `.github/workflows/windows-validation.yml:32`
- `.github/workflows/windows-validation.yml:34`
- `.github/workflows/windows-validation.yml:44`
- `.github/workflows/windows-validation.yml:45`
- `.github/workflows/windows-validation.yml:60`
- `.github/workflows/windows-validation.yml:64`
- `.github/workflows/windows-validation.yml:104`
- `.github/workflows/windows-validation.yml:113`

### missing-permissions (severity: medium)

Seven workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs. Without explicit permissions, workflows inherit the default repository token permissions (which may be broad), violating the principle of least privilege. Affected files: basic-validation.yml, check-dist.yml, codeql-analysis.yml, licensed.yml, update-config-files.yml, versions.yml, windows-validation.yml.

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

1. unpinned-uses: Pinned all action references to full 40-char SHAs: actions/reusable-workflows@main→4735e71081024a944852f4ab9d1495b6dd2de8f2, actions/checkout@v4→11d5960a326750d5838078e36cf38b85af677262, actions/publish-immutable-action@v0.0.4→4bc8754ffc40f27910afb20287dbbbb675a4e978, actions/publish-action@v0.3.0→f784495ce78a41bac4ed7e34a73f0034015764bb.

2. script-injection: Moved all ${{ matrix.go }} and ${{ needs.find-default-go.outputs.version }} expressions out of run: shell scripts into step env: blocks (MATRIX_GO and DEFAULT_GO_VERSION respectively), then referenced them as plain shell variables.

3. missing-permissions: Added permissions: contents: read to versions.yml and windows-validation.yml (which use checkout), and permissions: {} to basic-validation.yml, check-dist.yml, codeql-analysis.yml, licensed.yml, and update-config-files.yml (reusable workflow callers needing no permissions).

