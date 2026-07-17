<!-- markdownlint-disable -->

# Hardening Report: aws-actions--setup-sam/v2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **aws-actions--setup-sam/v2** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference GitHub Actions using mutable version tags instead of full 40-character SHA commit hashes. This exposes the workflow to supply-chain attacks where a tag could be silently moved to a malicious commit.

Failing references:
- .github/workflows/test.yml: `actions/checkout@v6`, `actions/setup-node@v6` (line 17, 18), `actions/checkout@v6` (line 33), `actions/checkout@v6` (line 71), `actions/setup-python@v6` (line 72), `actions/setup-node@v6` (line 113)
- .github/workflows/release.yml: `actions/checkout@v6` (line 10)
- .github/workflows/codeql-analysis.yml: `actions/checkout@v6` (line 35), `github/codeql-action/init@v4` (line 39), `github/codeql-action/autobuild@v4` (line 46), `github/codeql-action/analyze@v4` (line 60)

Locations:

- `.github/workflows/test.yml:17`
- `.github/workflows/test.yml:18`
- `.github/workflows/test.yml:33`
- `.github/workflows/test.yml:71`
- `.github/workflows/test.yml:72`
- `.github/workflows/test.yml:113`
- `.github/workflows/release.yml:10`
- `.github/workflows/codeql-analysis.yml:35`
- `.github/workflows/codeql-analysis.yml:39`
- `.github/workflows/codeql-analysis.yml:46`
- `.github/workflows/codeql-analysis.yml:60`

### script-injection (severity: high)

Two `run:` steps in .github/workflows/test.yml directly interpolate `${{ env.* }}` expressions into shell commands (rule a). Even though `env.*` values are set in the same workflow's `env:` block from `matrix.*` context, any `${{ ... }}` expression interpolated directly into a `run:` block is a script-injection risk because the value is substituted into the shell command string before the shell parses it, allowing shell metacharacters to be injected.

Offending lines:
- Line 79: `run: sam --version | grep -F ${{ env.INSTALLER_VERSION }}`
- Line 97: `run: sam --version | grep -F ${{ env.SAM_VERSION }}`

Fix: Move the value into an env var and reference it as a quoted shell variable, e.g.:
```yaml
env:
  INSTALLER_VERSION: ${{ env.INSTALLER_VERSION }}
run: sam --version | grep -F "$INSTALLER_VERSION"
```

Locations:

- `.github/workflows/test.yml:79`
- `.github/workflows/test.yml:97`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all 11 unpinned action references across test.yml, release.yml, and codeql-analysis.yml by pinning each to its full 40-character SHA with the original tag preserved as a comment. Fixed 2 script-injection vulnerabilities in test.yml (lines 79 and 97) by moving ${{ env.INSTALLER_VERSION }} and ${{ env.SAM_VERSION }} expressions into step-level env: blocks and referencing them as quoted shell variables ($INSTALLER_VERSION and $SAM_VERSION) in the run: commands.

