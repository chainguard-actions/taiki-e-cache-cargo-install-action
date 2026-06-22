<!-- markdownlint-disable -->

# Hardening Report: taiki-e--cache-cargo-install-action/v3.0.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **taiki-e--cache-cargo-install-action/v3.0.5** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

pre.sh writes user-controlled values to $GITHUB_PATH and $GITHUB_OUTPUT without sanitization (no `printf '%s' ... | tr -d '\n\r'` step). Specifically: (1) `printf '%s\n' "${bin_dir}" >> "${GITHUB_PATH}"` where bin_dir is derived from `${RUNNER_TOOL_CACHE}/${tool}/bin` and `tool` comes from INPUT_TOOL (inputs.tool — attacker-controlled); (2) `cat >> "${GITHUB_OUTPUT}" <<EOF` writes multiple values including `tool`, `git`, `tag`, `rev`, `key`, and `features_flag` that are all derived from user-supplied inputs without newline sanitization. An attacker could inject newlines into these inputs to poison GITHUB_PATH or GITHUB_OUTPUT with arbitrary key=value pairs.

Locations:

- `pre.sh:281`
- `pre.sh:289`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed pre.sh to sanitize all user-controlled values before writing to $GITHUB_PATH and $GITHUB_OUTPUT. For GITHUB_PATH: added `safe_bin_dir=$(printf '%s' "${bin_dir}" | tr -d '\n\r')` and used `safe_bin_dir` in the printf. For GITHUB_OUTPUT: replaced the heredoc `cat >> "${GITHUB_OUTPUT}" <<EOF` with individual `printf` statements for each output variable, with each user-controlled value sanitized via `printf '%s' ... | tr -d '\n\r'` to strip newlines before writing. This prevents attackers from injecting newlines via INPUT_TOOL, INPUT_GIT, INPUT_TAG, INPUT_REV, or INPUT_FEATURES to poison GITHUB_PATH or GITHUB_OUTPUT with arbitrary key=value pairs.

