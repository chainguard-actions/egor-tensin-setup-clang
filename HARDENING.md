<!-- markdownlint-disable -->

# Hardening Report: egor-tensin--setup-clang/v2.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **egor-tensin--setup-clang/v2.1** was hardened automatically. 9 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple ${{ }} expressions are interpolated directly inside run: PowerShell script blocks in action.yml. This means YAML template substitution happens before the shell sees the script, allowing an attacker-controlled value to inject arbitrary PowerShell commands. Offending expressions in step 1 (id: install): `${{ runner.os }}` (line 30), `${{ inputs.version }}` (line 34), `${{ inputs.platform }}` (line 35). Offending expressions in step 2: `${{ runner.os }}`, `${{ inputs.cc }}`, `${{ steps.install.outputs.clang }}`, `${{ steps.install.outputs.clangxx }}`. All values should be passed via env: variables and referenced as $ENV_VAR inside the script.

Locations:

- `action.yml:30`
- `action.yml:34`
- `action.yml:35`

### script-injection (severity: high)

Sub-rule (a): Multiple ${{ }} expressions are interpolated directly inside a run: PowerShell script block in .github/actions/build-foo/action.yml. Offending expressions: `${{ inputs.version }}`, `${{ matrix.platform }}`, `${{ runner.os }}`, and `${{ inputs.binary }}` are all substituted into the script before the shell executes it, enabling command injection via attacker-controlled inputs or matrix values.

Locations:

- `.github/actions/build-foo/action.yml:12`

### script-injection (severity: high)

Sub-rule (a): `${{ inputs.version }}` is interpolated directly inside a run: PowerShell script block in .github/actions/check-cc/action.yml. The value is substituted into the script before the shell executes it, and an attacker-controlled version input could inject arbitrary PowerShell commands.

Locations:

- `.github/actions/check-cc/action.yml:22`

### github-env-injection (severity: high)

In action.yml (step id: install), the variables `$clang` and `$clangxx` are derived from `${{ inputs.version }}` (an untrusted input) and written to `$env:GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). Additionally, `$bin_dir` (derived from the inherited `$env:ProgramFiles` environment variable, which is workflow-controlled) is written to `$env:GITHUB_PATH` without sanitization. A newline-containing value could inject additional key=value pairs into GITHUB_OUTPUT or additional paths into GITHUB_PATH.

Locations:

- `action.yml:107`
- `action.yml:110`
- `action.yml:111`

### unpinned-uses (severity: high)

The workflow file .github/workflows/test.yml references external actions using mutable tag refs instead of full 40-character SHA digests, making the workflow vulnerable to supply-chain attacks if the tag is moved. Failing references: `actions/checkout@v6` and `egor-tensin/cleanup-path@v4`. These should be pinned to their full commit SHAs (e.g. `actions/checkout@<40-char-sha> # v6`).

Locations:

- `.github/workflows/test.yml:19`
- `.github/workflows/test.yml:21`

### missing-permissions (severity: medium)

The workflow file .github/workflows/test.yml has no top-level `permissions:` key and no job-level `permissions:` keys on any of its jobs (`test`, `versions`). Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad. A minimal `permissions: {}` or specific scopes (e.g. `contents: read`) should be declared.

Locations:

- `.github/workflows/test.yml:1`

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

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, missing-permissions, static-inline-injection

**Notes:**

Fixed all findings across 4 files:

1. action.yml: Moved all ${{ }} expressions (runner.os, inputs.version, inputs.platform, inputs.cc, steps.install.outputs.clang/clangxx) from run: blocks into env: blocks in both steps. Added PowerShell sanitization (-replace '[\r\n]','') for values written to $env:GITHUB_OUTPUT and $env:GITHUB_PATH to prevent newline injection.

2. .github/actions/build-foo/action.yml: Moved ${{ inputs.version }}, ${{ matrix.platform }}, ${{ runner.os }}, ${{ inputs.binary }} from run: block into env: block; updated script to use $env:INPUT_* variables.

3. .github/actions/check-cc/action.yml: Moved ${{ inputs.version }} from run: block into env: block; updated script to use $env:INPUT_VERSION.

4. .github/workflows/test.yml: Added top-level 'permissions: contents: read' block; pinned actions/checkout@v6 to SHA d23441a48e516b6c34aea4fa41551a30e30af803 and egor-tensin/cleanup-path@v4 to SHA cf0901d753db0bf4d15baf625a6fa537978b03a9.

