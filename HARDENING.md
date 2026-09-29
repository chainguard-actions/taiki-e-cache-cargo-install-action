<!-- markdownlint-disable -->

# Hardening Report: taiki-e--cache-cargo-install-action/v3.0.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **taiki-e--cache-cargo-install-action/v3.0.4** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

pre.sh writes user-controlled input values to $GITHUB_PATH without sanitization. The variable `bin_dir` is constructed as `${RUNNER_TOOL_CACHE}/${tool}/bin` where `tool` is derived from `INPUT_TOOL` (mapped from `inputs.tool` in action.yml). The write `printf '%s\n' "${bin_dir}" >> "${GITHUB_PATH}"` does not apply `tr -d '\n\r'` before writing, allowing a crafted tool name containing newlines to inject arbitrary entries into the runner's PATH.

Locations:

- `pre.sh:287`

### github-env-injection (severity: high)

pre.sh writes multiple user-controlled input values to $GITHUB_OUTPUT via a heredoc (`cat >> "${GITHUB_OUTPUT}" << EOF`) without sanitization. The values written include `tool` (from `inputs.tool`), `version`, `key`, `git` (from `inputs.git`), `tag` (from `inputs.tag`), `rev` (from `inputs.rev`), `features_flag` (from `inputs.features`), `no_default_features_flag`, and `all_features_flag`. None of these are passed through `printf '%s' ... | tr -d '\n\r'` before being written, allowing a crafted input containing newlines to inject arbitrary key=value pairs into GITHUB_OUTPUT, potentially overwriting subsequent step outputs.

Locations:

- `pre.sh:295`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in pre.sh:

1. Line 287 (GITHUB_PATH): Added sanitization of `bin_dir` (derived from user-controlled `INPUT_TOOL`) before writing to `$GITHUB_PATH`. Now uses `safe_bin_dir=$(printf '%s' "${bin_dir}" | tr -d '\n\r')` and writes the sanitized value.

2. Line 295 (GITHUB_OUTPUT): Replaced the unsanitized heredoc (`cat >> "${GITHUB_OUTPUT}" << EOF`) with individual sanitized `printf` calls. Each user-controlled value (`tool`, `version`, `key`, `git`, `tag`, `rev`, `features_flag`, `no_default_features_flag`, `all_features_flag`, `locked`) is first passed through `printf '%s' "${var}" | tr -d '\n\r'` to strip newlines before being written to `$GITHUB_OUTPUT`, preventing newline injection attacks that could inject arbitrary key=value pairs.

