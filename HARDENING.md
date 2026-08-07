<!-- markdownlint-disable -->

# Hardening Report: shogo82148--actions-create-release/v1.9.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **shogo82148--actions-create-release/v1.9.1** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The `cleanup` job's `run:` block in test.yml directly interpolates GitHub Actions expressions into shell commands (sub-rule a). The expressions `${{ github.repository }}`, `${{ needs.integrated.outputs.id }}`, `${{ needs.integrated-overwrite.outputs.id }}`, and `${{ github.run_id }}` are substituted into the shell string before the shell parses it, allowing an attacker who can influence these values to inject arbitrary shell commands. These should be passed via `env:` variables and referenced as quoted shell variables (e.g. `"$GITHUB_REPOSITORY"`) instead.

Locations:

- `.github/workflows/test.yml:85`
- `.github/workflows/test.yml:86`
- `.github/workflows/test.yml:87`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key and no job-level `permissions:` key on any job. Without explicit permissions, the GITHUB_TOKEN is granted its default (potentially broad) permissions. A `permissions:` block with minimal required scopes should be added.

Locations:

- `.github/workflows/check-dist.yml:1`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key and no job-level `permissions:` key on any job. Without explicit permissions, the GITHUB_TOKEN is granted its default (potentially broad) permissions. A `permissions:` block with minimal required scopes should be added.

Locations:

- `.github/workflows/reviewdog.yml:1`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key and no job-level `permissions:` key on any of its jobs (`test`, `check-permission`, `integrated`, `integrated-overwrite`, `cleanup`). Without explicit permissions, the GITHUB_TOKEN is granted its default (potentially broad) permissions. A `permissions:` block with minimal required scopes should be added.

Locations:

- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, missing-permissions

**Notes:**

Fixed all four findings across three workflow files:

1. **test.yml - script-injection**: Moved `${{ needs.integrated.outputs.id }}` and `${{ needs.integrated-overwrite.outputs.id }}` into the `env:` block as `INTEGRATED_ID` and `INTEGRATED_OVERWRITE_ID`. The `${{ github.repository }}` and `${{ github.run_id }}` expressions were replaced with the built-in `GITHUB_REPOSITORY` and `GITHUB_RUN_ID` environment variables that GitHub Actions already provides. Shell commands now reference these as quoted `$VAR` references.

2. **test.yml - missing-permissions**: Added `permissions: {}` at the top level and job-level permissions: `contents: read` for `test` and `check-permission` jobs; `contents: write` for `integrated`, `integrated-overwrite`, and `cleanup` jobs (which create/delete GitHub releases and tags).

3. **check-dist.yml - missing-permissions**: Added `permissions: {}` at the top level since this workflow only reads code and uploads artifacts.

4. **reviewdog.yml - missing-permissions**: Added `permissions: contents: read` and `pull-requests: write` since the reviewdog action with `github-pr-review` reporter needs to post PR review comments.

