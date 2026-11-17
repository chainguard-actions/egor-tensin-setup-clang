<!-- markdownlint-disable -->

# Hardening Report: egor-tensin--setup-clang/v2.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **egor-tensin--setup-clang/v2.3** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): the 'install' step's pwsh run block puts a GitHub expression directly into the script: "$linux_host = '${{ runner.os }}' -eq 'Linux'" and "$windows_host = '${{ runner.os }}' -eq 'Windows'". Read it from the environment instead with $env:RUNNER_OS.

Locations:

- `action.yml:34`
- `action.yml:35`

### script-injection (severity: high)

Rule (a): the second step's pwsh run block (the one that creates the cc/clang links) puts a GitHub expression directly into the script: "$linux_host = '${{ runner.os }}' -eq 'Linux'" and "$windows_host = '${{ runner.os }}' -eq 'Windows'". Use $env:RUNNER_OS instead.

Locations:

- `action.yml:177`
- `action.yml:178`

### github-env-injection (severity: high)

The 'install' step writes $clang, $clangxx and $lld to $env:GITHUB_OUTPUT (echo "clang=$clang" >> $env:GITHUB_OUTPUT and so on). These values come from inputs.version, passed in through env ARG_VERSION ($clang = "$clang-$pkg_version"). Nothing removes CR/LF characters before the write, so a version value containing a newline could add extra output entries. Strip \r and \n from the values first, for example with -replace '[\r\n]', ''.

Locations:

- `action.yml:167`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index 1a0af3c..24bcfc3 100644
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
@@ -164,6 +181,9 @@ runs:
             throw "Sorry, installing Clang is unsupported on $os"
         }
 
+        $clang   = $clang -replace '[\r\n]', ''
+        $clangxx = $clangxx -replace '[\r\n]', ''
+        $lld     = $lld -replace '[\r\n]', ''
         echo "clang=$clang"     >> $env:GITHUB_OUTPUT
         echo "clangxx=$clangxx" >> $env:GITHUB_OUTPUT
         echo "lld=$lld"         >> $env:GITHUB_OUTPUT
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

I changed action.yml in two places, the same changes the previous harden made. Nothing has been run yet.

1. Both pwsh steps put `${{ runner.os }}` straight into the script. They now read `$env:RUNNER_OS` instead.
2. Before `$clang`, `$clangxx` and `$lld` are written to `$env:GITHUB_OUTPUT`, CR and LF characters are now stripped from each with `-replace '[\r\n]', ''`.

