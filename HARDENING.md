<!-- markdownlint-disable -->

# Hardening Report: taiki-e--cache-cargo-install-action/v3.0.7

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **taiki-e--cache-cargo-install-action/v3.0.7** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

pre.sh writes user-controlled input values to $GITHUB_PATH and $GITHUB_OUTPUT without the required sanitization (`printf '%s' ... | tr -d '\n\r'`). 

1. `printf '%s\n' "${bin_dir}" >> "${GITHUB_PATH}"` — `bin_dir` is derived from `${RUNNER_TOOL_CACHE}/${tool}/bin` where `tool` comes from `INPUT_TOOL` (mapped from `inputs.tool` in action.yml). A newline embedded in the tool name could inject additional PATH entries.

2. `cat >> "${GITHUB_OUTPUT}" << EOF` — the heredoc writes multiple unsanitized values (`tool`, `version`, `key`, `git`, `tag`, `rev`, `features_flag`, etc.) all derived from user-supplied inputs. A newline in any of these values could inject arbitrary key=value pairs into GITHUB_OUTPUT, allowing an attacker to override subsequent step outputs.

Neither write is preceded by the required sanitization step (`safe=$(printf '%s' "$VAR" | tr -d '\n\r')`).

Locations:

- `pre.sh:283`
- `pre.sh:291`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection issues in pre.sh:
1. GITHUB_PATH write (line ~283): Added sanitization of bin_dir via `safe_bin_dir=$(printf '%s' "${bin_dir}" | tr -d '\n\r')` before writing to $GITHUB_PATH.
2. GITHUB_OUTPUT heredoc (line ~291): Added sanitization for all 11 user-controlled values (tool, version, key, path, locked, git, tag, rev, features_flag, no_default_features_flag, all_features_flag) using `printf '%s' "${VAR}" | tr -d '\n\r'` before writing them to $GITHUB_OUTPUT. This prevents newline injection attacks where a malicious tool name or other input could inject additional PATH entries or arbitrary GITHUB_OUTPUT key=value pairs.

