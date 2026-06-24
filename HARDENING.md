<!-- markdownlint-disable -->

# Hardening Report: taiki-e--cache-cargo-install-action/v3.0.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **taiki-e--cache-cargo-install-action/v3.0.6** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In pre.sh, the variable `bin_dir` (derived from `INPUT_TOOL`, an untrusted user-controlled input) is written directly to `$GITHUB_PATH` without sanitization (`printf '%s' ... | tr -d '\n\r'`). A newline in the tool name could inject arbitrary entries into the PATH. The offending line is: `printf '%s\n' "${bin_dir}" >> "${GITHUB_PATH}"`

Locations:

- `pre.sh:289`

### github-env-injection (severity: high)

In pre.sh, a heredoc writes multiple values derived from untrusted inputs (`tool`, `version`, `key`, `git`, `tag`, `rev`, `features_flag`, etc. — all sourced from `INPUT_TOOL`, `INPUT_GIT`, `INPUT_TAG`, `INPUT_REV`, `INPUT_FEATURES`, etc.) directly to `$GITHUB_OUTPUT` without sanitization (`printf '%s' ... | tr -d '\n\r'`). A newline in any of these values could inject arbitrary output variables. The offending block is: `cat >> "${GITHUB_OUTPUT}" << EOF`

Locations:

- `pre.sh:292`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in pre.sh:
1. Line 289 (GITHUB_PATH): Added sanitization of bin_dir using `printf '%s' "${bin_dir}" | tr -d '\n\r'` before writing to $GITHUB_PATH.
2. Lines 292+ (GITHUB_OUTPUT): Replaced the heredoc with individual printf statements, sanitizing all user-controlled values (tool, version, key, path, locked, git, tag, rev, features_flag, no_default_features_flag, all_features_flag) using `tr -d '\n\r'` before writing to $GITHUB_OUTPUT. Used a grouped `{ ... } >> "${GITHUB_OUTPUT}"` block for efficiency.

