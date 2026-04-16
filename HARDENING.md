# Hardening Report: bufbuild--buf-push-action/v1.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `c40cfe5fa14e08549b1b988e7e5a26da4816abf0`

**Test Policy SHA:** `f2e7d85641cde4267138117189b8eba7ba2bfbde`

Action **bufbuild--buf-push-action/v1.2.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

In action.yml, the `run:` step directly interpolates `${{ inputs.input }}` into the shell command line: `run: $GITHUB_ACTION_PATH/push.bash ${{ inputs.input }}`. This means any value supplied for the `input` parameter is expanded directly into the shell command string before execution, allowing an attacker who controls that input to inject arbitrary shell commands (e.g. by passing a value like `.; malicious-command`). The fix is to assign `inputs.input` to an environment variable (e.g. `INPUT_PATH: ${{ inputs.input }}`) and then reference `"$INPUT_PATH"` in the run command instead.

Locations:

- `action.yml:29`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.input }}" appears directly in run: block of step "push"; move to env: map

Locations:

- `action.yml:31`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Fixed both findings in action.yml by moving `${{ inputs.input }}` out of the `run:` block and into the `env:` block as `INPUT_PATH: ${{ inputs.input }}`. The shell command was updated from `$GITHUB_ACTION_PATH/push.bash ${{ inputs.input }}` to `$GITHUB_ACTION_PATH/push.bash "$INPUT_PATH"`, preventing shell command injection via the input parameter.

