<!-- markdownlint-disable -->

# Hardening Report: taiki-e--cache-cargo-install-action/v3.0.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **taiki-e--cache-cargo-install-action/v3.0.6** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

pre.sh writes values derived from untrusted inputs to $GITHUB_PATH and $GITHUB_OUTPUT without the required sanitization (`printf '%s' ... | tr -d '\n\r'`). 

(1) `printf '%s\n' "${bin_dir}" >> "${GITHUB_PATH}"` — `bin_dir` is constructed as `${RUNNER_TOOL_CACHE}/${tool}/bin` where `tool` comes directly from `INPUT_TOOL` (a caller-controlled input). A newline embedded in the tool name would inject an arbitrary extra path entry.

(2) The heredoc `cat >> "${GITHUB_OUTPUT}" <<EOF` writes `tool`, `git`, `tag`, `rev`, `version`, `key`, `features_flag`, etc. — all derived from untrusted inputs (INPUT_TOOL, INPUT_GIT, INPUT_TAG, INPUT_REV, INPUT_FEATURES, etc.) — directly to $GITHUB_OUTPUT. A newline in any of these values would inject additional key=value pairs into the output context, potentially overwriting outputs consumed by downstream steps.

Locations:

- `pre.sh:280`
- `pre.sh:289`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed pre.sh to sanitize all values derived from untrusted inputs before writing to $GITHUB_PATH and $GITHUB_OUTPUT. (1) For the GITHUB_PATH write: computed safe_bin_dir using `printf '%s' "${bin_dir}" | tr -d '\n\r'` and used it in the printf write. (2) For the GITHUB_OUTPUT heredoc: sanitized all 11 output values (tool, version, key, path/bin_dir, locked, git, tag, rev, features_flag, no_default_features_flag, all_features_flag) with `printf '%s' ... | tr -d '\n\r'` before writing them to the heredoc. This prevents newline injection attacks via caller-controlled inputs like INPUT_TOOL, INPUT_GIT, INPUT_TAG, INPUT_REV, and INPUT_FEATURES.

