<!-- markdownlint-disable -->

# Hardening Report: egor-tensin--setup-clang/v2.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **egor-tensin--setup-clang/v2.3** was hardened automatically. 5 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Both `run:` blocks in action.yml directly interpolate GitHub Actions expressions (`${{ ... }}`) inside PowerShell script strings. Sub-rule (a) violation: YAML template substitution occurs before PowerShell processes the string, so an attacker controlling `inputs.version`, `inputs.platform`, or `inputs.cc` can inject arbitrary PowerShell commands.

Step 1 offending lines:
- `New-Variable os -Value '${{ runner.os }}' -Option Constant`
- `New-Variable version -Value ('${{ inputs.version }}') -Option Constant`
- `New-Variable x64 -Value ('${{ inputs.platform }}' -eq 'x64') -Option Constant`

Step 2 offending lines:
- `New-Variable os -Value '${{ runner.os }}' -Option Constant`
- `New-Variable cc -Value ('${{ inputs.cc }}' -eq '1') -Option Constant`
- `New-Variable clang -Value '${{ steps.install.outputs.clang }}' -Option Constant`
- `New-Variable clangxx -Value '${{ steps.install.outputs.clangxx }}' -Option Constant`

Fix: pass all values via environment variables (e.g. `env: VERSION: ${{ inputs.version }}`) and reference them as `$env:VERSION` inside the PowerShell script.

Locations:

- `action.yml:30`
- `action.yml:34`
- `action.yml:36`
- `action.yml:138`
- `action.yml:142`
- `action.yml:144`
- `action.yml:145`

### github-env-injection (severity: high)

In the first `run:` block (step `install`), the variables `$clang`, `$clangxx`, and `$lld` are derived from `${{ inputs.version }}` (interpolated directly into the script as `$version`, then used to construct `$clang`/`$clangxx`/`$lld`) and written to `$env:GITHUB_OUTPUT` without sanitization. An attacker-controlled `inputs.version` containing newlines could inject arbitrary key=value pairs into GITHUB_OUTPUT. The required sanitization step (`printf '%s' ... | tr -d '\n\r'`) is absent before the writes:

```
echo "clang=$clang"     >> $env:GITHUB_OUTPUT
echo "clangxx=$clangxx" >> $env:GITHUB_OUTPUT
echo "lld=$lld"         >> $env:GITHUB_OUTPUT
```

Locations:

- `action.yml:128`
- `action.yml:129`
- `action.yml:130`

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

- `action.yml:179`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, static-inline-injection

**Notes:**

Fixed all script-injection and github-env-injection findings in action.yml:

1. Step 1 (id: install): Added env: block with INPUT_OS=${{ runner.os }}, INPUT_VERSION=${{ inputs.version }}, INPUT_PLATFORM=${{ inputs.platform }}. Updated PowerShell to reference $env:INPUT_OS, $env:INPUT_VERSION, $env:INPUT_PLATFORM instead of direct ${{ }} interpolation.

2. GITHUB_OUTPUT sanitization: Added PowerShell -replace '[\r\n]', '' sanitization for $clang, $clangxx, and $lld before writing to $env:GITHUB_OUTPUT to prevent newline injection.

3. Step 2: Added env: block with INPUT_OS=${{ runner.os }}, INPUT_CC=${{ inputs.cc }}, INPUT_CLANG=${{ steps.install.outputs.clang }}, INPUT_CLANGXX=${{ steps.install.outputs.clangxx }}. Updated PowerShell to reference $env:INPUT_OS, $env:INPUT_CC, $env:INPUT_CLANG, $env:INPUT_CLANGXX.

All ${{ }} expressions now only appear in outputs.value: fields and env: blocks, which are safe from shell injection.

### Iteration 2

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed script injection in three composite action files (build-foo, check-cc, check-lld) by moving all ${{ }} expressions into step-level env: blocks and referencing them as $env:VAR_NAME in PowerShell. Pinned actions/checkout@v6 to SHA d23441a48e516b6c34aea4fa41551a30e30af803 and egor-tensin/cleanup-path@v5 to SHA 3974efd722fa10f283df76342125776a352072dd in test.yml. Added top-level `permissions: {}` to test.yml to enforce least-privilege.

