<!-- markdownlint-disable -->

# Hardening Report: taiki-e--cache-cargo-install-action/v3.0.8

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **taiki-e--cache-cargo-install-action/v3.0.8** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

pre.sh is run by the 'Prepare' steps in action.yml. It writes values taken from action inputs to special environment files without the required `printf '%s' ... | tr -d '\n\r'` sanitization. (1) Line 379: `printf '%s\n' "${bin_dir}" >>"${GITHUB_PATH}"`. Here bin_dir="${RUNNER_TOOL_CACHE}/${tool}/bin" and tool comes from INPUT_TOOL (inputs.tool). A newline in the input adds extra PATH entries. (2) Line 387: `cat >>"${GITHUB_OUTPUT}" <<EOF` writes tool=${tool}, git=${git}, tag=${tag}, rev=${rev}, key=${key}, path=${bin_dir} and features_flag=--features=${features} as key=value lines. tool, git, tag, rev and features are never checked for newlines. An embedded newline can inject or override step outputs such as path, key, git or locked. The later Install step and actions/cache use these outputs. Fix: strip CR/LF from each value with `printf '%s' "$v" | tr -d '\n\r'` before writing it, or reject values that contain newlines.

Locations:

- `pre.sh:379`
- `pre.sh:387`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index 0e5484f..65f3f34 100644
--- a/action.yml
+++ b/action.yml
@@ -1,6 +1,5 @@
 name: cache-cargo-install-action
 description: GitHub Action for cargo install with cache
-
 inputs:
   tool:
     description: Crate to install
@@ -33,13 +32,29 @@ inputs:
     description: Pass `--all-features` to `cargo install`
     required: false
     default: 'false'
-
 # Note:
 # - inputs.* should be manually mapped to INPUT_* due to https://github.com/actions/runner/issues/665
 # - For context containing path, use GITHUB_*/RUNNER_* instead of github.*/runner.* due to https://github.com/actions/runner/issues/2185
 runs:
   using: composite
   steps:
     - name: Prepare (non-Windows)
       id: pre-unix
       run: |
@@ -122,7 +137,7 @@ runs:
       if: runner.os == 'Windows'
     - name: Restore Cache
       id: cache
       uses: actions/cache@55cc8345863c7cc4c66a329aec7e433d2d1c52a9 # v6.1.0
       with:
         path: ${{ runner.os == 'Windows' && steps.pre-windows.outputs.path || steps.pre-unix.outputs.path }}
         key: ${{ runner.os == 'Windows' && steps.pre-windows.outputs.key || steps.pre-unix.outputs.key }}

```

Locations:

- `action.yml`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed pre.sh without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/pre.sh b/pre.sh
index 2d0ee86..7b2d90c 100755
--- a/pre.sh
+++ b/pre.sh
@@ -376,7 +376,8 @@ if [[ "${version}" == "latest" ]] || [[ -n "${fetch}" ]]; then
 fi
 
 bin_dir="${RUNNER_TOOL_CACHE}/${tool}/bin"
-printf '%s\n' "${bin_dir}" >>"${GITHUB_PATH}"
+safe_bin_dir=$(printf '%s' "${bin_dir}" | tr -d '\n\r')
+printf '%s\n' "${safe_bin_dir}" >>"${GITHUB_PATH}"
 
 if [[ -n "${git}" ]]; then
   key="${tool}-git-${tag:+"tag-${tag}"}${rev:+"rev-${rev}"}-${host_arch}-${host_os}${locked_key}-features-${features:-default}-no-default-features-${no_default_features}-all-features-${all_features}"
@@ -384,16 +385,26 @@ else
   features="${features//,/-}" # Commas are not allowed in cache key names
   key="${tool}-${version}-${host_arch}-${host_os}${locked_key}-features-${features:-default}-no-default-features-${no_default_features}-all-features-${all_features}"
 fi
+safe_tool=$(printf '%s' "${tool}" | tr -d '\n\r')
+safe_version=$(printf '%s' "${version}" | tr -d '\n\r')
+safe_key=$(printf '%s' "${key}" | tr -d '\n\r')
+safe_locked=$(printf '%s' "${locked}" | tr -d '\n\r')
+safe_git=$(printf '%s' "${git}" | tr -d '\n\r')
+safe_tag=$(printf '%s' "${tag}" | tr -d '\n\r')
+safe_rev=$(printf '%s' "${rev}" | tr -d '\n\r')
+safe_features_flag=$(printf '%s' "${features_flag}" | tr -d '\n\r')
+safe_no_default_features_flag=$(printf '%s' "${no_default_features_flag}" | tr -d '\n\r')
+safe_all_features_flag=$(printf '%s' "${all_features_flag}" | tr -d '\n\r')
 cat >>"${GITHUB_OUTPUT}" <<EOF
-tool=${tool}
-version=${version}
-key=${key}
-path=${bin_dir}
-locked=${locked}
-git=${git}
-tag=${tag}
-rev=${rev}
-features_flag=${features_flag}
-no_default_features_flag=${no_default_features_flag}
-all_features_flag=${all_features_flag}
+tool=${safe_tool}
+version=${safe_version}
+key=${safe_key}
+path=${safe_bin_dir}
+locked=${safe_locked}
+git=${safe_git}
+tag=${safe_tag}
+rev=${safe_rev}
+features_flag=${safe_features_flag}
+no_default_features_flag=${safe_no_default_features_flag}
+all_features_flag=${safe_all_features_flag}
 EOF

```

Locations:

- `pre.sh`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, prior-fix

**Notes:**

I fixed pre.sh so it strips CR/LF from every value it writes to $GITHUB_PATH and $GITHUB_OUTPUT. That covers the bin_dir PATH entry and the outputs tool, version, key, path, locked, git, tag, rev and the three feature flags. This is the same change the previous harden made to pre.sh. Each value is a plain variable, so a failed command substitution can't be hidden. Nothing was run to test it. I made no changes to action.yml. The previous patch for that file is cut off: the part shown only removes blank lines, and its first hunk claims 16 added lines that aren't in the text, so I couldn't check them. In the current file, every input already goes through env:, no run: script contains a ${{ }} expression, and actions/cache is pinned to SHA 55cc8345863c7cc4c66a329aec7e433d2d1c52a9 (v6.1.0).

