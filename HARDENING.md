<!-- markdownlint-disable -->

# Hardening Report: taiki-e--cache-cargo-install-action/v3.0.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **taiki-e--cache-cargo-install-action/v3.0.4** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In pre.sh, user-controlled input values are written to $GITHUB_PATH and $GITHUB_OUTPUT without the required newline-stripping sanitization (`printf '%s' ... | tr -d '\n\r'`). 

(1) `printf '%s\n' "${bin_dir}" >> "${GITHUB_PATH}"` — `bin_dir` is constructed as `${RUNNER_TOOL_CACHE}/${tool}/bin` where `tool` is read from `INPUT_TOOL` (set from `${{ inputs.tool }}`). An attacker-controlled `inputs.tool` value containing embedded newlines can inject additional entries into GITHUB_PATH.

(2) `cat >> "${GITHUB_OUTPUT}" << EOF` — the heredoc writes `tool`, `version`, `key`, `git`, `tag`, `rev`, `features_flag`, `no_default_features_flag`, and `all_features_flag` to GITHUB_OUTPUT. All of these are derived from user-controlled inputs (`inputs.tool`, `inputs.git`, `inputs.tag`, `inputs.rev`, `inputs.features`) without any `tr -d '\n\r'` sanitization. An attacker can inject arbitrary key=value pairs into GITHUB_OUTPUT by embedding newlines in these inputs.

Locations:

- `pre.sh:295`
- `pre.sh:305`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection vulnerabilities in pre.sh:

1. GITHUB_PATH write (line 295): Added `safe_bin_dir=$(printf '%s' "${bin_dir}" | tr -d '\n\r')` and used `safe_bin_dir` in the `printf` write to `$GITHUB_PATH`.

2. GITHUB_OUTPUT write (line 305): Replaced the heredoc `cat >> "${GITHUB_OUTPUT}" << EOF` with individual sanitized writes. Each user-controlled variable (tool, version, key, bin_dir, locked, git, tag, rev, features_flag, no_default_features_flag, all_features_flag) is first sanitized via `printf '%s' "${var}" | tr -d '\n\r'` into a `safe_*` variable, then written using `printf 'key=%s\n' "${safe_var}"` in a grouped `{ ... } >> "${GITHUB_OUTPUT}"` block. This prevents newline injection attacks where attacker-controlled inputs containing embedded newlines could inject arbitrary key=value pairs.

