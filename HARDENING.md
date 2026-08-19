<!-- markdownlint-disable -->

# Hardening Report: egor-tensin--setup-clang/v2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **egor-tensin--setup-clang/v2.0** was hardened automatically. 6 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Step 1 (id: install) directly interpolates ${{ }} expressions inside a PowerShell run: block. GitHub Actions substitutes these values into the script text before PowerShell parses it, so an attacker-controlled value can break out of the string literal and inject arbitrary PowerShell commands. Offending lines:
- `New-Variable os -Value '${{ runner.os }}' -Option Constant` (rule a: runner.* expression in run:)
- `New-Variable version -Value ('${{ inputs.version }}') -Option Constant` (rule a: attacker-controlled inputs.*)
- `New-Variable x64 -Value ('${{ inputs.platform }}' -eq 'x64') -Option Constant` (rule a: attacker-controlled inputs.*)

Locations:

- `action.yml:29`
- `action.yml:33`
- `action.yml:35`

### script-injection (severity: high)

Step 2 directly interpolates ${{ }} expressions inside a PowerShell run: block. GitHub Actions substitutes these values into the script text before PowerShell parses it, enabling command injection. Offending lines:
- `New-Variable os -Value '${{ runner.os }}' -Option Constant` (rule a: runner.* expression in run:)
- `New-Variable cc -Value ('${{ inputs.cc }}' -eq '1') -Option Constant` (rule a: attacker-controlled inputs.*)
- `New-Variable clang -Value '${{ steps.install.outputs.clang }}' -Option Constant` (rule a: steps.*.outputs.* expression in run:)
- `New-Variable clangxx -Value '${{ steps.install.outputs.clangxx }}' -Option Constant` (rule a: steps.*.outputs.* expression in run:)

Locations:

- `action.yml:124`
- `action.yml:128`
- `action.yml:130`
- `action.yml:131`

### github-env-injection (severity: high)

Step 1 (id: install) writes $clang and $clangxx to $GITHUB_OUTPUT without sanitization. These variables are derived from ${{ inputs.version }} and ${{ inputs.platform }} (attacker-controlled inputs) via Format-UpstreamVersion and string concatenation. No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before the write, allowing newline injection to poison GITHUB_OUTPUT.
- `echo "clang=$clang" >> $env:GITHUB_OUTPUT`
- `echo "clangxx=$clangxx" >> $env:GITHUB_OUTPUT`

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

- `action.yml:151`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, static-inline-injection

**Notes:**

Fixed all script injection findings in action.yml by moving all ${{ }} expressions from run: blocks into env: blocks for both steps. Step 1: moved runner.os, inputs.version, inputs.platform to env: INPUT_OS/INPUT_VERSION/INPUT_PLATFORM; Step 2: moved runner.os, inputs.cc, steps.install.outputs.clang, steps.install.outputs.clangxx to env: INPUT_OS/INPUT_CC/INPUT_CLANG/INPUT_CLANGXX. Fixed github-env-injection by sanitizing $clang and $clangxx with PowerShell's -replace '[\r\n]', '' before writing to $GITHUB_OUTPUT. All ${{ }} expressions in run: blocks have been eliminated.

### Iteration 2

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed 4 findings: (1) build-foo/action.yml: moved ${{ inputs.version }}, ${{ matrix.platform }}, ${{ runner.os }}, and ${{ inputs.binary }} out of the run: shell string into an env: block, referencing them as $env:INPUT_VERSION, $env:MATRIX_PLATFORM, $env:RUNNER_OS_VAL, $env:INPUT_BINARY in PowerShell. (2) check-cc/action.yml: moved ${{ inputs.version }} into an env: block as INPUT_VERSION, referenced as $env:INPUT_VERSION in PowerShell. (3) test.yml: pinned actions/checkout@v6 → @d23441a48e516b6c34aea4fa41551a30e30af803 # v6 (both occurrences) and egor-tensin/cleanup-path@v4 → @cf0901d753db0bf4d15baf625a6fa537978b03a9 # v4. (4) test.yml: added top-level `permissions: contents: read` block.

