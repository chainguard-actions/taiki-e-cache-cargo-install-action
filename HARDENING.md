<!-- markdownlint-disable -->

# Hardening Report: taiki-e--cache-cargo-install-action/v3.0.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **taiki-e--cache-cargo-install-action/v3.0.4** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

pre.sh writes user-controlled input values to $GITHUB_PATH and $GITHUB_OUTPUT without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`).

(1) `printf '%s\n' "${bin_dir}" >> "${GITHUB_PATH}"` — `bin_dir` is constructed as `${RUNNER_TOOL_CACHE}/${tool}/bin` where `tool` is read from `INPUT_TOOL` (mapped from `inputs.tool` in action.yml). An attacker-controlled crate name containing newlines could inject additional entries into GITHUB_PATH.

(2) `cat >> "${GITHUB_OUTPUT}" << EOF ... EOF` — the heredoc writes multiple user-controlled values (`tool`, `version`, `key`, `git`, `tag`, `rev`, `features_flag`, `no_default_features_flag`, `all_features_flag`) directly to $GITHUB_OUTPUT. These values are derived from `inputs.tool`, `inputs.git`, `inputs.tag`, `inputs.rev`, `inputs.features`, etc. A value containing a newline could inject additional key=value pairs into GITHUB_OUTPUT, potentially overwriting outputs consumed by downstream steps.

Neither write is preceded by the required sanitization pipeline (`printf '%s' "$VAR" | tr -d '\n\r'`).

Locations:

- `pre.sh:271`
- `pre.sh:278`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed pre.sh at the two locations identified in the finding:

1. GITHUB_PATH write (line ~271): Added `safe_bin_dir=$(printf '%s' "${bin_dir}" | tr -d '\n\r')` before the printf write, and used `safe_bin_dir` in the write instead of `bin_dir`.

2. GITHUB_OUTPUT heredoc write (line ~278): Added sanitization for all 10 user-controlled values (tool, version, key, locked, git, tag, rev, features_flag, no_default_features_flag, all_features_flag) using `printf '%s' "${VAR}" | tr -d '\n\r'` before the heredoc, and replaced all variable references in the heredoc with their sanitized `safe_*` counterparts. This prevents newline injection attacks where an attacker-controlled crate name or other input containing newlines could inject additional entries into GITHUB_PATH or additional key=value pairs into GITHUB_OUTPUT.

