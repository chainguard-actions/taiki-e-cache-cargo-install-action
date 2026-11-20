<!-- markdownlint-disable -->

# Hardening Report: taiki-e--cache-cargo-install-action/v3.0.7

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **taiki-e--cache-cargo-install-action/v3.0.7** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The supporting script pre.sh, which the 'Prepare' steps in action.yml run, writes values taken from action inputs to special environment files without removing newlines. (1) `printf '%s\n' "${bin_dir}" >>"${GITHUB_PATH}"` writes bin_dir="${RUNNER_TOOL_CACHE}/${tool}/bin", where tool comes from INPUT_TOOL (inputs.tool). The only check on tool rejects ',' and splits on '@'. A newline in the value adds extra PATH entries. (2) `cat >>"${GITHUB_OUTPUT}" <<EOF` writes tool=${tool}, key=${key}, git=${git}, tag=${tag}, rev=${rev} and features_flag=--features=${features} as key=value lines. These come from inputs.tool/git/tag/rev/features with no `printf '%s' ... | tr -d '\n\r'` step and no random delimiter. A newline in any of these inputs can inject or override step outputs such as path, key, locked or features_flag, which later steps use for the cache and the install command. The heredoc label EOF here is only shell syntax. Each value is written as a single-line key=value entry, so the newlines need to be stripped.

Locations:

- `pre.sh:318`
- `pre.sh:327`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index 338c91a..18abe12 100644
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
@@ -122,7 +138,7 @@ runs:
       if: runner.os == 'Windows'
     - name: Restore Cache
       id: cache
       uses: actions/cache@27d5ce7f107fe9357f9df03efb73ab90386fccae # v5.0.5
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

I fixed the newline injection in pre.sh and redid the previous harden's change there. action.yml needed no edit.

pre.sh: the path written to GITHUB_PATH now has newlines and carriage returns removed first. The same goes for every value written to GITHUB_OUTPUT (tool, version, key, path, locked, git, tag, rev and the three feature flags). Each one is cleaned with printf '%s' | tr -d '\n\r' and stored in its own variable before the write, so a newline in an input can no longer add or override an output. The commands that produce these values are unchanged, so a failing command still stops the script as before.

action.yml: the previous harden only changed blank lines there and pinned actions/cache to a commit SHA (# v5.0.5). That pin is already in the file, so there was nothing to redo.

