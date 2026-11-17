<!-- markdownlint-disable -->

# Hardening Report: egor-tensin--setup-clang/v2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **egor-tensin--setup-clang/v2.0** was hardened automatically. 7 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): the pwsh run: block in step 'install' puts GitHub expressions straight into the script: `New-Variable os -Value '${{ runner.os }}'`, `New-Variable version -Value ('${{ inputs.version }}')` and `New-Variable x64 -Value ('${{ inputs.platform }}' -eq 'x64')`. An input value containing a single quote can break out of the PowerShell string literal and run any PowerShell code. Fix: move these values into env: entries and read them as $env:VAR.

Locations:

- `action.yml:31`
- `action.yml:36`
- `action.yml:38`

### script-injection (severity: high)

Rule (a): the second pwsh run: block (the link step) puts GitHub expressions straight into the script: `New-Variable os -Value '${{ runner.os }}'`, `New-Variable cc -Value ('${{ inputs.cc }}' -eq '1')`, `New-Variable clang -Value '${{ steps.install.outputs.clang }}'` and `New-Variable clangxx -Value '${{ steps.install.outputs.clangxx }}'`. The step outputs come from inputs.version, so they are also attacker-influenced. Fix: pass these values through env: and read them as $env:VAR.

Locations:

- `action.yml:146`
- `action.yml:151`
- `action.yml:153`
- `action.yml:154`

### github-env-injection (severity: high)

Step 'install' runs `echo "clang=$clang" >> $env:GITHUB_OUTPUT` and `echo "clangxx=$clangxx" >> $env:GITHUB_OUTPUT`. Here $clang and $clangxx are built from inputs.version (through Format-UpstreamVersion, which passes unknown values through unchanged). Newlines are not stripped before the write, so a version value containing a newline can inject extra output entries.

Locations:

- `action.yml:141`
- `action.yml:142`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.version }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:36`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.platform }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:38`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cc }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:151`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index 4d7f606..da9a6da 100644
--- a/action.yml
+++ b/action.yml
@@ -1,6 +1,5 @@
 name: Install Clang
 description: Install Clang & LLVM
-
 inputs:
   version:
     description: Version to install
@@ -14,7 +13,6 @@ inputs:
     description: Set up cc/clang/c++/clang++ executables
     required: false
     default: 1
-
 outputs:
   clang:
     description: clang binary name
@@ -22,20 +20,40 @@ outputs:
   clangxx:
     description: clang++ binary name
     value: '${{ steps.install.outputs.clangxx }}'
-
 runs:
   using: composite
   steps:
     - id: install
+      env:
+        INPUT_OS: ${{ runner.os }}
+        INPUT_VERSION: ${{ inputs.version }}
+        INPUT_PLATFORM: ${{ inputs.platform }}
       run: |
-        New-Variable os -Value '${{ runner.os }}' -Option Constant
+        New-Variable os -Value $env:INPUT_OS -Option Constant
 
         New-Variable linux_host -Value ($os -eq 'Linux') -Option Constant
         New-Variable windows_host -Value ($os -eq 'Windows') -Option Constant
 
-        New-Variable version -Value ('${{ inputs.version }}') -Option Constant
+        New-Variable version -Value ($env:INPUT_VERSION) -Option Constant
         New-Variable latest -Value ($version -eq 'latest') -Option Constant
-        New-Variable x64 -Value ('${{ inputs.platform }}' -eq 'x64') -Option Constant
+        New-Variable x64 -Value ($env:INPUT_PLATFORM -eq 'x64') -Option Constant
 
         function Locate-Choco {
             $path = Get-Command 'choco' -ErrorAction SilentlyContinue
@@ -138,20 +156,26 @@ runs:
             throw "Sorry, installing Clang is unsupported on $os"
         }
 
-        echo "clang=$clang"     >> $env:GITHUB_OUTPUT
-        echo "clangxx=$clangxx" >> $env:GITHUB_OUTPUT
+        $safe_clang   = $clang   -replace '[\r\n]', ''
+        $safe_clangxx = $clangxx -replace '[\r\n]', ''
+        echo "clang=$safe_clang"     >> $env:GITHUB_OUTPUT
+        echo "clangxx=$safe_clangxx" >> $env:GITHUB_OUTPUT
       shell: pwsh
-
-    - run: |
-        New-Variable os -Value '${{ runner.os }}' -Option Constant
+    - env:
+        INPUT_OS: ${{ runner.os }}
+        INPUT_CC: ${{ inputs.cc }}
+        INPUT_CLANG: ${{ steps.install.outputs.clang }}
+        INPUT_CLANGXX: ${{ steps.install.outputs.clangxx }}
+      run: |
+        New-Variable os -Value $env:INPUT_OS -Option Constant
 
         New-Variable linux_host -Value ($os -eq 'Linux') -Option Constant
         New-Variable windows_host -Value ($os -eq 'Windows') -Option Constant
 
-        New-Variable cc -Value ('${{ inputs.cc }}' -eq '1') -Option Constant
+        New-Variable cc -Value ($env:INPUT_CC -eq '1') -Option Constant
 
-        New-Variable clang -Value '${{ steps.install.outputs.clang }}' -Option Constant
-        New-Variable clangxx -Value '${{ steps.install.outputs.clangxx }}' -Option Constant
+        New-Variable clang -Value $env:INPUT_CLANG -Option Constant
+        New-Variable clangxx -Value $env:INPUT_CLANGXX -Option Constant
 
         function Link-Exe {
             param(
@@ -184,7 +208,6 @@ runs:
             }
         }
       shell: pwsh
-
 branding:
   icon: star
   color: green

```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, static-inline-injection, prior-fix

**Notes:**

I fixed every finding in action.yml by re-applying the earlier harden's security changes. Nothing was run or tested after the edits.

- **Script injection:** both pwsh steps no longer paste `${{ }}` values into the script. The install step now gets `runner.os`, `inputs.version` and `inputs.platform` through env: (INPUT_OS, INPUT_VERSION, INPUT_PLATFORM). The link step gets `runner.os`, `inputs.cc` and the clang/clangxx outputs from the install step the same way (INPUT_OS, INPUT_CC, INPUT_CLANG, INPUT_CLANGXX). Both scripts read these as `$env:VAR`.
- **Output injection:** before `clang` and `clangxx` are written to `$env:GITHUB_OUTPUT`, carriage returns and newlines are stripped with `-replace '[\r\n]', ''`.

The rest of the script logic is unchanged.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

In the Windows branch of the 'install' step (pwsh), $bin_dir is now cleaned with -replace '[\r\n]', '' before it is written to $GITHUB_PATH. This uses the same pattern as the existing $GITHUB_OUTPUT writes. No other changes were made.

