<!-- markdownlint-disable -->

# Hardening Report: egor-tensin--setup-clang/v2.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **egor-tensin--setup-clang/v2.3** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Both run: blocks in action.yml directly interpolate ${{ }} expressions into PowerShell shell commands (sub-rule a). In step 1 (id: install): '${{ runner.os }}', '${{ inputs.version }}', and '${{ inputs.platform }}' are interpolated directly into the script string before the shell ever sees it. An attacker controlling inputs.version or inputs.platform can inject arbitrary PowerShell. In step 2: '${{ runner.os }}', '${{ inputs.cc }}', '${{ steps.install.outputs.clang }}', and '${{ steps.install.outputs.clangxx }}' are similarly interpolated. All ${{ ... }} expressions inside run: blocks are script-injection risks regardless of context.

Locations:

- `action.yml:30`
- `action.yml:33`
- `action.yml:35`
- `action.yml:116`
- `action.yml:119`
- `action.yml:121`

### github-env-injection (severity: high)

In the first run: block (step id: install), the variables $clang, $clangxx, and $lld are written to $GITHUB_OUTPUT without sanitization. These variables are derived from '${{ inputs.version }}' (user-controlled) — e.g. $clang is set to "clang-$pkg_version" where $pkg_version comes from inputs.version. An attacker can inject newlines into inputs.version to poison GITHUB_OUTPUT with arbitrary key=value pairs. The required sanitization step (printf '%s' ... | tr -d '\n\r') is absent before each write. Similarly, $bin_dir (derived from $env:ProgramFiles, an inherited env var) is written to $GITHUB_PATH without sanitization.

Locations:

- `action.yml:112`
- `action.yml:114`
- `action.yml:115`
- `action.yml:116`

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

1. Step 1 (id: install): Added env: block with INPUT_OS=${{ runner.os }}, INPUT_VERSION=${{ inputs.version }}, INPUT_PLATFORM=${{ inputs.platform }}. Updated PowerShell script to read $env:INPUT_OS, $env:INPUT_VERSION, $env:INPUT_PLATFORM instead of inline ${{ }} expressions.

2. Step 2: Added env: block with INPUT_OS=${{ runner.os }}, INPUT_CC=${{ inputs.cc }}, INPUT_CLANG=${{ steps.install.outputs.clang }}, INPUT_CLANGXX=${{ steps.install.outputs.clangxx }}. Updated PowerShell script to read $env:INPUT_* variables.

3. github-env-injection: Added PowerShell sanitization using `-replace '[\r\n]', ''` before writing $clang, $clangxx, $lld to $GITHUB_OUTPUT and $bin_dir to $GITHUB_PATH, preventing newline injection attacks.

All ${{ }} expressions in run: blocks have been eliminated. They now only appear in env: blocks and outputs: value declarations, which are safe.

