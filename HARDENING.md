<!-- markdownlint-disable -->

# Hardening Report: actions--setup-go/v6.4.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **actions--setup-go/v6.4.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference external actions using mutable tags or branch names instead of pinned 40-character SHA digests, making them vulnerable to supply-chain attacks.

Failing references:
- basic-validation.yml: `uses: actions/reusable-workflows/.github/workflows/basic-validation.yml@main`
- check-dist.yml: `uses: actions/reusable-workflows/.github/workflows/check-dist.yml@main`
- codeql-analysis.yml: `uses: actions/reusable-workflows/.github/workflows/codeql-analysis.yml@main`
- licensed.yml: `uses: actions/reusable-workflows/.github/workflows/licensed.yml@main`
- microsoft-validation.yml: `uses: actions/checkout@v6` (×4 occurrences)
- publish-immutable-actions.yml: `uses: actions/checkout@v6`, `uses: actions/publish-immutable-action@v0.0.4`
- release-new-action-version.yml: `uses: actions/publish-action@v0.4.0`
- update-config-files.yml: `uses: actions/reusable-workflows/.github/workflows/update-config-files.yml@main`
- versions.yml: `uses: actions/checkout@v6` (×12 occurrences)
- windows-validation.yml: `uses: actions/checkout@v6` (×3 occurrences)

Locations:

- `.github/workflows/basic-validation.yml:15`
- `.github/workflows/check-dist.yml:16`
- `.github/workflows/codeql-analysis.yml:13`
- `.github/workflows/licensed.yml:13`
- `.github/workflows/microsoft-validation.yml:19`
- `.github/workflows/publish-immutable-actions.yml:13`
- `.github/workflows/publish-immutable-actions.yml:16`
- `.github/workflows/release-new-action-version.yml:20`
- `.github/workflows/update-config-files.yml:13`
- `.github/workflows/versions.yml:23`
- `.github/workflows/windows-validation.yml:22`

### missing-permissions (severity: medium)

Multiple workflow files have no top-level `permissions:` key and no job-level `permissions:` blocks on any job. Without explicit permissions, workflows inherit the default repository permissions (which may include write access), violating the principle of least privilege.

Locations:

- `.github/workflows/basic-validation.yml:1`
- `.github/workflows/check-dist.yml:1`
- `.github/workflows/codeql-analysis.yml:1`
- `.github/workflows/licensed.yml:1`
- `.github/workflows/microsoft-validation.yml:1`
- `.github/workflows/update-config-files.yml:1`
- `.github/workflows/versions.yml:1`
- `.github/workflows/windows-validation.yml:1`

### script-injection (severity: high)

Multiple `run:` blocks directly interpolate `${{ ... }}` expressions into shell commands (sub-rule a). Before the shell executes the command, GitHub Actions performs template substitution, allowing an attacker who controls the matrix or step output values to inject arbitrary shell commands.

In versions.yml, `${{ matrix.go }}` is interpolated directly into a shell command in three jobs:
- local-cache job: `run: __tests__/verify-go.sh ${{ matrix.go }}`
- setup-versions-from-manifest job: `run: __tests__/verify-go.sh ${{ matrix.go }}`
- setup-versions-from-dist job: `run: __tests__/verify-go.sh ${{ matrix.go }}`

In windows-validation.yml, `${{ matrix.go }}` and `${{ needs.find-default-go.outputs.version }}` are interpolated directly inside multi-line `run:` blocks, e.g.:
- `du -m -s 'C:\hostedtoolcache\windows\go\${{ matrix.go }}\x64'`
- `size=$(du -m -s 'C:\hostedtoolcache\windows\go\${{ needs.find-default-go.outputs.version }}\x64'|...)`
- `if [ -e 'D:\hostedtoolcache\windows\go\${{ needs.find-default-go.outputs.version }}\x64' ]`

Fix: move the values into `env:` variables and reference them as quoted shell variables (e.g. `"$GO_VERSION"`) instead of using `${{ }}` directly in the script.

Locations:

- `.github/workflows/versions.yml:75`
- `.github/workflows/versions.yml:152`
- `.github/workflows/versions.yml:171`
- `.github/workflows/windows-validation.yml:32`
- `.github/workflows/windows-validation.yml:43`
- `.github/workflows/windows-validation.yml:53`
- `.github/workflows/windows-validation.yml:80`
- `.github/workflows/windows-validation.yml:87`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings across 10 workflow files:

1. unpinned-uses: Pinned all mutable action references to full 40-char SHAs:
   - actions/reusable-workflows@main → @4735e71081024a944852f4ab9d1495b6dd2de8f2 (5 files)
   - actions/checkout@v6 → @df4cb1c069e1874edd31b4311f1884172cec0e10 (4 files, 20 total occurrences)
   - actions/publish-immutable-action@v0.0.4 → @4bc8754ffc40f27910afb20287dbbbb675a4e978
   - actions/publish-action@v0.4.0 → @23f4c6f12633a2da8f44938b71fde9afec138fb4

2. missing-permissions: Added `permissions: {}` top-level block to 8 workflow files (basic-validation.yml, check-dist.yml, codeql-analysis.yml, licensed.yml, microsoft-validation.yml, update-config-files.yml, versions.yml, windows-validation.yml).

3. script-injection: Moved ${{ matrix.go }} and ${{ needs.find-default-go.outputs.version }} expressions from run: blocks into env: blocks, referencing them as $GO_VERSION and $DEFAULT_GO_VERSION shell variables. Fixed in versions.yml (3 jobs) and windows-validation.yml (5 run blocks across 2 jobs).

