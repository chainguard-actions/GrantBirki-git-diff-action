<!-- markdownlint-disable -->

# Hardening Report: GrantBirki--git-diff-action/v2.6.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **GrantBirki--git-diff-action/v2.6.1** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation in run: blocks. In update-latest-release-tag.yml, workflow_dispatch inputs are interpolated directly into shell commands: `run: git tag -f ${{ github.event.inputs.major_version_tag }} ${{ github.event.inputs.source_tag }}` and `run: git push origin ${{ github.event.inputs.major_version_tag }} --force`. An attacker who can trigger the workflow_dispatch (or a maintainer supplying malicious input) could inject arbitrary shell commands via these values.

Locations:

- `.github/workflows/update-latest-release-tag.yml:32`
- `.github/workflows/update-latest-release-tag.yml:35`

### script-injection (severity: high)

Sub-rule (b): Unquoted shell variable expansion of untrusted data. In acceptance.yml and sample-workflow.yml, the env var $DIFF (set from `${{ steps.git-diff-action.outputs.json-diff-path }}` and `${{ steps.git-diff-action.outputs.raw-diff-path }}`) is used unquoted in `run: cat $DIFF`. Shell metacharacters in the step output value could be parsed by the shell. The variable should be quoted: `run: cat "$DIFF"`.

Locations:

- `.github/workflows/acceptance.yml:31`
- `.github/workflows/acceptance.yml:36`
- `.github/workflows/sample-workflow.yml:31`
- `.github/workflows/sample-workflow.yml:36`

### unpinned-uses (severity: high)

All workflow files reference GitHub Actions using mutable version tags (e.g. @v4, @v3, @v2) instead of pinned 40-character commit SHA hashes. This exposes the workflows to supply-chain attacks if any referenced action's tag is moved to point to malicious code. Affected references include: actions/checkout@v4, actions/setup-node@v4, actions/upload-artifact@v4, github/codeql-action/init@v3, github/codeql-action/autobuild@v3, github/codeql-action/analyze@v3, GrantBirki/json-yaml-validate@v2.

Locations:

- `.github/workflows/acceptance.yml:16`
- `.github/workflows/acceptance.yml:47`
- `.github/workflows/codeql-analysis.yml:22`
- `.github/workflows/codeql-analysis.yml:27`
- `.github/workflows/codeql-analysis.yml:33`
- `.github/workflows/codeql-analysis.yml:37`
- `.github/workflows/json-yaml-validate.yml:15`
- `.github/workflows/json-yaml-validate.yml:18`
- `.github/workflows/lint.yml:13`
- `.github/workflows/lint.yml:17`
- `.github/workflows/package-check.yml:17`
- `.github/workflows/package-check.yml:21`
- `.github/workflows/package-check.yml:38`
- `.github/workflows/sample-workflow.yml:16`
- `.github/workflows/sample-workflow.yml:42`
- `.github/workflows/test.yml:13`
- `.github/workflows/test.yml:17`
- `.github/workflows/update-latest-release-tag.yml:22`

### missing-permissions (severity: medium)

package-check.yml has no top-level `permissions:` key and no job-level `permissions:` key on any of its jobs. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad (write access to contents and other scopes). A `permissions:` block with minimal required scopes should be added.

Locations:

- `.github/workflows/package-check.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all 4 findings across 7 workflow files:

1. script-injection in update-latest-release-tag.yml (lines 32, 35): Moved github.event.inputs.major_version_tag and github.event.inputs.source_tag into env: blocks as MAJOR_VERSION_TAG and SOURCE_TAG, then referenced them as quoted shell variables in run: commands.

2. script-injection in acceptance.yml and sample-workflow.yml (lines 31, 36 in each): Quoted the $DIFF variable in `run: cat "$DIFF"` to prevent shell metacharacter interpretation.

3. unpinned-uses across all workflow files: Pinned all 7 action references to full 40-character commit SHAs (actions/checkout@v4→11d5960a, actions/setup-node@v4→49933ea5, actions/upload-artifact@v4→ea165f8d, github/codeql-action/*@v3→4187e74d, GrantBirki/json-yaml-validate@v2→d7814b94), preserving the original tag in a comment.

4. missing-permissions in package-check.yml: Added `permissions: contents: read` top-level block.

