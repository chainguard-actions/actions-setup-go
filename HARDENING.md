<!-- markdownlint-disable -->

# Hardening Report: actions--setup-go/v4.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **actions--setup-go/v4.3.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags or branch names instead of full 40-character SHA commit hashes. This exposes the workflow to supply-chain attacks if the referenced tag or branch is moved or compromised. Affected references include: basic-validation.yml uses actions/reusable-workflows@main; check-dist.yml uses actions/reusable-workflows@main; codeql-analysis.yml uses actions/reusable-workflows@main; licensed.yml uses actions/reusable-workflows@main; publish-immutable-action.yml uses actions/checkout@v4 and actions/publish-immutable-action@v0.0.4; release-new-action-version.yml uses actions/publish-action@v0.2.2; update-config-files.yml uses actions/reusable-workflows@main; versions.yml uses actions/checkout@v3 (×10); windows-validation.yml uses actions/checkout@v3 (×2).

Locations:

- `.github/workflows/basic-validation.yml:14`
- `.github/workflows/check-dist.yml:15`
- `.github/workflows/codeql-analysis.yml:13`
- `.github/workflows/licensed.yml:12`
- `.github/workflows/publish-immutable-action.yml:14`
- `.github/workflows/publish-immutable-action.yml:17`
- `.github/workflows/release-new-action-version.yml:21`
- `.github/workflows/update-config-files.yml:13`
- `.github/workflows/versions.yml:21`
- `.github/workflows/versions.yml:35`
- `.github/workflows/versions.yml:55`
- `.github/workflows/versions.yml:76`
- `.github/workflows/versions.yml:95`
- `.github/workflows/versions.yml:113`
- `.github/workflows/versions.yml:131`
- `.github/workflows/versions.yml:153`
- `.github/workflows/versions.yml:175`
- `.github/workflows/versions.yml:193`
- `.github/workflows/windows-validation.yml:20`
- `.github/workflows/windows-validation.yml:62`

### missing-permissions (severity: medium)

Several workflow files have no top-level 'permissions:' key and no job-level 'permissions:' keys on any of their jobs. Without explicit permissions, the GITHUB_TOKEN is granted default (potentially write) permissions, violating the principle of least privilege. Affected files: basic-validation.yml, check-dist.yml, codeql-analysis.yml, licensed.yml, update-config-files.yml, versions.yml, windows-validation.yml.

Locations:

- `.github/workflows/basic-validation.yml:1`
- `.github/workflows/check-dist.yml:1`
- `.github/workflows/codeql-analysis.yml:1`
- `.github/workflows/licensed.yml:1`
- `.github/workflows/update-config-files.yml:1`
- `.github/workflows/versions.yml:1`
- `.github/workflows/windows-validation.yml:1`

### script-injection (severity: high)

Multiple run: blocks directly interpolate ${{ }} expressions into shell commands, enabling script injection. (a) In versions.yml, three steps use 'run: __tests__/verify-go.sh ${{ matrix.go }}' — the matrix value is substituted directly into the shell command string before the shell parses it. (b) In windows-validation.yml, ${{ matrix.go }} is interpolated directly inside run: blocks in path strings used with du and comparison expressions (e.g. 'du -m -s C:\hostedtoolcache\windows\go\${{ matrix.go }}\x64'). Additionally, ${{ needs.find-default-go.outputs.version }} (a step output, which is workflow-controllable) is interpolated directly in run: blocks. All of these violate sub-rule (a): any ${{ ... }} expression inside a run: shell command is a script-injection risk.

Locations:

- `.github/workflows/versions.yml:48`
- `.github/workflows/versions.yml:126`
- `.github/workflows/versions.yml:144`
- `.github/workflows/windows-validation.yml:26`
- `.github/workflows/windows-validation.yml:27`
- `.github/workflows/windows-validation.yml:33`
- `.github/workflows/windows-validation.yml:34`
- `.github/workflows/windows-validation.yml:44`
- `.github/workflows/windows-validation.yml:45`
- `.github/workflows/windows-validation.yml:47`
- `.github/workflows/windows-validation.yml:75`
- `.github/workflows/windows-validation.yml:80`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings across 9 workflow files:

1. unpinned-uses: Pinned all mutable references to full SHA hashes:
   - actions/reusable-workflows@main → @4735e71081024a944852f4ab9d1495b6dd2de8f2 (5 files)
   - actions/checkout@v3 → @a37ce9120846195fa4ece8f58b268e6043cb2f26 (12 occurrences across versions.yml and windows-validation.yml)
   - actions/checkout@v4 → @11d5960a326750d5838078e36cf38b85af677262 (publish-immutable-action.yml)
   - actions/publish-immutable-action@v0.0.4 → @4bc8754ffc40f27910afb20287dbbbb675a4e978
   - actions/publish-action@v0.2.2 → @dca2315f75060c81e52b00dfc86b660107013642

2. missing-permissions: Added top-level `permissions: {}` to basic-validation.yml, check-dist.yml, codeql-analysis.yml, licensed.yml, update-config-files.yml, versions.yml, and windows-validation.yml.

3. script-injection: Moved all ${{ }} expressions out of run: blocks into env: blocks:
   - versions.yml: 3 verify-go.sh steps now use GO_VERSION env var instead of ${{ matrix.go }}
   - windows-validation.yml: All path-building run: blocks now use GO_VERSION/$DEFAULT_GO_VERSION env vars instead of ${{ matrix.go }}/${{ needs.find-default-go.outputs.version }}

