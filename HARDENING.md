<!-- markdownlint-disable -->

# Hardening Report: taiki-e--cache-cargo-install-action/v3.0.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **taiki-e--cache-cargo-install-action/v3.0.4** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

pre.sh writes the unsanitized value of `${bin_dir}` (derived from `${RUNNER_TOOL_CACHE}/${tool}`, where `tool` comes from `INPUT_TOOL` = `${{ inputs.tool }}`) directly to `$GITHUB_PATH` without the required `printf '%s' ... | tr -d '\n\r'` sanitization step. An attacker-controlled crate name containing newlines could inject arbitrary entries into GITHUB_PATH.

Locations:

- `pre.sh:248`

### github-env-injection (severity: high)

pre.sh writes multiple unsanitized user-controlled values to `$GITHUB_OUTPUT` via a heredoc: `tool`, `version`, `key`, `git`, `tag`, `rev`, `features_flag`, `no_default_features_flag`, and `all_features_flag` — all derived from `inputs.*` (e.g. `INPUT_TOOL`, `INPUT_GIT`, `INPUT_TAG`, `INPUT_REV`, `INPUT_FEATURES`). None of these values are passed through `printf '%s' ... | tr -d '\n\r'` before being written. An attacker-controlled input containing newlines could inject arbitrary key=value pairs into GITHUB_OUTPUT, potentially overwriting subsequent step outputs.

Locations:

- `pre.sh:255`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in pre.sh:
1. Line 248 (GITHUB_PATH): Added sanitization of bin_dir via `safe_bin_dir=$(printf '%s' "${bin_dir}" | tr -d '\n\r')` before writing to $GITHUB_PATH.
2. Lines 255-267 (GITHUB_OUTPUT heredoc): Added sanitization for all 10 user-controlled values (tool, version, key, locked, git, tag, rev, features_flag, no_default_features_flag, all_features_flag) using `printf '%s' "${VAR}" | tr -d '\n\r'` before writing them to $GITHUB_OUTPUT. This prevents newline injection attacks where attacker-controlled inputs could inject arbitrary entries into GITHUB_PATH or arbitrary key=value pairs into GITHUB_OUTPUT.

