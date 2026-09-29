<!-- markdownlint-disable -->

# Hardening Report: taiki-e--cache-cargo-install-action/v3.0.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **taiki-e--cache-cargo-install-action/v3.0.4** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In pre.sh, user-controlled input values are written to $GITHUB_PATH and $GITHUB_OUTPUT without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). 

(1) `printf '%s\n' "${bin_dir}" >> "${GITHUB_PATH}"` — `bin_dir` is constructed as `${RUNNER_TOOL_CACHE}/${tool}/bin` where `tool` is derived directly from `INPUT_TOOL` (the `inputs.tool` value set by the calling workflow). A newline embedded in the tool name could inject an arbitrary path into GITHUB_PATH.

(2) `cat >> "${GITHUB_OUTPUT}" <<EOF ... tool=${tool} ... git=${git} tag=${tag} rev=${rev} features_flag=${features_flag} ... EOF` — multiple user-controlled values (tool, git, tag, rev, and derived flags) are written to GITHUB_OUTPUT via a heredoc without sanitization. A newline in any of these values could inject additional key=value pairs into GITHUB_OUTPUT, potentially overwriting outputs consumed by downstream steps.

Locations:

- `pre.sh:276`
- `pre.sh:284`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection vulnerabilities in pre.sh:

1. GITHUB_PATH write (line ~276): Added sanitization of `bin_dir` (derived from user-controlled `INPUT_TOOL`) using `safe_bin_dir=$(printf '%s' "${bin_dir}" | tr -d '\n\r')` before writing to GITHUB_PATH.

2. GITHUB_OUTPUT write (line ~284): Added sanitization for all 10 user-controlled values written to GITHUB_OUTPUT (tool, version, key, path/bin_dir, locked, git, tag, rev, features_flag, no_default_features_flag, all_features_flag) using `printf '%s' "${VAR}" | tr -d '\n\r'` for each. The heredoc now uses the sanitized `safe_*` variables, preventing newline injection attacks that could inject additional key=value pairs into GITHUB_OUTPUT.

