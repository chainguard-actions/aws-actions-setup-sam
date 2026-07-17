<!-- markdownlint-disable -->

# Hardening Report: aws-actions--setup-sam/v2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **aws-actions--setup-sam/v2** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files use tag-based or version-based `uses:` references instead of full 40-character SHA commit hashes, making them vulnerable to supply-chain attacks if the referenced tag is moved or overwritten.

Failing references:
- `.github/workflows/codeql-analysis.yml`: `actions/checkout@v6`, `github/codeql-action/init@v4`, `github/codeql-action/autobuild@v4`, `github/codeql-action/analyze@v4`
- `.github/workflows/release.yml`: `actions/checkout@v6`
- `.github/workflows/test.yml`: `actions/checkout@v6`, `actions/setup-node@v6`, `actions/setup-python@v6`, `actions/setup-node@v6` (multiple occurrences)

Locations:

- `.github/workflows/codeql-analysis.yml:36`
- `.github/workflows/codeql-analysis.yml:40`
- `.github/workflows/codeql-analysis.yml:49`
- `.github/workflows/codeql-analysis.yml:55`
- `.github/workflows/release.yml:10`
- `.github/workflows/test.yml:14`
- `.github/workflows/test.yml:15`
- `.github/workflows/test.yml:28`
- `.github/workflows/test.yml:57`
- `.github/workflows/test.yml:58`
- `.github/workflows/test.yml:95`

### script-injection (severity: high)

Sub-rule (a): `${{ ... }}` expressions are interpolated directly inside `run:` shell command strings in `.github/workflows/test.yml`. Specifically, `${{ env.INSTALLER_VERSION }}` and `${{ env.SAM_VERSION }}` are embedded directly in shell `run:` commands (e.g., `run: sam --version | grep -F ${{ env.INSTALLER_VERSION }}` and `run: sam --version | grep -F ${{ env.SAM_VERSION }}`). Any `${{ ... }}` expression inside a `run:` block undergoes YAML template substitution before the shell ever sees it, bypassing shell quoting and enabling injection if the value contains shell metacharacters.

Locations:

- `.github/workflows/test.yml:67`
- `.github/workflows/test.yml:82`
- `.github/workflows/test.yml:88`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all unpinned action references by replacing tag-based references with full 40-character SHA commit hashes (with tag comments for readability) in codeql-analysis.yml, release.yml, and test.yml. Fixed script injection vulnerabilities in test.yml by moving ${{ env.INSTALLER_VERSION }} and ${{ env.SAM_VERSION }} expressions out of run: shell strings and into step-level env: blocks, then referencing them as plain shell variables ($INSTALLER_VERSION and $SAM_VERSION) in the shell commands.

