<!-- markdownlint-disable -->

# Hardening Report: GrantBirki--git-diff-action/v3.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **GrantBirki--git-diff-action/v3.0.0** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation of user-controlled inputs inside run: blocks. In update-latest-release-tag.yml, workflow_dispatch inputs `github.event.inputs.major_version_tag` and `github.event.inputs.source_tag` are interpolated directly into shell commands: `run: git tag -f ${{ github.event.inputs.major_version_tag }} ${{ github.event.inputs.source_tag }}` (line 30) and `run: git push origin ${{ github.event.inputs.major_version_tag }} --force` (line 33). An attacker with dispatch access can inject arbitrary shell commands via these inputs.

Locations:

- `.github/workflows/update-latest-release-tag.yml:30`
- `.github/workflows/update-latest-release-tag.yml:33`

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation inside a run: block. In acceptance.yml, `${{ steps.outside-workspace.outcome }}` and `${{ steps.bad-git-options.outcome }}` (steps.*.outputs.* context) are interpolated directly into shell command strings: `test "${{ steps.outside-workspace.outcome }}" = "failure"` and `test "${{ steps.bad-git-options.outcome }}" = "failure"`. Any ${{ ... }} expression inside a run: block is a script-injection risk regardless of context.

Locations:

- `.github/workflows/acceptance.yml:113`
- `.github/workflows/acceptance.yml:114`

### script-injection (severity: high)

Sub-rule (b): Unquoted shell variable expansion of workflow-controllable data. In sample-workflow.yml, the env var `DIFF` is set from `${{ steps.git-diff-action.outputs.json-diff-path }}` and `${{ steps.git-diff-action.outputs.raw-diff-path }}` (steps.*.outputs.* context) and then expanded unquoted in the run: block as `cat $DIFF` (lines 32 and 37). An unquoted expansion allows the shell to parse metacharacters from the value.

Locations:

- `.github/workflows/sample-workflow.yml:32`
- `.github/workflows/sample-workflow.yml:37`

### unpinned-uses (severity: high)

Multiple workflow files reference GitHub Actions using mutable version tags instead of full 40-character SHA commit hashes, making them vulnerable to supply-chain attacks if the tag is moved. Failing references include: actions/checkout@v6, actions/upload-artifact@v4, actions/setup-node@v4, github/codeql-action/init@v3, github/codeql-action/autobuild@v3, github/codeql-action/analyze@v3, GrantBirki/json-yaml-validate@v3.

Locations:

- `.github/workflows/acceptance.yml:17`
- `.github/workflows/acceptance.yml:52`
- `.github/workflows/acceptance.yml:57`
- `.github/workflows/acceptance.yml:82`
- `.github/workflows/acceptance.yml:97`
- `.github/workflows/codeql-analysis.yml:25`
- `.github/workflows/codeql-analysis.yml:29`
- `.github/workflows/codeql-analysis.yml:35`
- `.github/workflows/codeql-analysis.yml:39`
- `.github/workflows/copilot-setup-steps.yml:14`
- `.github/workflows/copilot-setup-steps.yml:19`
- `.github/workflows/json-yaml-validate.yml:14`
- `.github/workflows/json-yaml-validate.yml:17`
- `.github/workflows/lint.yml:12`
- `.github/workflows/lint.yml:16`
- `.github/workflows/package-check.yml:18`
- `.github/workflows/package-check.yml:22`
- `.github/workflows/package-check.yml:40`
- `.github/workflows/sample-workflow.yml:15`
- `.github/workflows/sample-workflow.yml:40`
- `.github/workflows/test.yml:12`
- `.github/workflows/test.yml:16`
- `.github/workflows/update-latest-release-tag.yml:22`

### missing-permissions (severity: medium)

The workflow file package-check.yml has no top-level `permissions:` key and no job-level `permissions:` key on any of its jobs. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be broader than necessary.

Locations:

- `.github/workflows/package-check.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all 5 findings across 9 workflow files:

1. script-injection in update-latest-release-tag.yml: Moved github.event.inputs.major_version_tag and github.event.inputs.source_tag into env: blocks and quoted the shell variable references.

2. script-injection in acceptance.yml: Moved steps.outside-workspace.outcome and steps.bad-git-options.outcome into an env: block (OUTSIDE_WORKSPACE_OUTCOME, BAD_GIT_OPTIONS_OUTCOME) and referenced them as quoted shell variables.

3. script-injection in sample-workflow.yml: Fixed unquoted `cat $DIFF` to `cat "$DIFF"` in both print steps.

4. unpinned-uses: Pinned all action references to full 40-char SHAs with tag comments: actions/checkout@v6→d23441a4, actions/upload-artifact@v4→ea165f8d, actions/setup-node@v4→49933ea5, github/codeql-action/*@v3→4187e74d, GrantBirki/json-yaml-validate@v3→250fa0dc.

5. missing-permissions in package-check.yml: Added top-level `permissions: contents: read` block.

