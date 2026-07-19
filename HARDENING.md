<!-- markdownlint-disable -->

# Hardening Report: aws-actions--setup-sam/v3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **aws-actions--setup-sam/v3** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference GitHub Actions using mutable tag refs (@v4, @v6) instead of pinned 40-character SHA commit digests. This exposes the workflow to supply-chain attacks if the tag is moved to a malicious commit.

codeql-analysis.yml: actions/checkout@v6, github/codeql-action/init@v4, github/codeql-action/autobuild@v4, github/codeql-action/analyze@v4
release.yml: actions/checkout@v6
test.yml: actions/checkout@v6, actions/setup-node@v6, actions/setup-python@v6

Locations:

- `.github/workflows/codeql-analysis.yml:34`
- `.github/workflows/codeql-analysis.yml:38`
- `.github/workflows/codeql-analysis.yml:46`
- `.github/workflows/codeql-analysis.yml:53`
- `.github/workflows/release.yml:10`
- `.github/workflows/test.yml:13`
- `.github/workflows/test.yml:14`
- `.github/workflows/test.yml:50`
- `.github/workflows/test.yml:72`
- `.github/workflows/test.yml:100`

### script-injection (severity: high)

Two run: steps in test.yml directly interpolate ${{ ... }} expressions inside shell command strings (sub-rule a). Although the values are derived from env: variables set to matrix-computed literals, any ${{ ... }} expression inside a run: block is a script-injection risk because YAML template substitution occurs before the shell ever sees the string.

Offending lines:
  `run: sam --version | grep -F ${{ env.INSTALLER_VERSION }}`
  `run: sam --version | grep -F ${{ env.SAM_VERSION }}`

Fix: move the value into an env: var and reference it as a quoted shell variable, e.g. `grep -F "$INSTALLER_VERSION"`.

Locations:

- `.github/workflows/test.yml:83`
- `.github/workflows/test.yml:107`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all unpinned action references across 3 workflow files by resolving mutable tags to full 40-character SHA digests: actions/checkout@v6→df4cb1c, github/codeql-action/{init,autobuild,analyze}@v4→7188fc3, actions/setup-node@v6→249970729, actions/setup-python@v6→ece7cb06. Fixed 2 script-injection instances in test.yml where ${{ env.INSTALLER_VERSION }} and ${{ env.SAM_VERSION }} were interpolated directly in run: shell strings — moved each into the step's env: block and referenced as quoted shell variables ($INSTALLER_VERSION and $SAM_VERSION).

