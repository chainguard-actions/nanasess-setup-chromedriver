<!-- markdownlint-disable -->

# Hardening Report: nanasess--setup-chromedriver/v2.2.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **nanasess--setup-chromedriver/v2.2.2** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Both workflow files reference actions by mutable tags or branch names instead of full 40-character commit SHAs. Specifically: `actions/checkout@v4` (tag) appears in both test.yml and windows.yml, and `nanasess/setup-chromedriver@master` (branch) appears in both files. These can be silently redirected to malicious code if the upstream repository is compromised.

Locations:

- `.github/workflows/test.yml:30`
- `.github/workflows/test.yml:55`
- `.github/workflows/test.yml:68`
- `.github/workflows/test.yml:79`
- `.github/workflows/windows.yml:26`
- `.github/workflows/windows.yml:46`
- `.github/workflows/windows.yml:58`
- `.github/workflows/windows.yml:67`

### missing-permissions (severity: medium)

Neither `.github/workflows/test.yml` nor `.github/workflows/windows.yml` declares a top-level `permissions:` key, and no individual job within either file declares its own `permissions:` block. Without explicit permissions, the default GITHUB_TOKEN permissions (which may include write access) apply to all jobs, violating the principle of least privilege.

Locations:

- `.github/workflows/test.yml:1`
- `.github/workflows/windows.yml:1`

### script-injection (severity: high)

Sub-rule (b) violation: In test.yml, `CHROME_VERSION` is populated from the workflow-controllable `matrix.chrome_version` via `env: CHROME_VERSION: ${{ matrix.chrome_version }}`, then expanded unquoted inside the `run:` script: `CHROME_VERSION=$(echo $CHROME_VERSION | cut -d '.' -f 1)`. The unquoted `$CHROME_VERSION` allows shell metacharacter injection (e.g. semicolons, pipes, backticks) from the matrix value. The env var must be double-quoted: `$(echo "$CHROME_VERSION" | cut -d '.' -f 1)`.

Locations:

- `.github/workflows/test.yml:42`

### github-env-injection (severity: high)

In test.yml, values derived from workflow-controllable sources are written to `$GITHUB_ENV` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). (1) At line 43, `CHROME_VERSION` (sourced from `matrix.chrome_version` via env) is embedded unsanitized in a curl URL and the result written to GITHUB_ENV: `echo "CHROMEDRIVER_VERSION=$(curl ... LATEST_RELEASE_${CHROME_VERSION})" >> $GITHUB_ENV`. (2) At line 46, `CHROME_VERSION` derived from the chrome binary's version string (an external, uncontrolled value) is written unsanitized: `echo "CHROMEDRIVER_VERSION=$CHROME_VERSION" >> $GITHUB_ENV`. A newline injected into any of these values could define additional environment variables for subsequent steps.

Locations:

- `.github/workflows/test.yml:43`
- `.github/workflows/test.yml:46`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection, github-env-injection

**Notes:**

Fixed all four findings in .github/workflows/test.yml and .github/workflows/windows.yml:

1. unpinned-uses: Pinned actions/checkout@v4 → @11d5960a326750d5838078e36cf38b85af677262 and nanasess/setup-chromedriver@master → @904b949d6a9e6f5bb87b61ba516e716ddcc1ff8c in both files (4 occurrences each).

2. missing-permissions: Added `permissions: {}` top-level block to both test.yml and windows.yml.

3. script-injection: In test.yml, double-quoted all unquoted `$CHROME_VERSION` expansions in `echo $CHROME_VERSION | cut ...` commands (3 occurrences).

4. github-env-injection: In test.yml, sanitized all values written to $GITHUB_ENV using `safe=$(printf '%s' "..." | tr -d '\n\r')` then `echo "CHROMEDRIVER_VERSION=$safe" >> $GITHUB_ENV` — covers the curl-fetched version and the chrome binary version string.

