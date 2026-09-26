<!-- markdownlint-disable -->

# Hardening Report: taiki-e--cache-cargo-install-action/v3.0.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **taiki-e--cache-cargo-install-action/v3.0.4** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

pre.sh writes `bin_dir` (derived from user-controlled `INPUT_TOOL` / `tool`) to `$GITHUB_PATH` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). An attacker-controlled crate name containing newlines could inject arbitrary entries into the runner's PATH. The offending line is: `printf '%s\n' "${bin_dir}" >> "${GITHUB_PATH}"`

Locations:

- `pre.sh:314`

### github-env-injection (severity: high)

pre.sh writes multiple user-controlled values (`tool`, `version`, `key`, `git`, `tag`, `rev`, `features_flag`, etc.) to `$GITHUB_OUTPUT` via a heredoc (`cat >> "${GITHUB_OUTPUT}" << EOF`) without any sanitization (`tr -d '\n\r'`). Values like `tool`, `tag`, and `rev` originate from `INPUT_TOOL`, `INPUT_TAG`, and `INPUT_REV` — all caller-supplied inputs. A newline embedded in any of these values could inject additional key=value pairs into GITHUB_OUTPUT, poisoning downstream step outputs.

Locations:

- `pre.sh:328`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in pre.sh:
1. Line ~314 (GITHUB_PATH): Added sanitization of `bin_dir` (derived from user-controlled `INPUT_TOOL`) using `safe_bin_dir=$(printf '%s' "${bin_dir}" | tr -d '\n\r')` before writing to $GITHUB_PATH.
2. Line ~328 (GITHUB_OUTPUT heredoc): Replaced direct use of user-controlled variables in the heredoc with sanitized versions. Each value (tool, version, key, path/bin_dir, locked, git, tag, rev, features_flag, no_default_features_flag, all_features_flag) is now sanitized via `printf '%s' "${var}" | tr -d '\n\r'` before being written to $GITHUB_OUTPUT, preventing newline injection attacks.

