<!-- markdownlint-disable -->

# Hardening Report: taiki-e--cache-cargo-install-action/v3.0.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **taiki-e--cache-cargo-install-action/v3.0.5** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

pre.sh writes user-controlled input values to $GITHUB_PATH and $GITHUB_OUTPUT without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`).

(1) Line ~311: `printf '%s\n' "${bin_dir}" >> "${GITHUB_PATH}"` — `bin_dir` is constructed as `${RUNNER_TOOL_CACHE}/${tool}/bin` where `tool` is read from `INPUT_TOOL` (mapped from `inputs.tool`, a caller-controlled value). A newline embedded in the tool name could inject arbitrary entries into $GITHUB_PATH.

(2) Line ~319: A heredoc `cat >> "${GITHUB_OUTPUT}" <<EOF` writes `tool`, `version`, `key`, `git`, `tag`, `rev`, `features_flag`, `no_default_features_flag`, and `all_features_flag` — all derived from caller-controlled inputs (`inputs.tool`, `inputs.git`, `inputs.tag`, `inputs.rev`, `inputs.features`, etc.) — directly to $GITHUB_OUTPUT without sanitization. A newline in any of these values could inject additional key=value pairs into the output context.

Locations:

- `pre.sh:311`
- `pre.sh:319`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection vulnerabilities in pre.sh:

1. GITHUB_PATH write (line ~311): Changed `printf '%s\n' "${bin_dir}" >> "${GITHUB_PATH}"` to sanitize bin_dir using `printf '%s' "${bin_dir}" | tr -d '\n\r'` before writing. bin_dir is derived from INPUT_TOOL (caller-controlled), so a newline in the tool name could inject arbitrary entries into $GITHUB_PATH.

2. GITHUB_OUTPUT write (line ~319): Replaced the heredoc `cat >> "${GITHUB_OUTPUT}" <<EOF` (which wrote tool, version, key, path, locked, git, tag, rev, features_flag, no_default_features_flag, all_features_flag directly) with individual `printf` calls that each sanitize their value with `printf '%s' "${VAR}" | tr -d '\n\r'` before writing. This prevents a newline embedded in any caller-controlled input from injecting additional key=value pairs into the output context.

