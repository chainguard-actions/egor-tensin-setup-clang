<!-- markdownlint-disable -->

# Hardening Report: egor-tensin--setup-clang/v2.3.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **egor-tensin--setup-clang/v2.3.2** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): `${{ runner.os }}` is interpolated directly inside `run:` shell command strings in both composite action steps. Any `${{ ... }}` expression directly inside a `run:` block is a script-injection risk because the value is substituted by the Actions template engine before the shell ever sees it, bypassing shell quoting. Step 1 (the install step): `$linux_host = '${{ runner.os }}' -eq 'Linux'` and `$windows_host = '${{ runner.os }}' -eq 'Windows'`. Step 2 (the link step): same two lines. These should be replaced with the environment variable `$env:RUNNER_OS` which is already available in the runner environment.

Locations:

- `action.yml:34`
- `action.yml:35`
- `action.yml:163`
- `action.yml:164`

### github-env-injection (severity: high)

In the first composite step (install), the variables `$clang`, `$clangxx`, and `$lld` are written to `$GITHUB_OUTPUT` without sanitization. When `inputs.version` is not 'latest', these variables are constructed by appending `$pkg_version` (derived from the untrusted `inputs.version` input via `$env:ARG_VERSION`), e.g. `$clang = "$clang-$pkg_version"`. The values are then written directly: `echo "clang=$clang" >> $env:GITHUB_OUTPUT`, `echo "clangxx=$clangxx" >> $env:GITHUB_OUTPUT`, `echo "lld=$lld" >> $env:GITHUB_OUTPUT`. No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before these writes, allowing a newline-injection attack via a crafted `inputs.version` value to inject arbitrary key-value pairs into GITHUB_OUTPUT.

Locations:

- `action.yml:148`
- `action.yml:149`
- `action.yml:150`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed two security issues in hardened/action/action.yml:
1. script-injection: Replaced all 4 instances of '${{ runner.os }}' in run: blocks with $env:RUNNER_OS (the pre-existing runner environment variable). This was done in both the install step (lines 34-35) and the link step (lines 163-164). The RUNNER_OS env var is already set by the Actions runner and doesn't go through the template engine, eliminating the injection risk.
2. github-env-injection: Added PowerShell sanitization before writing clang, clangxx, and lld values to $GITHUB_OUTPUT. Used PowerShell's -replace '[\r\n]', '' operator to strip newline and carriage-return characters from the values (which can be influenced by the untrusted inputs.version input) before writing them to $GITHUB_OUTPUT, preventing newline injection attacks.

