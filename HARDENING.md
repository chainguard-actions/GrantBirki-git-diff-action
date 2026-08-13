<!-- markdownlint-disable -->

# Hardening Report: GrantBirki--git-diff-action/v2.8.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **GrantBirki--git-diff-action/v2.8.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All `uses:` references across workflow files use mutable tags instead of pinned 40-character SHA commit hashes, making the workflows vulnerable to supply-chain attacks if the referenced action tags are moved. Failing references include: actions/checkout@v4, actions/upload-artifact@v4, actions/setup-node@v4, github/codeql-action/init@v3, github/codeql-action/autobuild@v3, github/codeql-action/analyze@v3, GrantBirki/json-yaml-validate@v3.

Locations:

- `.github/workflows/acceptance.yml:16`
- `.github/workflows/acceptance.yml:43`
- `.github/workflows/codeql-analysis.yml:22`
- `.github/workflows/codeql-analysis.yml:26`
- `.github/workflows/codeql-analysis.yml:31`
- `.github/workflows/codeql-analysis.yml:34`
- `.github/workflows/copilot-setup-steps.yml:16`
- `.github/workflows/copilot-setup-steps.yml:21`
- `.github/workflows/json-yaml-validate.yml:14`
- `.github/workflows/json-yaml-validate.yml:17`
- `.github/workflows/lint.yml:13`
- `.github/workflows/lint.yml:17`
- `.github/workflows/package-check.yml:18`
- `.github/workflows/package-check.yml:22`
- `.github/workflows/package-check.yml:38`
- `.github/workflows/sample-workflow.yml:16`
- `.github/workflows/sample-workflow.yml:34`
- `.github/workflows/test.yml:13`
- `.github/workflows/test.yml:17`
- `.github/workflows/update-latest-release-tag.yml:21`

### script-injection (severity: high)

Sub-rule (a): update-latest-release-tag.yml directly interpolates workflow_dispatch user-controlled inputs into run: shell commands without going through an env: variable. The expressions `${{ github.event.inputs.major_version_tag }}` and `${{ github.event.inputs.source_tag }}` are expanded by the template engine before the shell sees them, allowing an attacker to inject arbitrary shell commands via the input values. Offending lines: `run: git tag -f ${{ github.event.inputs.major_version_tag }} ${{ github.event.inputs.source_tag }}` and `run: git push origin ${{ github.event.inputs.major_version_tag }} --force`.

Sub-rule (b): acceptance.yml and sample-workflow.yml expand `$DIFF` (sourced from `steps.git-diff-action.outputs.*`) unquoted in `run: cat $DIFF`. The env var holds a workflow-controllable value and must be double-quoted: `run: cat "$DIFF"`.

Locations:

- `.github/workflows/update-latest-release-tag.yml:31`
- `.github/workflows/update-latest-release-tag.yml:34`
- `.github/workflows/acceptance.yml:30`
- `.github/workflows/acceptance.yml:36`
- `.github/workflows/sample-workflow.yml:28`
- `.github/workflows/sample-workflow.yml:34`

### missing-permissions (severity: medium)

package-check.yml has no top-level `permissions:` key and no job-level `permissions:` key on any of its jobs. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad (e.g. write access to contents). A minimal `permissions:` block such as `permissions: contents: read` should be added.

Locations:

- `.github/workflows/package-check.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three findings across 8 workflow files:

1. unpinned-uses: Pinned all 20 `uses:` references to full 40-char SHAs with tag comments preserved: actions/checkout@v4→11d5960a, actions/upload-artifact@v4→ea165f8d, actions/setup-node@v4→49933ea5, github/codeql-action/{init,autobuild,analyze}@v3→4187e74d, GrantBirki/json-yaml-validate@v3→250fa0dc.

2. script-injection: (a) In update-latest-release-tag.yml, moved `${{ github.event.inputs.major_version_tag }}` and `${{ github.event.inputs.source_tag }}` into `env:` blocks and used double-quoted `"$MAJOR_VERSION_TAG"` / `"$SOURCE_TAG"` in shell commands. (b) In acceptance.yml and sample-workflow.yml, changed `cat $DIFF` to `cat "$DIFF"` to properly quote the env var.

3. missing-permissions: Added `permissions: contents: read` top-level block to package-check.yml.

