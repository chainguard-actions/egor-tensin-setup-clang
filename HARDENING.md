<!-- markdownlint-disable -->

# Hardening Report: egor-tensin--setup-clang/v2.3.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **egor-tensin--setup-clang/v2.3.3** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): a GitHub Actions expression is placed straight into the pwsh run: script of the 'install' step: `$linux_host = '${{ runner.os }}' -eq 'Linux'` and `$windows_host = '${{ runner.os }}' -eq 'Windows'`. Read $env:RUNNER_OS instead.

Locations:

- `action.yml:34`
- `action.yml:35`

### script-injection (severity: high)

Rule (a): a GitHub Actions expression is placed straight into the pwsh run: script of the second (link) step: `$linux_host = '${{ runner.os }}' -eq 'Linux'` and `$windows_host = '${{ runner.os }}' -eq 'Windows'`. Read $env:RUNNER_OS instead.

Locations:

- `action.yml:177`
- `action.yml:178`

### github-env-injection (severity: high)

The 'install' step writes values built from untrusted input to $env:GITHUB_OUTPUT without removing newlines first. When version is not 'latest', $clang, $clangxx and $lld are built as "$clang-$pkg_version", and $pkg_version comes from inputs.version (env ARG_VERSION). They are then written with `echo "clang=$clang" >> $env:GITHUB_OUTPUT` (and the same for clangxx and lld). Strip CR/LF from the version (e.g. -replace '[\r\n]','') or check that it is numeric before using it.

Locations:

- `action.yml:167`
- `action.yml:168`
- `action.yml:169`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index 1a0af3c..93346e6 100644
--- a/action.yml
+++ b/action.yml
@@ -26,13 +26,30 @@ outputs:
 runs:
   using: composite
   steps:
     - id: install
       env:
         ARG_VERSION: ${{ inputs.version }}
         ARG_PLATFORM: ${{ inputs.platform }}
       run: |
-        $linux_host = '${{ runner.os }}' -eq 'Linux'
-        $windows_host = '${{ runner.os }}' -eq 'Windows'
+        $linux_host = $env:RUNNER_OS -eq 'Linux'
+        $windows_host = $env:RUNNER_OS -eq 'Windows'
 
         $version = (Get-ChildItem env:ARG_VERSION).Value
         $latest = $version -eq 'latest'
@@ -164,9 +181,12 @@ runs:
             throw "Sorry, installing Clang is unsupported on $os"
         }
 
-        echo "clang=$clang"     >> $env:GITHUB_OUTPUT
-        echo "clangxx=$clangxx" >> $env:GITHUB_OUTPUT
-        echo "lld=$lld"         >> $env:GITHUB_OUTPUT
+        $safe_clang   = $clang   -replace '[\r\n]', ''
+        $safe_clangxx = $clangxx -replace '[\r\n]', ''
+        $safe_lld     = $lld     -replace '[\r\n]', ''
+        echo "clang=$safe_clang"     >> $env:GITHUB_OUTPUT
+        echo "clangxx=$safe_clangxx" >> $env:GITHUB_OUTPUT
+        echo "lld=$safe_lld"         >> $env:GITHUB_OUTPUT
       shell: pwsh
 
     - env:
@@ -174,8 +194,8 @@ runs:
         CLANG: ${{ steps.install.outputs.clang }}
         CLANGXX: ${{ steps.install.outputs.clangxx }}
       run: |
-        $linux_host = '${{ runner.os }}' -eq 'Linux'
-        $windows_host = '${{ runner.os }}' -eq 'Windows'
+        $linux_host = $env:RUNNER_OS -eq 'Linux'
+        $windows_host = $env:RUNNER_OS -eq 'Windows'
 
         $cc = (Get-ChildItem env:ARG_CC).Value -eq '1'
 

```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, prior-fix

**Notes:**

In both pwsh steps, '${{ runner.os }}' now reads $env:RUNNER_OS. The install step now removes CR/LF from $clang, $clangxx and $lld (-replace '[\r\n]','') before writing them to $env:GITHUB_OUTPUT. This puts back the changes from the previous harden. Nothing has been run or tested yet.

