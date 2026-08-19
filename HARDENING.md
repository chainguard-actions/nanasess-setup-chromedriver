<!-- markdownlint-disable -->

# Hardening Report: nanasess--setup-chromedriver/v3.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **nanasess--setup-chromedriver/v3.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The workflow files reference `nanasess/setup-chromedriver@master` (a mutable branch name, not a pinned SHA) in two jobs each. If the `master` branch of that action is compromised, all workflows using it will execute malicious code. All other `uses:` references in these files are correctly pinned to full 40-character commit SHAs.

Locations:

- `.github/workflows/test.yml:68`
- `.github/workflows/test.yml:113`
- `.github/workflows/windows.yml:65`
- `.github/workflows/windows.yml:107`

### script-injection (severity: high)

Sub-rule (a): `${{ runner.os }}` is interpolated directly inside a `run:` shell command string in two steps of test.yml. Even though `runner.os` is not attacker-controlled, any `${{ ... }}` expression inside a `run:` block is subject to YAML template substitution before the shell sees it, making it a script-injection risk. The offending lines are: `if [[ "${{ runner.os }}" == "Linux" ]]; then` in the `setup` step of both the `test` and `test_default_version` jobs. These should use the environment variable `$RUNNER_OS` instead.

Locations:

- `.github/workflows/test.yml:72`
- `.github/workflows/test.yml:119`

### missing-permissions (severity: medium)

Five workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs. Without explicit permissions, GitHub Actions grants the default token permissions (which may be read/write depending on repository settings), violating the principle of least privilege.

Locations:

- `.github/workflows/container-test.yml:1`
- `.github/workflows/legacy-test.yml:1`
- `.github/workflows/test.yml:1`
- `.github/workflows/version-resolution-test.yml:1`
- `.github/workflows/windows.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all findings across 5 workflow files: (1) Pinned all 4 occurrences of `nanasess/setup-chromedriver@master` to full SHA `904b949d6a9e6f5bb87b61ba516e716ddcc1ff8c` with `# master` comment in test.yml and windows.yml. (2) Replaced both `${{ runner.os }}` expressions inside `run:` blocks in test.yml with the `$RUNNER_OS` environment variable to eliminate script injection risk. (3) Added `permissions: {}` top-level block to all 5 workflow files (test.yml, windows.yml, container-test.yml, legacy-test.yml, version-resolution-test.yml) to enforce least-privilege.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed both findings in .github/workflows/test.yml at the step that handles non-'current' chrome_version matrix values:
1. script-injection: Added double-quotes around $CHROME_VERSION in the echo command: `echo "$CHROME_VERSION"` instead of `echo $CHROME_VERSION`.
2. github-env-injection: Split the curl+echo one-liner into separate steps: (a) capture curl response into CHROMEDRIVER_VERSION_RAW, (b) sanitize with `printf '%s' "$CHROMEDRIVER_VERSION_RAW" | tr -d '\n\r'` into `safe`, (c) write `CHROMEDRIVER_VERSION=$safe` to `"$GITHUB_ENV"` (also quoted the path). This prevents newline injection from the external HTTP response or a crafted matrix value.

