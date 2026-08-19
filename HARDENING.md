<!-- markdownlint-disable -->

# Hardening Report: nanasess--setup-chromedriver/v2.4.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **nanasess--setup-chromedriver/v2.4.0** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags or branch names instead of pinned 40-character SHA digests, making them vulnerable to supply-chain attacks.

.github/workflows/test.yml:
  - uses: actions/checkout@v6 (tag)
  - uses: nanasess/setup-chromedriver@master (branch)

.github/workflows/windows.yml:
  - uses: actions/checkout@v6 (tag)
  - uses: nanasess/setup-chromedriver@master (branch)

.github/workflows/claude-code-review.yml:
  - uses: actions/checkout@v6 (tag)
  - uses: anthropics/claude-code-action@beta (branch)

.github/workflows/claude.yml:
  - uses: actions/checkout@v6 (tag)
  - uses: anthropics/claude-code-action@beta (branch)

Locations:

- `.github/workflows/test.yml:33`
- `.github/workflows/test.yml:60`
- `.github/workflows/windows.yml:27`
- `.github/workflows/windows.yml:55`
- `.github/workflows/claude-code-review.yml:27`
- `.github/workflows/claude-code-review.yml:33`
- `.github/workflows/claude.yml:27`
- `.github/workflows/claude.yml:33`

### missing-permissions (severity: medium)

Workflow files test.yml and windows.yml have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs. Without explicit permissions, the GITHUB_TOKEN is granted default (potentially write) permissions, violating the principle of least privilege.

Locations:

- `.github/workflows/test.yml:1`
- `.github/workflows/windows.yml:1`

### script-injection (severity: high)

Sub-rule (a): The expression `${{ runner.os }}` is interpolated directly inside `run:` shell command strings in two steps of test.yml. Any `${{ ... }}` expression inside a run: block is a script-injection risk because the value is substituted into the shell command string before the shell parses it, allowing metacharacter injection if the value were ever attacker-influenced. The offending lines are:
  - `if [[ "${{ runner.os }}" == "Linux" ]]; then` (in the 'test' job 'setup' step)
  - `if [[ "${{ runner.os }}" == "Linux" ]]; then` (in the 'test_default_version' job 'setup' step)

Locations:

- `.github/workflows/test.yml:57`
- `.github/workflows/test.yml:88`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all findings across 4 workflow files:

1. unpinned-uses: Pinned all mutable action references to full 40-char SHAs with tag comments preserved:
   - actions/checkout@v6 → @d23441a48e516b6c34aea4fa41551a30e30af803 (all 4 files)
   - nanasess/setup-chromedriver@master → @904b949d6a9e6f5bb87b61ba516e716ddcc1ff8c (test.yml, windows.yml)
   - anthropics/claude-code-action@beta → @28f83620103c48a57093dcc2837eec89e036bb9f (claude-code-review.yml, claude.yml)

2. missing-permissions: Added top-level `permissions: contents: read` to test.yml and windows.yml.

3. script-injection: In test.yml, both `${{ runner.os }}` expressions inside run: blocks were moved to the step's env: block as RUNNER_OS, and the shell scripts now reference $RUNNER_OS instead of the inline expression.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all four findings across test.yml and windows.yml:

1. test.yml (script-injection + github-env-injection): In the 'matrix.chrome_version != current' step, replaced unquoted `echo $CHROME_VERSION` with `printf '%s' "$CHROME_VERSION"` (quoted), stored curl response in RAW_VERSION, then sanitized with `printf '%s' "$RAW_VERSION" | tr -d '\n\r'` before writing to GITHUB_ENV.

2. test.yml (github-env-injection): In the 'current ubuntu' step, captured Chrome binary version in RAW_VERSION, then sanitized with `printf '%s' "$RAW_VERSION" | tr -d '\n\r'` before writing to GITHUB_ENV.

3. windows.yml (github-env-injection): In the PowerShell step, sanitized both the HTTP response content (`$response.Content -replace '[\r\n]', ''`) and the file version string (`$chrome_fullversion -replace '[\r\n]', ''`) before writing to GITHUB_ENV. Also changed from `echo "KEY=VALUE" | Out-File` to `"KEY=VALUE" | Out-File` to avoid echo adding extra content.

