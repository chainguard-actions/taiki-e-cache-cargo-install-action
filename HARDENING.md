<!-- markdownlint-disable -->

# Hardening Report: taiki-e--cache-cargo-install-action/v3.0.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **taiki-e--cache-cargo-install-action/v3.0.5** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references in workflow files are pinned to mutable branch or tag refs instead of immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks if those refs are updated maliciously.

Failing references:
- `uses: taiki-e/github-actions/.github/workflows/tidy.yml@main` (branch ref)
- `uses: taiki-e/checkout-action@v1` (tag ref, appears twice)
- `uses: taiki-e/github-actions/install-rust@stable` (branch ref, appears twice)
- `uses: taiki-e/github-actions/.github/workflows/action-release.yml@main` (branch ref)

Locations:

- `.github/workflows/ci.yml:26`
- `.github/workflows/ci.yml:68`
- `.github/workflows/ci.yml:69`
- `.github/workflows/ci.yml:152`
- `.github/workflows/ci.yml:157`
- `.github/workflows/release.yml:29`

### github-env-injection (severity: high)

In pre.sh, user-controlled values derived from action inputs (INPUT_TOOL, INPUT_GIT, INPUT_TAG, INPUT_REV, INPUT_FEATURES, etc.) are written to $GITHUB_PATH and $GITHUB_OUTPUT without the required newline-stripping sanitization (`printf '%s' "$VAR" | tr -d '\n\r'`). An attacker-controlled input containing a newline character could inject arbitrary key=value pairs into the runner's environment or path.

1. `printf '%s\n' "${bin_dir}" >> "${GITHUB_PATH}"` — `bin_dir` is constructed from `tool` (user-supplied via INPUT_TOOL) without sanitization.
2. The heredoc `cat >> "${GITHUB_OUTPUT}" <<EOF` writes `tool`, `version`, `key`, `git`, `tag`, `rev`, `features_flag`, etc. — all derived from user inputs — without sanitization.

Locations:

- `pre.sh:315`
- `pre.sh:328`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, github-env-injection

**Notes:**

Fixed all 6 unpinned `uses:` references in .github/workflows/ci.yml and .github/workflows/release.yml by pinning them to full 40-character commit SHAs (taiki-e/github-actions@3da7d39bb26122d232edd731a1813082711e3037 for main/stable refs, taiki-e/checkout-action@7d1e50e93dc4fb3bba58f85018fadf77898aee8b for v1). Fixed github-env-injection in pre.sh by sanitizing all user-controlled values (tool, version, key, bin_dir, locked, git, tag, rev, features_flag, no_default_features_flag, all_features_flag) with `printf '%s' "$VAR" | tr -d '\n\r'` before writing to $GITHUB_PATH and $GITHUB_OUTPUT.

