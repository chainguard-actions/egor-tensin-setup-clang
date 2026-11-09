<!-- markdownlint-disable -->

# Hardening Report: egor-tensin--setup-clang/v2.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **egor-tensin--setup-clang/v2.3** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): an expression is placed directly in the script text. Step 'install' (pwsh) contains `$linux_host = '${{ runner.os }}' -eq 'Linux'` and `$windows_host = '${{ runner.os }}' -eq 'Windows'`. Use the $env:RUNNER_OS environment variable instead.

Locations:

- `action.yml:34`
- `action.yml:35`

### script-injection (severity: high)

Rule (a): an expression is placed directly in the script text. The second step (pwsh, the cc link step) contains `$linux_host = '${{ runner.os }}' -eq 'Linux'` and `$windows_host = '${{ runner.os }}' -eq 'Windows'`. Use the $env:RUNNER_OS environment variable instead.

Locations:

- `action.yml:177`
- `action.yml:178`

### github-env-injection (severity: high)

Step 'install' builds $clang, $clangxx and $lld from inputs.version (ARG_VERSION -> $version -> $pkg_version, e.g. `$clang = "$clang-$pkg_version"`). It then writes them with `echo "clang=$clang" >> $env:GITHUB_OUTPUT` and the same for clangxx and lld. Newlines and carriage returns are not removed before the write, so a caller could inject extra outputs.

Locations:

- `action.yml:167`
- `action.yml:168`
- `action.yml:169`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

In both pwsh steps, replaced '${{ runner.os }}' with $env:RUNNER_OS. Before the clang, clangxx and lld values are written to $env:GITHUB_OUTPUT, CR/LF characters are now removed with -replace '[\r\n]', ''.

