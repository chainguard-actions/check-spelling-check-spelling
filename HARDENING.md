<!-- markdownlint-disable -->

# Hardening Report: check-spelling--check-spelling/v0.0.24

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **check-spelling--check-spelling/v0.0.24** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation in run: blocks. In the 'parse alternate engine' step, `${{ inputs.alternate_engine }}` is interpolated directly inside the shell command: `echo "repo=$(echo '${{ inputs.alternate_engine }}' | perl -pe 's/\@.*//')" >> "$GITHUB_OUTPUT"`. An attacker-controlled input value is expanded by the YAML template engine before the shell ever sees it, enabling command injection. In the 'save sha' step, `${{ github.event.pull_request.number }}` is interpolated directly inside: `PRIVATE_SARIF_REF="refs/pull/${{ github.event.pull_request.number }}/merge"`, allowing a malicious PR number to inject shell commands.

Locations:

- `action.yml:344`
- `action.yml:430`

### github-env-injection (severity: high)

Unsanitized values are written to GitHub special environment files. (1) In the 'parse alternate engine' step, `${{ inputs.alternate_engine }}` is written directly to $GITHUB_OUTPUT without the required `printf '%s' ... | tr -d '\n\r'` sanitization: `echo "repo=$(echo '${{ inputs.alternate_engine }}' | perl -pe 's/\@.*//')" >> "$GITHUB_OUTPUT"`. A newline in the input value can inject arbitrary key=value pairs into GITHUB_OUTPUT. (2) In the 'save sha' step, `${{ github.event.pull_request.number }}` is embedded in PRIVATE_SARIF_REF and written to $GITHUB_ENV without sanitization: `PRIVATE_SARIF_REF="refs/pull/${{ github.event.pull_request.number }}/merge"` followed by `echo "PRIVATE_SARIF_REF=$PRIVATE_SARIF_REF" >> "$GITHUB_ENV"`. A newline in the PR number could inject arbitrary environment variables.

Locations:

- `action.yml:344`
- `action.yml:430`

### unpinned-uses (severity: high)

Multiple `uses:` references in composite action files use mutable version tags instead of pinned 40-character commit SHAs, making the action vulnerable to supply-chain attacks if the referenced tag is moved or the repository is compromised. Failing references: `actions/checkout@v4` (actions/checkout/action.yml), `check-spelling/checkout-merge@v0.0.6` (actions/checkout-merge/action.yml), `actions/upload-artifact@v4` (actions/upload-artifact/action.yml), `github/codeql-action/upload-sarif@v3` (actions/upload-sarif/action.yml).

Locations:

- `actions/checkout/action.yml:63`
- `actions/checkout-merge/action.yml:27`
- `actions/upload-artifact/action.yml:33`
- `actions/upload-sarif/action.yml:37`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.alternate_engine }}" appears directly in run: block of step "parse alternate engine"; move to env: map

Locations:

- `action.yml:383`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.alternate_engine }}" appears directly in run: block of step "parse alternate engine"; move to env: map

Locations:

- `action.yml:384`

### static-unsanitized-env-write (severity: medium)

unsanitized write to $GITHUB_ENV: variable $THIS_GITHUB_JOB_ID in step "shim" comes from a ${{...}} expression and should be sanitized with printf/tr before writing

Locations:

- `action.yml:398`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, static-inline-injection, static-unsanitized-env-write

**Notes:**

Fixed all 6 findings:
1. Pinned 4 unpinned action references to full commit SHAs: actions/checkout@v4→11d5960a, check-spelling/checkout-merge@v0.0.6→29407b1b, actions/upload-artifact@v4→ea165f8d, github/codeql-action/upload-sarif@v3→1190a975.
2. Fixed 'parse alternate engine' step: moved ${{ inputs.alternate_engine }} to env: block as ALTERNATE_ENGINE, sanitized with printf/tr before writing to $GITHUB_OUTPUT.
3. Fixed 'save sha' step: moved ${{ github.event.pull_request.number }} to env: block as PR_NUMBER, sanitized with printf/tr before writing to $GITHUB_ENV.
4. Fixed 'shim' step: sanitized THIS_GITHUB_JOB_ID with printf/tr before writing to $GITHUB_ENV.

