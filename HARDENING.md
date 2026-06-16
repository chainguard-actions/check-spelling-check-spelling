<!-- markdownlint-disable -->

# Hardening Report: check-spelling--check-spelling/v0.0.26

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **check-spelling--check-spelling/v0.0.26** was hardened automatically. 6 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Parse alternate engine' step in action.yml directly interpolates `${{ inputs.alternate_engine }}` inside a `run:` shell command string. This user-controlled input is embedded into the shell before quoting occurs, enabling command injection. Offending lines:
  echo "repo=$(echo '${{ inputs.alternate_engine }}' | perl -pe 's/\@.*//')" >> "$GITHUB_OUTPUT"
  echo "branch=$(echo '${{ inputs.alternate_engine }}' | perl -ne 'next unless s/.*\@//; print')" >> "$GITHUB_OUTPUT"

Locations:

- `action.yml:276`
- `action.yml:277`

### github-env-injection (severity: high)

The 'Parse alternate engine' step writes values derived from `${{ inputs.alternate_engine }}` (a user-controlled input) directly to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). An attacker can inject newlines into the input to poison subsequent output variable reads.

Locations:

- `action.yml:276`
- `action.yml:277`

### unpinned-uses (severity: high)

Multiple action files reference external actions using mutable tags or branch names instead of full 40-character commit SHAs, making them vulnerable to supply-chain attacks:
- actions/checkout/action.yml: `uses: actions/checkout@v6` (tag)
- actions/publish-coverage/action.yml: `uses: check-spelling/publish-coverage@main` (branch)
- actions/upload-artifact/action.yml: `uses: actions/upload-artifact@v7` (tag)

Locations:

- `actions/checkout/action.yml:68`
- `actions/publish-coverage/action.yml:22`
- `actions/upload-artifact/action.yml:33`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.alternate_engine }}" appears directly in run: block of step "Parse alternate engine"; move to env: map

Locations:

- `action.yml:414`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.alternate_engine }}" appears directly in run: block of step "Parse alternate engine"; move to env: map

Locations:

- `action.yml:415`

### static-unsanitized-env-write (severity: medium)

unsanitized write to $GITHUB_ENV: variable $THIS_GITHUB_JOB_ID in step "Shim path and local actions" comes from a ${{...}} expression and should be sanitized with printf/tr before writing

Locations:

- `action.yml:430`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, static-inline-injection, static-unsanitized-env-write, unpinned-uses

**Notes:**

Fixed all 6 findings across action.yml and the three sub-action files:

1. script-injection + github-env-injection + static-inline-injection (action.yml lines 276-277/414-415): In the 'Parse alternate engine' step, moved `${{ inputs.alternate_engine }}` from the run: shell string into an env: block as ALTERNATE_ENGINE. Added `printf '%s' "$ALTERNATE_ENGINE" | tr -d '\n\r'` sanitization pipeline before writing repo and branch values to $GITHUB_OUTPUT.

2. static-unsanitized-env-write (action.yml line 430): In the 'Shim path and local actions' step, added `safe_job_id=$(printf '%s' "$THIS_GITHUB_JOB_ID" | tr -d '\n\r')` before writing to $GITHUB_ENV to prevent newline injection.

3. unpinned-uses: Pinned all three mutable references to full 40-character commit SHAs:
   - actions/checkout@v6 → @df4cb1c069e1874edd31b4311f1884172cec0e10 # v6
   - check-spelling/publish-coverage@main → @e4b5547dea6a84ea9981fbd60c8a8601f62823f3 # main
   - actions/upload-artifact@v7 → @043fb46d1a93c77aae656e7c1c64a875d1fc6a0a # v7

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Save SHA' step in action.yml to sanitize the PR_NUMBER value (sourced from github.event.pull_request.number) before using it to construct PRIVATE_CHECKOUT_REF. Added `safe_pr_number=$(printf '%s' "$PR_NUMBER" | tr -d '\n\r')` to strip newlines from PR_NUMBER before building the ref string, and added `safe_ref=$(printf '%s' "$PRIVATE_CHECKOUT_REF" | tr -d '\n\r')` to sanitize the final value before writing it to $GITHUB_ENV. This prevents any potential newline injection via the github context value written to the special environment file.

