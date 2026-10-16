<!-- markdownlint-disable -->

# Hardening Report: egor-tensin--setup-clang/v2.3.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **egor-tensin--setup-clang/v2.3.1** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): GitHub Actions expressions are directly interpolated inside `run:` shell (PowerShell) command strings in two steps of action.yml. In the first step (id: install), `${{ runner.os }}` (line 29), `${{ inputs.version }}` (line 32), and `${{ inputs.platform }}` (line 34) are substituted by the YAML template engine before PowerShell parses the script. An attacker-controlled input (e.g. inputs.version = `'; Invoke-Expression malicious; '`) can inject arbitrary PowerShell commands. In the second step, `${{ runner.os }}` (line 153), `${{ inputs.cc }}` (line 157), `${{ steps.install.outputs.clang }}` (line 159), and `${{ steps.install.outputs.clangxx }}` (line 160) are similarly interpolated directly into the run script.

Locations:

- `action.yml:29`
- `action.yml:32`
- `action.yml:34`
- `action.yml:153`
- `action.yml:157`
- `action.yml:159`
- `action.yml:160`

### github-env-injection (severity: high)

In the first `run:` block (id: install), the variables `$clang`, `$clangxx`, and `$lld` are derived from `${{ inputs.version }}` (an attacker-controlled input) via string concatenation (e.g. `$clang = "$clang-$pkg_version"` where `$pkg_version` comes from `Format-UpstreamVersion $version` and `$version` is set from `inputs.version`). These variables are then written to `$env:GITHUB_OUTPUT` without any newline-stripping sanitization (`printf '%s' ... | tr -d '\n\r'`). A newline character embedded in `inputs.version` could inject additional key=value pairs into GITHUB_OUTPUT, potentially overwriting outputs consumed by downstream steps.

Locations:

- `action.yml:147`
- `action.yml:148`
- `action.yml:149`

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

Fixed all security findings in hardened/action/action.yml:

1. script-injection & static-inline-injection: Moved all ${{ }} expressions out of run: blocks into env: blocks for both steps. In step 1 (id: install): INPUT_OS (${{ runner.os }}), INPUT_VERSION (${{ inputs.version }}), INPUT_PLATFORM (${{ inputs.platform }}). In step 2: INPUT_OS (${{ runner.os }}), INPUT_CC (${{ inputs.cc }}), INPUT_CLANG (${{ steps.install.outputs.clang }}), INPUT_CLANGXX (${{ steps.install.outputs.clangxx }}). All are referenced in PowerShell as $env:VAR_NAME (correct pwsh syntax).

2. github-env-injection: Added newline sanitization before writing $clang, $clangxx, and $lld to $env:GITHUB_OUTPUT using PowerShell's -replace '[\r\n]', '' operator to strip carriage returns and newlines that could inject additional key=value pairs.

The outputs value: fields retain their ${{ }} expressions as those are YAML template fields, not shell code.

