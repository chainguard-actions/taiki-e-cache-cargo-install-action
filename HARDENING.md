<!-- markdownlint-disable -->

# Hardening Report: taiki-e--cache-cargo-install-action/v3.0.8

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **taiki-e--cache-cargo-install-action/v3.0.8** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

pre.sh writes user-controlled input values to $GITHUB_PATH and $GITHUB_OUTPUT without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`).

(1) GITHUB_PATH write: `printf '%s\n' "${bin_dir}" >> "${GITHUB_PATH}"` — `bin_dir` is constructed as `${RUNNER_TOOL_CACHE}/${tool}/bin` where `tool` is read from the `INPUT_TOOL` environment variable, which is set from `${{ inputs.tool }}` in action.yml. An attacker-controlled crate name containing a newline could inject an arbitrary additional path entry.

(2) GITHUB_OUTPUT heredoc write: `cat >> "${GITHUB_OUTPUT}" <<EOF` writes multiple values derived from user inputs (`tool`, `version`, `key`, `git`, `tag`, `rev`, `features_flag`, etc.) without sanitization. A newline embedded in any of these values (e.g. `inputs.tool`, `inputs.git`, `inputs.tag`, `inputs.rev`, `inputs.features`) could inject additional `key=value` pairs into $GITHUB_OUTPUT, potentially overwriting outputs consumed by downstream steps.

Locations:

- `pre.sh:296`
- `pre.sh:305`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed pre.sh to sanitize all user-controlled values before writing to $GITHUB_PATH and $GITHUB_OUTPUT:

1. GITHUB_PATH write (line 296): Added `safe_bin_dir=$(printf '%s' "${bin_dir}" | tr -d '\n\r')` and used `safe_bin_dir` in the printf write instead of the raw `bin_dir`.

2. GITHUB_OUTPUT heredoc write (line 305): Added sanitized versions of all user-controlled variables (tool, version, key, locked, git, tag, rev, features_flag, no_default_features_flag, all_features_flag, bin_dir) using `printf '%s' "${VAR}" | tr -d '\n\r'`, then used the safe_ prefixed variables in the heredoc write. This prevents newline injection attacks where attacker-controlled input values containing embedded newlines could inject additional key=value pairs into $GITHUB_OUTPUT or additional path entries into $GITHUB_PATH.

