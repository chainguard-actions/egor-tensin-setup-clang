<!-- markdownlint-disable -->

# Hardening Report: egor-tensin--setup-clang/v2.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **egor-tensin--setup-clang/v2.2** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Step 1 (id: install) directly interpolates ${{ }} expressions inside a PowerShell run: block (sub-rule a). Attacker-controlled inputs are interpolated before the shell parses them: `New-Variable version -Value ('${{ inputs.version }}')` and `New-Variable x64 -Value ('${{ inputs.platform }}' -eq 'x64')`. Also `${{ runner.os }}` is interpolated. A malicious caller can supply crafted values containing PowerShell metacharacters to achieve arbitrary code execution.

Locations:

- `action.yml:27`
- `action.yml:31`
- `action.yml:33`

### script-injection (severity: high)

Step 2 (second run: block) directly interpolates ${{ }} expressions inside a PowerShell run: block (sub-rule a): `New-Variable os -Value '${{ runner.os }}'`, `New-Variable cc -Value ('${{ inputs.cc }}' -eq '1')` (attacker-controlled), `New-Variable clang -Value '${{ steps.install.outputs.clang }}'`, and `New-Variable clangxx -Value '${{ steps.install.outputs.clangxx }}'`. Injecting PowerShell metacharacters via inputs.cc or the step outputs allows arbitrary command execution.

Locations:

- `action.yml:113`
- `action.yml:117`
- `action.yml:119`
- `action.yml:120`

### github-env-injection (severity: high)

Step 1 writes $clang, $clangxx, and $lld to $env:GITHUB_OUTPUT without sanitization. These variables are derived from ${{ inputs.version }} (an attacker-controlled input interpolated earlier in the same script). If inputs.version contains newline characters, the attacker can inject arbitrary key=value pairs into GITHUB_OUTPUT, poisoning downstream steps. The required newline-stripping sanitization is absent. Affected lines: `echo "clang=$clang" >> $env:GITHUB_OUTPUT`, `echo "clangxx=$clangxx" >> $env:GITHUB_OUTPUT`, `echo "lld=$lld" >> $env:GITHUB_OUTPUT`.

Locations:

- `action.yml:107`
- `action.yml:108`
- `action.yml:109`

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

Fixed all script injection and github-env-injection findings in hardened/action/action.yml:

1. Step 1 (id: install): Moved ${{ runner.os }}, ${{ inputs.version }}, and ${{ inputs.platform }} out of the PowerShell run: block into an env: block (as RUNNER_OS_VAL, INPUT_VERSION, INPUT_PLATFORM). The script now reads them via $env:RUNNER_OS_VAL, $env:INPUT_VERSION, $env:INPUT_PLATFORM.

2. Step 1 GITHUB_OUTPUT writes: Added newline sanitization using PowerShell's -replace '[\r\n]', '' on $clang, $clangxx, and $lld before writing to $env:GITHUB_OUTPUT, preventing newline injection attacks.

3. Step 2: Moved ${{ runner.os }}, ${{ inputs.cc }}, ${{ steps.install.outputs.clang }}, and ${{ steps.install.outputs.clangxx }} out of the PowerShell run: block into an env: block (as RUNNER_OS_VAL, INPUT_CC, INSTALL_CLANG, INSTALL_CLANGXX). The script now reads them via environment variables.

All ${{ }} expressions in run: blocks have been eliminated. They now only appear in value: declarations (action outputs, evaluated by GitHub Actions infrastructure) and env: blocks (safe environment variable passing).

