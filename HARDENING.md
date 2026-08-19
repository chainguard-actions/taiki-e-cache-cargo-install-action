<!-- markdownlint-disable -->

# Hardening Report: taiki-e--cache-cargo-install-action/v3.0.7

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **taiki-e--cache-cargo-install-action/v3.0.7** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

pre.sh writes user-controlled input values to $GITHUB_PATH and $GITHUB_OUTPUT without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). 

(1) `printf '%s\n' "${bin_dir}" >> "${GITHUB_PATH}"` — `bin_dir` is constructed from `${tool}`, which is read from the `INPUT_TOOL` environment variable set by `inputs.tool`. An attacker-controlled crate name containing newlines could inject arbitrary entries into GITHUB_PATH.

(2) `cat >> "${GITHUB_OUTPUT}" <<EOF\ntool=${tool}\nversion=${version}\nkey=${key}\npath=${bin_dir}\nlocked=${locked}\ngit=${git}\ntag=${tag}\nrev=${rev}\nfeatures_flag=${features_flag}\n...\nEOF` — multiple values derived from user-controlled inputs (`inputs.tool`, `inputs.git`, `inputs.tag`, `inputs.rev`, `inputs.features`, etc.) are written to GITHUB_OUTPUT via a heredoc without sanitization. A newline embedded in any of these values would allow injection of additional output variables.

Locations:

- `pre.sh:278`
- `pre.sh:289`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection vulnerabilities in pre.sh:

1. GITHUB_PATH write (line ~278): Added sanitization of `bin_dir` (which contains user-controlled `tool` input) using `safe_bin_dir=$(printf '%s' "${bin_dir}" | tr -d '\n\r')` before writing to $GITHUB_PATH.

2. GITHUB_OUTPUT heredoc (line ~289): Added sanitization for all 10 user-controlled values written to $GITHUB_OUTPUT (`tool`, `version`, `key`, `bin_dir`, `locked`, `git`, `tag`, `rev`, `features_flag`, `no_default_features_flag`, `all_features_flag`) using `printf '%s' "${VAR}" | tr -d '\n\r'` for each, then using the sanitized `safe_*` variables in the heredoc. This prevents newline injection attacks where an attacker-controlled crate name or other input containing newlines could inject arbitrary entries into GITHUB_PATH or additional output variables into GITHUB_OUTPUT.

