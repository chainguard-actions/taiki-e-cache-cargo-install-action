<!-- markdownlint-disable -->

# Hardening Report: taiki-e--cache-cargo-install-action/v3.0.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **taiki-e--cache-cargo-install-action/v3.0.4** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

pre.sh writes the value of `${bin_dir}` (derived from `inputs.tool` via the `INPUT_TOOL` env var) to `$GITHUB_PATH` using `printf '%s\n' "${bin_dir}" >> "${GITHUB_PATH}"` without first sanitizing newlines with `tr -d '\n\r'`. An attacker-controlled `inputs.tool` value containing embedded newlines could inject arbitrary additional entries into `$GITHUB_PATH`.

Locations:

- `pre.sh:298`

### github-env-injection (severity: high)

pre.sh writes multiple untrusted `inputs.*`-derived values (`tool`, `version`, `key`, `git`, `tag`, `rev`, `features_flag`, etc.) to `$GITHUB_OUTPUT` via a heredoc (`cat >> "${GITHUB_OUTPUT}" <<EOF`) without sanitizing newlines with `tr -d '\n\r'`. An attacker-controlled input (e.g. `inputs.tool`, `inputs.git`, `inputs.tag`, `inputs.rev`) containing embedded newlines could inject additional key=value pairs into `$GITHUB_OUTPUT`, potentially overwriting subsequent outputs consumed by downstream steps.

Locations:

- `pre.sh:313`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in pre.sh:
1. GITHUB_PATH write (line 298): Added `safe_bin_dir=$(printf '%s' "${bin_dir}" | tr -d '\n\r')` and used `safe_bin_dir` in the printf to GITHUB_PATH, preventing newline injection via the user-controlled `tool` input embedded in `bin_dir`.
2. GITHUB_OUTPUT heredoc (line 313): Added sanitization for all 11 user-input-derived values (tool, version, key, path, locked, git, tag, rev, features_flag, no_default_features_flag, all_features_flag) using `printf '%s' "${VAR}" | tr -d '\n\r'` before writing them to GITHUB_OUTPUT, preventing embedded newlines from injecting additional key=value pairs.

