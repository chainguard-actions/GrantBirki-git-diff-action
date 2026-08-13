<!-- markdownlint-disable -->

# Hardening Report: GrantBirki--git-diff-action/v2.8.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **GrantBirki--git-diff-action/v2.8.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): Direct expression interpolation in run: blocks. In update-latest-release-tag.yml, user-controlled workflow_dispatch inputs are interpolated directly into shell commands: `run: git tag -f ${{ github.event.inputs.major_version_tag }} ${{ github.event.inputs.source_tag }}` and `run: git push origin ${{ github.event.inputs.major_version_tag }} --force`. An attacker with write access (or a malicious dispatcher) could inject arbitrary shell commands via these inputs.

Locations:

- `.github/workflows/update-latest-release-tag.yml:31`
- `.github/workflows/update-latest-release-tag.yml:35`

### script-injection (severity: high)

Rule (b): Unquoted shell variable expansion of untrusted data. The env var DIFF is set from `${{ steps.git-diff-action.outputs.json-diff-path }}` and `${{ steps.git-diff-action.outputs.raw-diff-path }}` (step outputs, a workflow-controllable context), then used unquoted as `run: cat $DIFF`. The unquoted expansion allows shell metacharacter injection if the output path contains spaces or special characters.

Locations:

- `.github/workflows/acceptance.yml:30`
- `.github/workflows/acceptance.yml:36`
- `.github/workflows/sample-workflow.yml:30`
- `.github/workflows/sample-workflow.yml:36`

### unpinned-uses (severity: high)

All workflow files reference GitHub Actions using mutable tag refs (e.g. @v4, @v3, @v2) instead of immutable full 40-character commit SHA digests. This exposes the workflows to supply-chain attacks if any of these action repositories are compromised and the tag is moved. Affected references include: actions/checkout@v4, actions/upload-artifact@v4, actions/setup-node@v4, github/codeql-action/init@v3, github/codeql-action/autobuild@v3, github/codeql-action/analyze@v3, GrantBirki/json-yaml-validate@v2.

Locations:

- `.github/workflows/acceptance.yml:14`
- `.github/workflows/acceptance.yml:50`
- `.github/workflows/codeql-analysis.yml:22`
- `.github/workflows/codeql-analysis.yml:27`
- `.github/workflows/codeql-analysis.yml:33`
- `.github/workflows/codeql-analysis.yml:37`
- `.github/workflows/json-yaml-validate.yml:14`
- `.github/workflows/json-yaml-validate.yml:17`
- `.github/workflows/lint.yml:13`
- `.github/workflows/lint.yml:17`
- `.github/workflows/package-check.yml:15`
- `.github/workflows/package-check.yml:19`
- `.github/workflows/package-check.yml:40`
- `.github/workflows/sample-workflow.yml:14`
- `.github/workflows/sample-workflow.yml:44`
- `.github/workflows/test.yml:13`
- `.github/workflows/test.yml:17`
- `.github/workflows/update-latest-release-tag.yml:21`

### missing-permissions (severity: medium)

The workflow file package-check.yml has no top-level `permissions:` key and no job-level `permissions:` key on any of its jobs. Without explicit permissions, the workflow inherits the default repository permissions (which may include write access to contents and other scopes), violating the principle of least privilege.

Locations:

- `.github/workflows/package-check.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all 4 findings across 7 workflow files:
1. script-injection (update-latest-release-tag.yml): Moved workflow_dispatch inputs (major_version_tag, source_tag) into step env: blocks and quoted them as "$MAJOR_VERSION_TAG" / "$SOURCE_TAG" in shell commands.
2. script-injection (acceptance.yml, sample-workflow.yml): Quoted the $DIFF variable as "$DIFF" in `cat` commands to prevent shell metacharacter injection.
3. unpinned-uses: Pinned all 7 action references to full 40-char SHAs with tag comments: actions/checkout@34e114876b0b11c390a56381ad16ebd13914f8d5 # v4, actions/upload-artifact@ea165f8d65b6e75b540449e92b4886f43607fa02 # v4, actions/setup-node@49933ea5288caeca8642d1e84afbd3f7d6820020 # v4, github/codeql-action/{init,autobuild,analyze}@b7351df727350dca84cb9d725d57dcf5bc82ba26 # v3, GrantBirki/json-yaml-validate@d7814b94473939c1daaca2c96131b891d4703a3c # v2.
4. missing-permissions (package-check.yml): Added top-level `permissions: contents: read`.

