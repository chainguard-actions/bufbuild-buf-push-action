<!-- markdownlint-disable -->

# Hardening Report: bufbuild--buf-push-action/v1.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **bufbuild--buf-push-action/v1.2.0** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The `run:` block in action.yml directly interpolates `${{ inputs.input }}` into the shell command string: `run: $GITHUB_ACTION_PATH/push.bash ${{ inputs.input }}`. Before the shell ever sees the command, GitHub Actions substitutes the expression value verbatim, allowing an attacker to inject arbitrary shell metacharacters or commands via the `input` action parameter.

Locations:

- `action.yml:29`

### unpinned-uses (severity: high)

Multiple workflow files reference actions/workflows by mutable tags or branch names instead of full 40-character commit SHAs:
- ci.yaml: `uses: actions/checkout@v2` (tag, not SHA) — appears twice (lines 9 and 18)
- emergency-review-bypass.yaml: `uses: bufbuild/base-workflows/.github/workflows/emergency-review-bypass.yaml@main` (branch ref)
- notify-approval-bypass.yaml: `uses: bufbuild/base-workflows/.github/workflows/notify-approval-bypass.yaml@main` (branch ref)
- pr-title.yaml: `uses: bufbuild/base-workflows/.github/workflows/pr-title.yaml@main` (branch ref)

Locations:

- `.github/workflows/ci.yaml:9`
- `.github/workflows/ci.yaml:18`
- `.github/workflows/emergency-review-bypass.yaml:11`
- `.github/workflows/notify-approval-bypass.yaml:12`
- `.github/workflows/pr-title.yaml:15`

### missing-permissions (severity: medium)

ci.yaml has no top-level `permissions:` key and neither of its jobs (`test`, `action-test`) defines a job-level `permissions:` block. The workflow therefore runs with GitHub's default token permissions, which may be broader than necessary (e.g. write access to contents on push events).

Locations:

- `.github/workflows/ci.yaml:1`

### hardcoded-credentials (severity: high)

ci.yaml contains literal hardcoded token values assigned to token-named fields: `buf_token: fake-buf-token` (used as the `buf_token` input) and `WANT_BUF_TOKEN: fake-buf-token` (set as an env var). These are literal alphanumeric strings assigned to names matching the `token` pattern and are present in an in-scope workflow file.

Locations:

- `.github/workflows/ci.yaml:29`
- `.github/workflows/ci.yaml:32`
- `.github/workflows/ci.yaml:37`
- `.github/workflows/ci.yaml:41`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.input }}" appears directly in run: block of step "push"; move to env: map

Locations:

- `action.yml:31`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, unpinned-uses, missing-permissions, hardcoded-credentials

**Notes:**

Fixed all 5 findings:
1. script-injection/static-inline-injection (action.yml line 29/31): Moved `${{ inputs.input }}` from the `run:` shell string into the `env:` block as `INPUT`, and updated the run command to use `"$INPUT"` safely.
2. unpinned-uses (ci.yaml lines 9,18; emergency-review-bypass.yaml line 11; notify-approval-bypass.yaml line 12; pr-title.yaml line 15): Pinned `actions/checkout@v2` to SHA `0717577d45739eb3c851188b29f50ed6c0b2194e` and all three `bufbuild/base-workflows@main` references to SHA `7c0b54b718244c78f00037343ff7cb33ef7caca9`.
3. missing-permissions (ci.yaml): Added top-level `permissions: contents: read` block.
4. hardcoded-credentials (ci.yaml lines 29,32,37,41): Replaced all four occurrences of the literal `fake-buf-token` string with `${{ secrets.BUF_TOKEN }}` references.

