<!-- markdownlint-disable -->

# Hardening Report: check-spelling--check-spelling/v0.0.26

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **check-spelling--check-spelling/v0.0.26** was hardened automatically. 6 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Parse alternate engine' step in action.yml directly interpolates `${{ inputs.alternate_engine }}` inside a `run:` shell command string. The value is embedded unquoted inside a single-quoted echo argument: `echo "repo=$(echo '${{ inputs.alternate_engine }}' | perl ...)"`. An attacker-controlled value containing shell metacharacters or single-quote sequences can break out of the quoting and execute arbitrary commands.

Locations:

- `action.yml:308`

### github-env-injection (severity: high)

The 'Parse alternate engine' step writes the untrusted input `${{ inputs.alternate_engine }}` directly to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). Both lines `echo "repo=$(echo '${{ inputs.alternate_engine }}' | perl ...)" >> "$GITHUB_OUTPUT"` and `echo "branch=$(echo '${{ inputs.alternate_engine }}' | perl ...)" >> "$GITHUB_OUTPUT"` are vulnerable to newline injection, allowing an attacker to inject arbitrary key=value pairs into GITHUB_OUTPUT.

Locations:

- `action.yml:308`

### unpinned-uses (severity: high)

Three sub-action files reference external actions using mutable tags or branch names instead of full 40-character commit SHAs, making them vulnerable to supply-chain attacks: (1) actions/checkout/action.yml uses `actions/checkout@v6` (tag); (2) actions/upload-artifact/action.yml uses `actions/upload-artifact@v7` (tag); (3) actions/publish-coverage/action.yml uses `check-spelling/publish-coverage@main` (branch).

Locations:

- `actions/checkout/action.yml:68`
- `actions/upload-artifact/action.yml:32`
- `actions/publish-coverage/action.yml:24`

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

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, static-inline-injection, static-unsanitized-env-write

**Notes:**

Fixed all 6 findings:
1. script-injection / static-inline-injection (action.yml lines 308, 414, 415): Moved `${{ inputs.alternate_engine }}` from inline `run:` shell string to `env:` block as `ALTERNATE_ENGINE`. The shell script now uses `$ALTERNATE_ENGINE` via safe variable reference.
2. github-env-injection (action.yml line 308): Added `printf '%s' ... | tr -d '\n\r'` sanitization for both `repo` and `branch` values before writing to `$GITHUB_OUTPUT`.
3. static-unsanitized-env-write (action.yml line 430): Added `safe_job_id=$(printf '%s' "$THIS_GITHUB_JOB_ID" | tr -d '\n\r')` before writing to `$GITHUB_ENV`.
4. unpinned-uses: Pinned `actions/checkout@v6` → `@d23441a48e516b6c34aea4fa41551a30e30af803 # v6`, `actions/upload-artifact@v7` → `@043fb46d1a93c77aae656e7c1c64a875d1fc6a0a # v7`, and `check-spelling/publish-coverage@main` → `@e4b5547dea6a84ea9981fbd60c8a8601f62823f3 # main`.

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Save SHA' step in action.yml (around line 310). The PR_NUMBER env var (sourced from `${{ github.event.pull_request.number }}`) is now sanitized with `printf '%s' "$PR_NUMBER" | tr -d '\n\r'` before being used to construct PRIVATE_CHECKOUT_REF. The constructed ref is also sanitized with `tr -d '\n\r'` before being written to $GITHUB_ENV. This prevents newline injection attacks that could allow an attacker to set arbitrary environment variables via a maliciously crafted pull request number.

