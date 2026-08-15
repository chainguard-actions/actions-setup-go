<!-- markdownlint-disable -->

# Hardening Report: actions--setup-go/v6.5.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **actions--setup-go/v6.5.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All workflow files use mutable tag or branch refs instead of pinned 40-character SHA commits, making them vulnerable to supply-chain attacks if the referenced action is compromised or a tag is moved. Affected refs include: actions/reusable-workflows/.github/workflows/basic-validation.yml@main, actions/checkout@v6, actions/reusable-workflows/.github/workflows/codeql-analysis.yml@main, actions/reusable-workflows/.github/workflows/licensed.yml@main, actions/publish-immutable-action@v0.0.4, actions/publish-action@v0.4.0, actions/reusable-workflows/.github/workflows/update-config-files.yml@main, and many more.

Locations:

- `.github/workflows/basic-validation.yml:14`
- `.github/workflows/check-dist.yml:14`
- `.github/workflows/codeql-analysis.yml:13`
- `.github/workflows/licensed.yml:13`
- `.github/workflows/microsoft-validation.yml:22`
- `.github/workflows/publish-immutable-actions.yml:12`
- `.github/workflows/publish-immutable-actions.yml:15`
- `.github/workflows/release-new-action-version.yml:22`
- `.github/workflows/update-config-files.yml:13`
- `.github/workflows/versions.yml:22`
- `.github/workflows/windows-validation.yml:18`

### script-injection (severity: high)

Multiple run: blocks directly interpolate GitHub Actions expressions (${{ ... }}) inside shell command strings, violating rule (a). In versions.yml, three steps pass ${{ matrix.go }} directly as a shell argument: `run: __tests__/verify-go.sh ${{ matrix.go }}`. In windows-validation.yml, multiple steps embed ${{ matrix.go }} and ${{ needs.find-default-go.outputs.version }} directly inside shell strings used in du, if-comparisons, and path checks (e.g. `du -m -s 'C:\hostedtoolcache\windows\go\${{ matrix.go }}\x64'`). These values flow through YAML template substitution before the shell sees them, allowing injection of shell metacharacters.

Locations:

- `.github/workflows/versions.yml:76`
- `.github/workflows/versions.yml:130`
- `.github/workflows/versions.yml:155`
- `.github/workflows/windows-validation.yml:30`
- `.github/workflows/windows-validation.yml:40`
- `.github/workflows/windows-validation.yml:55`
- `.github/workflows/windows-validation.yml:78`
- `.github/workflows/windows-validation.yml:88`

### missing-permissions (severity: medium)

Eight workflow files have no top-level permissions: key and no job-level permissions: blocks on any job. Without explicit permissions, workflows inherit the repository's default token permissions (which may be read/write), granting broader access than necessary.

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
   - actions/reusable-workflows@main → @4735e71081024a944852f4ab9d1495b6dd2de8f2
   - actions/checkout@v6 → @d23441a48e516b6c34aea4fa41551a30e30af803
   - actions/publish-immutable-action@v0.0.4 → @4bc8754ffc40f27910afb20287dbbbb675a4e978
   - actions/publish-action@v0.4.0 → @23f4c6f12633a2da8f44938b71fde9afec138fb4

2. script-injection: Moved all ${{ }} expressions out of run: blocks into env: blocks:
   - versions.yml: 3 occurrences of ${{ matrix.go }} in verify-go.sh calls (local-cache, setup-versions-from-manifest, setup-versions-from-dist)
   - windows-validation.yml: 5 occurrences of ${{ matrix.go }} and ${{ needs.find-default-go.outputs.version }} in du/path checks

3. missing-permissions: Added 'permissions: {}' top-level block to 8 workflow files that lacked explicit permissions (basic-validation.yml, check-dist.yml, codeql-analysis.yml, licensed.yml, microsoft-validation.yml, update-config-files.yml, versions.yml, windows-validation.yml).

