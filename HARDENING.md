<!-- markdownlint-disable -->

# Hardening Report: taiki-e--cache-cargo-install-action/v3.0.8

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **taiki-e--cache-cargo-install-action/v3.0.8** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

pre.sh writes user-controlled input data to $GITHUB_PATH without sanitization. The variable `bin_dir` is constructed as `${RUNNER_TOOL_CACHE}/${tool}/bin` where `tool` is sourced directly from `INPUT_TOOL` (mapped from `inputs.tool` in action.yml). The write `printf '%s\n' "${bin_dir}" >> "${GITHUB_PATH}"` does not apply the required `tr -d '\n\r'` sanitization step before writing to the special environment file, allowing a newline injection attack that could add arbitrary entries to the runner's PATH.

Locations:

- `pre.sh:338`

### github-env-injection (severity: high)

pre.sh writes multiple user-controlled input values to $GITHUB_OUTPUT via a heredoc (`cat >> "${GITHUB_OUTPUT}" << EOF`) without sanitization. The values written include `tool` (from `inputs.tool`), `version`, `key`, `git` (from `inputs.git`), `tag` (from `inputs.tag`), `rev` (from `inputs.rev`), and feature flags derived from user inputs. A heredoc does not strip embedded newlines from variable values, so an attacker-controlled input containing a newline could inject additional key=value pairs into GITHUB_OUTPUT, potentially overwriting outputs consumed by later steps.

Locations:

- `pre.sh:344`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in pre.sh:
1. GITHUB_PATH write (line 338): Added `safe_bin_dir=$(printf '%s' "${bin_dir}" | tr -d '\n\r')` and used the sanitized variable in the printf write to GITHUB_PATH.
2. GITHUB_OUTPUT heredoc (line 344): Added sanitization for all 11 user-controlled variables (tool, version, key, path, locked, git, tag, rev, features_flag, no_default_features_flag, all_features_flag) using `printf '%s' "${var}" | tr -d '\n\r'` before the heredoc, and replaced raw variable references in the heredoc with their sanitized `safe_*` counterparts. This prevents newline injection attacks that could add arbitrary entries to the runner's PATH or inject additional key=value pairs into GITHUB_OUTPUT.

