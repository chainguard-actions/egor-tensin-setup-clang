<!-- markdownlint-disable -->

# Hardening Report: egor-tensin--setup-clang/v2.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **egor-tensin--setup-clang/v2.1** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple ${{ ... }} expressions are interpolated directly inside run: shell command strings in action.yml. This means GitHub Actions performs YAML template substitution before the shell ever sees the value, allowing an attacker-controlled input to inject arbitrary PowerShell commands.

Step 1 (id: install):
- Line 31: `New-Variable os -Value '${{ runner.os }}' -Option Constant`
- Line 36: `New-Variable version -Value ('${{ inputs.version }}') -Option Constant`
- Line 38: `New-Variable x64 -Value ('${{ inputs.platform }}' -eq 'x64') -Option Constant`

Step 2 (unnamed):
- Line 125: `New-Variable os -Value '${{ runner.os }}' -Option Constant`
- Line 129: `New-Variable cc -Value ('${{ inputs.cc }}' -eq '1') -Option Constant`
- Line 131: `New-Variable clang -Value '${{ steps.install.outputs.clang }}' -Option Constant`
- Line 132: `New-Variable clangxx -Value '${{ steps.install.outputs.clangxx }}' -Option Constant`

All of these should be passed via env: variables and referenced as PowerShell environment variables (e.g. $env:INPUT_VERSION) instead of being interpolated directly.

Locations:

- `action.yml:31`
- `action.yml:36`
- `action.yml:38`
- `action.yml:125`
- `action.yml:129`
- `action.yml:131`
- `action.yml:132`

### github-env-injection (severity: high)

The first run: block (id: install) writes $clang and $clangxx to $GITHUB_OUTPUT without sanitization. Both variables are derived from ${{ inputs.version }} (user-controlled input) — when a non-latest version is requested, $clang becomes "clang-$pkg_version" and $clangxx becomes "clangxx-$pkg_version" where $pkg_version is derived from the user-supplied inputs.version. An attacker could inject newlines into inputs.version to write arbitrary key=value pairs into GITHUB_OUTPUT (and transitively into GITHUB_ENV if a downstream step uses the output). The required sanitization step (stripping newlines before writing) is absent.

- Line 120: `echo "clang=$clang" >> $env:GITHUB_OUTPUT`
- Line 121: `echo "clangxx=$clangxx" >> $env:GITHUB_OUTPUT`

Locations:

- `action.yml:120`
- `action.yml:121`

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

- `action.yml:155`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, static-inline-injection

**Notes:**

Fixed action.yml by: (1) Moving all ${{ runner.os }}, ${{ inputs.version }}, ${{ inputs.platform }}, ${{ inputs.cc }}, ${{ steps.install.outputs.clang }}, and ${{ steps.install.outputs.clangxx }} expressions from run: blocks into env: blocks on both steps, referencing them as $env:INPUT_OS, $env:INPUT_VERSION, $env:INPUT_PLATFORM, $env:INPUT_CC, $env:INPUT_CLANG, $env:INPUT_CLANGXX in PowerShell. (2) Added newline sanitization before writing to $GITHUB_OUTPUT: $safe_clang = $clang -replace '[\r\n]', '' and $safe_clangxx = $clangxx -replace '[\r\n]', '' to prevent github-env-injection. All ${{ }} expressions now only appear in safe locations (env: blocks and output value: declarations).

