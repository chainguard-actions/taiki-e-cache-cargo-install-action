<!-- markdownlint-disable -->

# Hardening Report: taiki-e--cache-cargo-install-action/v3.0.7

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **taiki-e--cache-cargo-install-action/v3.0.7** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In pre.sh, the variable `bin_dir` (derived from `${RUNNER_TOOL_CACHE}/${tool}/bin` where `tool` comes from `INPUT_TOOL` / `inputs.tool`) is written to `$GITHUB_PATH` without sanitization: `printf '%s\n' "${bin_dir}" >> "${GITHUB_PATH}"`. The required `tr -d '\n\r'` step is missing. An attacker-controlled `inputs.tool` value containing an embedded newline could inject additional entries into GITHUB_PATH.

Locations:

- `pre.sh:330`

### github-env-injection (severity: high)

In pre.sh, a heredoc writes multiple values derived from action inputs (`tool`, `version`, `key`, `git`, `tag`, `rev`, `features_flag`, `no_default_features_flag`, `all_features_flag`) directly to `$GITHUB_OUTPUT` without sanitization (`tr -d '\n\r'`). All of these values trace back to `inputs.*` (e.g. `inputs.tool`, `inputs.git`, `inputs.tag`, `inputs.rev`, `inputs.features`). An attacker-controlled input containing a newline could inject additional key=value pairs into GITHUB_OUTPUT, potentially overwriting subsequent outputs.

Locations:

- `pre.sh:338`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in pre.sh:
1. Line 330: Added `safe_bin_dir=$(printf '%s' "${bin_dir}" | tr -d '\n\r')` and used `safe_bin_dir` when writing to `$GITHUB_PATH`, preventing newline injection via the `tool` input.
2. Line 338: Sanitized all 10 input-derived variables (`tool`, `version`, `key`, `locked`, `git`, `tag`, `rev`, `features_flag`, `no_default_features_flag`, `all_features_flag`) with `printf '%s' "${VAR}" | tr -d '\n\r'` before writing them to `$GITHUB_OUTPUT` via heredoc, preventing newline injection that could overwrite subsequent output values.

