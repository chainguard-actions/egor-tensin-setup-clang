<!-- markdownlint-disable -->

# Hardening Report: egor-tensin--setup-clang/v2.3.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **egor-tensin--setup-clang/v2.3.3** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): `${{ runner.os }}` is directly interpolated inside the `run:` PowerShell shell script in step 1 (the 'install' step). Any `${{ ... }}` expression directly inside a `run:` block is a script-injection finding, as the value flows through YAML template substitution before the shell processes it. Offending lines: `$linux_host = '${{ runner.os }}' -eq 'Linux'` and `$windows_host = '${{ runner.os }}' -eq 'Windows'`. Fix: use the environment variable `$env:RUNNER_OS` instead of the expression.

Locations:

- `action.yml:35`
- `action.yml:36`

### script-injection (severity: high)

Sub-rule (a): `${{ runner.os }}` is directly interpolated inside the `run:` PowerShell shell script in step 2 (the symlink/cc setup step). Same pattern as step 1 — the expression is substituted into the shell command string before execution. Offending lines: `$linux_host = '${{ runner.os }}' -eq 'Linux'` and `$windows_host = '${{ runner.os }}' -eq 'Windows'`. Fix: use `$env:RUNNER_OS` instead.

Locations:

- `action.yml:155`
- `action.yml:156`

### github-env-injection (severity: high)

Step 1 ('install') writes `$clang`, `$clangxx`, and `$lld` to `$GITHUB_OUTPUT` without sanitization. These values are derived from `$version`, which is read from the `ARG_VERSION` environment variable set to `${{ inputs.version }}` — an untrusted caller-controlled input. An attacker can inject newlines into `inputs.version` to poison `$GITHUB_OUTPUT`. The required sanitization step (`printf '%s' "$value" | tr -d '\n\r'`) is absent before each write. Offending lines: `echo "clang=$clang" >> $env:GITHUB_OUTPUT`, `echo "clangxx=$clangxx" >> $env:GITHUB_OUTPUT`, `echo "lld=$lld" >> $env:GITHUB_OUTPUT`.

Locations:

- `action.yml:143`
- `action.yml:144`
- `action.yml:145`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed three findings in hardened/action/action.yml:
1. Script injection (step 1): Replaced `'${{ runner.os }}'` with `$env:RUNNER_OS` in the PowerShell run block. PowerShell accesses environment variables via `$env:RUNNER_OS`, which is the correct idiomatic form and avoids YAML template substitution.
2. Script injection (step 2): Same fix applied to the second step's PowerShell run block.
3. GitHub env injection: Added sanitization before writing clang, clangxx, and lld values to $GITHUB_OUTPUT. Used PowerShell's `-replace '[\r\n]', ''` to strip carriage return and newline characters from values derived from the caller-controlled `inputs.version` input, preventing newline injection attacks on $GITHUB_OUTPUT.

