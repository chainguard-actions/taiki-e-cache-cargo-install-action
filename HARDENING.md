<!-- markdownlint-disable -->

# Hardening Report: taiki-e--cache-cargo-install-action/v3.0.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **taiki-e--cache-cargo-install-action/v3.0.4** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In pre.sh, the variable `bin_dir` is derived from `${tool}` (which comes from `INPUT_TOOL`, mapped from `inputs.tool` — a user-controlled value). It is written directly to `$GITHUB_PATH` via `printf '%s\n' "${bin_dir}" >> "${GITHUB_PATH}"` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A newline embedded in the tool name could inject arbitrary entries into GITHUB_PATH.

Locations:

- `pre.sh:279`

### github-env-injection (severity: high)

In pre.sh, a heredoc writes multiple user-controlled values (tool, version, key, git, tag, rev, features_flag, no_default_features_flag, all_features_flag) to `$GITHUB_OUTPUT` via `cat >> "${GITHUB_OUTPUT}" << EOF`. These values are derived from user inputs (inputs.tool, inputs.git, inputs.tag, inputs.rev, inputs.features, etc.) without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A newline in any of these values could inject arbitrary key=value pairs into GITHUB_OUTPUT.

Locations:

- `pre.sh:288`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in pre.sh: (1) Line 279: Added sanitization of bin_dir (derived from user-controlled INPUT_TOOL) before writing to GITHUB_PATH using 'safe_bin_dir=$(printf '%s' "${bin_dir}" | tr -d '\n\r')'. (2) Lines 288-300: Replaced the heredoc that wrote multiple user-controlled values (tool, version, key, git, tag, rev, features_flag, no_default_features_flag, all_features_flag) to GITHUB_OUTPUT with individual printf statements, each sanitizing the value with 'printf '%s' ... | tr -d '\n\r'' before writing.

