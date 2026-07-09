<!-- markdownlint-disable -->

# Hardening Report: actions--setup-go/v5.6.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **actions--setup-go/v5.6.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags or branch names instead of pinned full-length SHA commit hashes. Failing references include: basic-validation.yml uses actions/reusable-workflows/...@main; check-dist.yml uses actions/reusable-workflows/...@main; codeql-analysis.yml uses actions/reusable-workflows/...@main; licensed.yml uses actions/reusable-workflows/...@main; update-config-files.yml uses actions/reusable-workflows/...@main; publish-immutable-actions.yml uses actions/checkout@v4 and actions/publish-immutable-action@v0.0.4; release-new-action-version.yml uses actions/publish-action@v0.3.0; versions.yml uses actions/checkout@v4; windows-validation.yml uses actions/checkout@v4.

Locations:

- `.github/workflows/basic-validation.yml:14`
- `.github/workflows/check-dist.yml:15`
- `.github/workflows/codeql-analysis.yml:12`
- `.github/workflows/licensed.yml:13`
- `.github/workflows/publish-immutable-actions.yml:14`
- `.github/workflows/release-new-action-version.yml:21`
- `.github/workflows/update-config-files.yml:11`
- `.github/workflows/versions.yml:22`
- `.github/workflows/windows-validation.yml:23`

### permissions (severity: medium)

missing-permissions: These workflow files have no top-level 'permissions:' key and no job-level 'permissions:' key on any job, granting the default (potentially broad) token permissions: basic-validation.yml, check-dist.yml, codeql-analysis.yml, licensed.yml, update-config-files.yml, versions.yml, windows-validation.yml.

Locations:

- `.github/workflows/basic-validation.yml:1`
- `.github/workflows/check-dist.yml:1`
- `.github/workflows/codeql-analysis.yml:1`
- `.github/workflows/licensed.yml:1`
- `.github/workflows/update-config-files.yml:1`
- `.github/workflows/versions.yml:1`
- `.github/workflows/windows-validation.yml:1`

### script-injection (severity: high)

Rule (a): GitHub Actions expressions are interpolated directly inside run: shell command strings. In versions.yml, '${{ matrix.go }}' is used unquoted directly in run: commands (e.g., 'run: __tests__/verify-go.sh ${{ matrix.go }}'). In windows-validation.yml, '${{ matrix.go }}' and '${{ needs.find-default-go.outputs.version }}' are interpolated directly into shell commands inside run: blocks (e.g., 'du -m -s C:\hostedtoolcache\windows\go\${{ matrix.go }}\x64' and path comparisons with ${{ needs.find-default-go.outputs.version }}). Any ${{ ... }} expression inside a run: block is a script-injection risk as the value is substituted before the shell parses it.

Locations:

- `.github/workflows/versions.yml:72`
- `.github/workflows/versions.yml:112`
- `.github/workflows/versions.yml:131`
- `.github/workflows/windows-validation.yml:33`
- `.github/workflows/windows-validation.yml:42`
- `.github/workflows/windows-validation.yml:52`
- `.github/workflows/windows-validation.yml:88`
- `.github/workflows/windows-validation.yml:96`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, permissions, script-injection

**Notes:**

Fixed all three findings across 9 workflow files:

1. **unpinned-uses**: Pinned all action references to full SHA commits:
   - actions/reusable-workflows@main → @4735e71081024a944852f4ab9d1495b6dd2de8f2 (in basic-validation.yml, check-dist.yml, codeql-analysis.yml, licensed.yml, update-config-files.yml)
   - actions/checkout@v4 → @34e114876b0b11c390a56381ad16ebd13914f8d5 (in publish-immutable-actions.yml, versions.yml, windows-validation.yml)
   - actions/publish-immutable-action@v0.0.4 → @4bc8754ffc40f27910afb20287dbbbb675a4e978 (in publish-immutable-actions.yml)
   - actions/publish-action@v0.3.0 → @f784495ce78a41bac4ed7e34a73f0034015764bb (in release-new-action-version.yml)

2. **permissions**: Added `permissions: {}` top-level block to all 7 workflow files that were missing it: basic-validation.yml, check-dist.yml, codeql-analysis.yml, licensed.yml, update-config-files.yml, versions.yml, windows-validation.yml. The publish-immutable-actions.yml and release-new-action-version.yml already had appropriate permissions.

3. **script-injection**: Fixed all ${{ }} expressions in run: blocks:
   - versions.yml: Moved `${{ matrix.go }}` to `env: MATRIX_GO:` in local-cache, setup-versions-from-manifest, and setup-versions-from-dist jobs; shell scripts now use `"$MATRIX_GO"` instead
   - windows-validation.yml: Moved `${{ matrix.go }}` to `env: MATRIX_GO:` and `${{ needs.find-default-go.outputs.version }}` to `env: DEFAULT_GO_VERSION:` in all affected steps; shell scripts now use `${MATRIX_GO}` and `${DEFAULT_GO_VERSION}` instead

