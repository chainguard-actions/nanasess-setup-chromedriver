<!-- markdownlint-disable -->

# Hardening Report: nanasess--setup-chromedriver/v2.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **nanasess--setup-chromedriver/v2.3.0** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Both workflow files reference GitHub Actions using mutable tags and branch names instead of full 40-character commit SHAs. Specifically: `actions/checkout@v4` (a mutable tag) and `nanasess/setup-chromedriver@master` (a mutable branch) are used in both jobs of both workflow files. If these references are ever moved to point to malicious commits, the workflows will silently execute attacker-controlled code. All `uses:` references must be pinned to a full SHA (e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`).

Locations:

- `.github/workflows/test.yml:41`
- `.github/workflows/test.yml:77`
- `.github/workflows/test.yml:100`
- `.github/workflows/test.yml:116`
- `.github/workflows/windows.yml:28`
- `.github/workflows/windows.yml:55`
- `.github/workflows/windows.yml:80`
- `.github/workflows/windows.yml:95`

### missing-permissions (severity: medium)

Neither `.github/workflows/test.yml` nor `.github/workflows/windows.yml` declares a top-level `permissions:` key, and no individual job within either file declares its own `permissions:` block. Without explicit permissions, workflows run with the default token permissions, which may be overly broad (e.g. `write` access to repository contents). A minimal `permissions:` block (e.g. `contents: read`) should be added at the top level of each workflow.

Locations:

- `.github/workflows/test.yml:1`
- `.github/workflows/windows.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions

**Notes:**

Fixed both workflow files (.github/workflows/test.yml and .github/workflows/windows.yml):
1. unpinned-uses: Pinned all 8 mutable action references to full 40-character commit SHAs:
   - actions/checkout@v4 → actions/checkout@34e114876b0b11c390a56381ad16ebd13914f8d5 # v4 (4 occurrences)
   - nanasess/setup-chromedriver@master → nanasess/setup-chromedriver@904b949d6a9e6f5bb87b61ba516e716ddcc1ff8c # master (4 occurrences)
2. missing-permissions: Added top-level `permissions: contents: read` block to both workflow files, granting only the minimum read access needed for checkout operations.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all three findings across test.yml and windows.yml:

1. **test.yml script-injection** (lines 47-48, 57): Changed `echo $CHROME_VERSION | cut` to `printf '%s' "$CHROME_VERSION" | tr -d '\n\r' | cut` to properly quote and sanitize the workflow-controllable value before use in shell expansion and URL construction.

2. **test.yml github-env-injection** (lines 48, 52, 58): Added `safe=$(printf '%s' "$value" | tr -d '\n\r')` sanitization before each `echo "CHROMEDRIVER_VERSION=$safe" >> "$GITHUB_ENV"` write. Also quoted the curl URL and $GITHUB_ENV path.

3. **windows.yml github-env-injection** (lines 44, 49): Applied `-replace '[\r\n]', ''` to both `$version` (web response content) and `$chrome_fullversion` (file version info) before writing to $Env:GITHUB_ENV via Out-File.

