<!-- markdownlint-disable -->

# Hardening Report: taiki-e--cache-cargo-install-action/v3.0.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **taiki-e--cache-cargo-install-action/v3.0.6** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

pre.sh (run from the 'Prepare' steps in action.yml) copies user inputs into GitHub's special environment files without removing newlines first. (1) Line 379: `printf '%s\n' "${bin_dir}" >>"${GITHUB_PATH}"`. Here bin_dir="${RUNNER_TOOL_CACHE}/${tool}/bin" and tool comes from INPUT_TOOL (inputs.tool). The script only rejects ',' in the tool name, so a newline in inputs.tool adds attacker-chosen PATH entries. (2) Line 387: `cat >>"${GITHUB_OUTPUT}" <<EOF` writes tool=${tool}, git=${git}, tag=${tag}, rev=${rev}, key=${key} and features_flag=${features_flag} one per line. These come from inputs.tool, inputs.git, inputs.tag, inputs.rev and inputs.features with no `printf '%s' ... | tr -d '\n\r'` step. A newline in any of them adds or overrides step outputs such as path, key, locked or features_flag, which the later Restore Cache and Install steps use. Fix: strip CR/LF from each value before writing, e.g. `tool=$(printf '%s' "$tool" | tr -d '\n\r')`, or reject inputs that contain newlines.

Locations:

- `pre.sh:379`
- `pre.sh:387`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index f4cc405..b0640c7 100644
--- a/action.yml
+++ b/action.yml
@@ -1,6 +1,5 @@
 name: cache-cargo-install-action
 description: GitHub Action for cargo install with cache
-
 inputs:
   tool:
     description: Crate to install
@@ -14,6 +13,7 @@ inputs:
       Install from the specified Git URL
 
       This currently requires one of 'tag' or 'rev' input option.
+
     required: false
   tag:
     description: Tag to use when installing from git
@@ -33,13 +33,29 @@ inputs:
     description: Pass `--all-features` to `cargo install`
     required: false
     default: 'false'
-
 # Note:
 # - inputs.* should be manually mapped to INPUT_* due to https://github.com/actions/runner/issues/665
 # - Use GITHUB_*/RUNNER_* instead of github.*/runner.* due to https://github.com/actions/runner/issues/2185
 runs:
   using: composite
   steps:
     - name: Prepare (non-Windows)
       id: pre-unix
       run: |
@@ -113,7 +129,7 @@ runs:
       if: runner.os == 'Windows'
     - name: Restore Cache
       id: cache
       uses: actions/cache@668228422ae6a00e4ad889ee87cd7109ec5666a7 # v5.0.4
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
index 4c557eb..19d228d 100755
--- a/pre.sh
+++ b/pre.sh
@@ -375,7 +375,8 @@ if [[ "${version}" == "latest" ]] || [[ -n "${fetch}" ]]; then
 fi
 
 bin_dir="${RUNNER_TOOL_CACHE}/${tool}/bin"
-printf '%s\n' "${bin_dir}" >>"${GITHUB_PATH}"
+safe_bin_dir=$(printf '%s' "${bin_dir}" | tr -d '\n\r')
+printf '%s\n' "${safe_bin_dir}" >>"${GITHUB_PATH}"
 
 if [[ -n "${git}" ]]; then
   key="${tool}-git-${tag:+"tag-${tag}"}${rev:+"rev-${rev}"}-${host_arch}-${host_os}${locked_key}-features-${features:-default}-no-default-features-${no_default_features}-all-features-${all_features}"
@@ -383,16 +384,26 @@ else
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

I fixed the high-severity finding in pre.sh (newlines in inputs could reach GitHub's PATH and step-output files). I didn't run the action or any tests after the change.

- **PATH file:** pre.sh now removes newlines (CR/LF) from the tool directory path before adding it to $GITHUB_PATH.
- **Step outputs:** every value written to $GITHUB_OUTPUT has newlines removed first: tool, version, key, path, locked, git, tag, rev and the three features flags. This matches the patch from the previous harden.
- **action.yml:** I left it unchanged. The previous harden's patch to it only moved blank lines around and kept the same actions/cache pin. The file already uses that pinned commit and passes all inputs through env:, so there was no security change to re-apply.

