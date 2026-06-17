<!-- markdownlint-disable -->

# Hardening Report: egor-tensin--setup-clang/v2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **egor-tensin--setup-clang/v2.0** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Both `run:` blocks in action.yml directly interpolate `${{ ... }}` expressions inside PowerShell script strings (rule a). This includes attacker-controllable inputs: `${{ inputs.version }}` (line 33), `${{ inputs.platform }}` (line 35), `${{ inputs.cc }}` (line 128), as well as `${{ runner.os }}` (lines 29, 124) and `${{ steps.install.outputs.clang }}`/`${{ steps.install.outputs.clangxx }}` (lines 130–131). GitHub Actions template substitution occurs before the shell parses the script, so a malicious value (e.g. `inputs.version = "'; Invoke-Expression ...; '"`) can inject arbitrary PowerShell commands.

Locations:

- `action.yml:29`
- `action.yml:33`
- `action.yml:35`
- `action.yml:124`
- `action.yml:128`
- `action.yml:130`
- `action.yml:131`

### github-env-injection (severity: high)

In step 1 of action.yml, the variables `$clang` and `$clangxx` are derived from the user-controlled input `${{ inputs.version }}` (interpolated at line 33 and used to build package/binary names). They are written unsanitized to `$env:GITHUB_OUTPUT` via `echo "clang=$clang" >> $env:GITHUB_OUTPUT` and `echo "clangxx=$clangxx" >> $env:GITHUB_OUTPUT`. No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before the write. An attacker supplying a newline-containing `inputs.version` value could inject arbitrary key=value pairs into GITHUB_OUTPUT, poisoning downstream steps.

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

Fixed all script-injection and github-env-injection findings in action.yml:

1. Step 1 (id: install): Moved ${{ runner.os }}, ${{ inputs.version }}, and ${{ inputs.platform }} from the run: block into an env: block as INPUT_OS, INPUT_VERSION, and INPUT_PLATFORM. The PowerShell script now reads $env:INPUT_OS, $env:INPUT_VERSION, and $env:INPUT_PLATFORM.

2. Step 1 GITHUB_OUTPUT writes: Added newline sanitization using PowerShell's -replace '[\r\n]', '' on $clang and $clangxx before writing to $env:GITHUB_OUTPUT, preventing newline injection.

3. Step 2: Moved ${{ runner.os }}, ${{ inputs.cc }}, ${{ steps.install.outputs.clang }}, and ${{ steps.install.outputs.clangxx }} from the run: block into an env: block as INPUT_OS, INPUT_CC, INPUT_CLANG, and INPUT_CLANGXX. The PowerShell script now reads these from environment variables.

The outputs: block value fields retain their ${{ steps.install.outputs.* }} expressions as required — these are not shell scripts and are safe.

