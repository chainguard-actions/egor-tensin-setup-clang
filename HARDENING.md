<!-- markdownlint-disable -->

# Hardening Report: egor-tensin--setup-clang/v2.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **egor-tensin--setup-clang/v2.2** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple `${{ ... }}` expressions are directly interpolated inside `run:` shell (pwsh) blocks in action.yml. This allows script injection because GitHub Actions performs template substitution before the shell ever sees the string.

Step 1 (id: install):
- Line 30: `New-Variable os -Value '${{ runner.os }}' -Option Constant`
- Line 35: `New-Variable version -Value ('${{ inputs.version }}') -Option Constant`
- Line 37: `New-Variable x64 -Value ('${{ inputs.platform }}' -eq 'x64') -Option Constant`

Step 2:
- Line 127: `New-Variable os -Value '${{ runner.os }}' -Option Constant`
- Line 131: `New-Variable cc -Value ('${{ inputs.cc }}' -eq '1') -Option Constant`
- Line 133: `New-Variable clang -Value '${{ steps.install.outputs.clang }}' -Option Constant`
- Line 134: `New-Variable clangxx -Value '${{ steps.install.outputs.clangxx }}' -Option Constant`

All of these should be passed via `env:` variables and referenced as `$env:VAR_NAME` in the PowerShell script instead.

Locations:

- `action.yml:30`
- `action.yml:35`
- `action.yml:37`
- `action.yml:127`
- `action.yml:131`
- `action.yml:133`
- `action.yml:134`

### github-env-injection (severity: high)

Step 1 (id: install) writes `$clang`, `$clangxx`, and `$lld` to `$env:GITHUB_OUTPUT` without sanitization. These variables are derived from `inputs.version` (user-controlled), which is interpolated via `${{ inputs.version }}` into the `$version` variable, then used to construct `$clang`, `$clangxx`, and `$lld`. An attacker-controlled value containing newlines could inject arbitrary key=value pairs into GITHUB_OUTPUT. The required sanitization step (`printf '%s' "$VAR" | tr -d '\n\r'`) is absent before each write.

Offending lines:
- `echo "clang=$clang"     >> $env:GITHUB_OUTPUT`
- `echo "clangxx=$clangxx" >> $env:GITHUB_OUTPUT`
- `echo "lld=$lld"         >> $env:GITHUB_OUTPUT`

Locations:

- `action.yml:121`
- `action.yml:122`
- `action.yml:123`

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

- `action.yml:160`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, static-inline-injection

**Notes:**

Fixed all security findings in action.yml:
1. script-injection / static-inline-injection: Moved all ${{ runner.os }}, ${{ inputs.version }}, ${{ inputs.platform }}, ${{ inputs.cc }}, ${{ steps.install.outputs.clang }}, and ${{ steps.install.outputs.clangxx }} expressions from run: blocks into env: blocks on both steps. PowerShell scripts now reference $env:RUNNER_OS, $env:INPUT_VERSION, $env:INPUT_PLATFORM, $env:INPUT_CC, $env:INSTALL_CLANG, and $env:INSTALL_CLANGXX respectively.
2. github-env-injection: Added sanitization before writing to $env:GITHUB_OUTPUT using PowerShell's -replace '[\r\n]', '' to strip newlines from $clang, $clangxx, and $lld values before echoing them to GITHUB_OUTPUT.

