<!-- markdownlint-disable -->

# Hardening Report: check-spelling--check-spelling/v0.0.24

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **check-spelling--check-spelling/v0.0.24** was hardened automatically. 8 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The `parse-alternate-engine` step directly interpolates `${{ inputs.alternate_engine }}` inside a `run:` shell command string. An attacker-controlled value is passed directly into the shell, enabling command injection. Offending lines: echo "repo=$(echo '${{ inputs.alternate_engine }}' | perl ...)" >> "$GITHUB_OUTPUT" and echo "branch=$(echo '${{ inputs.alternate_engine }}' | perl ...)" >> "$GITHUB_OUTPUT"

Locations:

- `action.yml:296`
- `action.yml:297`

### script-injection (severity: high)

Sub-rule (a): The `save-sha` step directly interpolates `${{ github.event.pull_request.number }}` inside a `run:` shell command string. This GitHub context value flows through YAML template substitution before the shell sees it, enabling script injection. Offending line: PRIVATE_SARIF_REF="refs/pull/${{ github.event.pull_request.number }}/merge"

Locations:

- `action.yml:356`

### github-env-injection (severity: high)

The `parse-alternate-engine` step writes values derived from the untrusted input `${{ inputs.alternate_engine }}` directly to $GITHUB_OUTPUT without the required sanitization step (printf '%s' ... | tr -d '\n\r'). A newline in the input value could inject arbitrary output variables.

Locations:

- `action.yml:296`
- `action.yml:297`

### github-env-injection (severity: high)

The `save-sha` step constructs PRIVATE_SARIF_REF from `${{ github.event.pull_request.number }}` (an untrusted GitHub context value) and writes it to $GITHUB_ENV without sanitization. A newline injected via the PR number could set arbitrary environment variables. Offending lines: PRIVATE_SARIF_REF="refs/pull/${{ github.event.pull_request.number }}/merge" followed by echo "PRIVATE_SARIF_REF=$PRIVATE_SARIF_REF" >> "$GITHUB_ENV".

Locations:

- `action.yml:356`
- `action.yml:357`

### unpinned-uses (severity: high)

Multiple uses: references in sub-action files use mutable tag or version refs instead of pinned 40-character SHA digests, making the action vulnerable to supply-chain attacks. Failing references: actions/checkout@v4 (actions/checkout/action.yml), actions/upload-artifact@v4 (actions/upload-artifact/action.yml), check-spelling/checkout-merge@v0.0.6 (actions/checkout-merge/action.yml), github/codeql-action/upload-sarif@v3 (actions/upload-sarif/action.yml).

Locations:

- `actions/checkout/action.yml:47`
- `actions/upload-artifact/action.yml:27`
- `actions/checkout-merge/action.yml:22`
- `actions/upload-sarif/action.yml:42`

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

**Fixes applied:** script-injection, github-env-injection, static-inline-injection, static-unsanitized-env-write, unpinned-uses

**Notes:**

Fixed all 8 findings across action.yml and 4 sub-action files:

1. parse-alternate-engine step: Moved `${{ inputs.alternate_engine }}` to env: block as ALTERNATE_ENGINE; added printf/tr sanitization before writing repo and branch to $GITHUB_OUTPUT.

2. save-sha step: Moved `${{ github.event.pull_request.number }}` to env: block as PR_NUMBER; added printf/tr sanitization before writing PRIVATE_SARIF_REF to $GITHUB_ENV.

3. shim step: Added printf/tr sanitization for THIS_GITHUB_JOB_ID before writing to $GITHUB_ENV.

4. Pinned 4 unpinned action references to full commit SHAs:
   - actions/checkout@v4 → @34e114876b0b11c390a56381ad16ebd13914f8d5 # v4
   - actions/upload-artifact@v4 → @ea165f8d65b6e75b540449e92b4886f43607fa02 # v4
   - check-spelling/checkout-merge@v0.0.6 → @29407b1b8660c562313ed65c81feab3e3255c03c # v0.0.6
   - github/codeql-action/upload-sarif@v3 → @dd903d2e4f5405488e5ef1422510ee31c8b32357 # v3

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in the `retrieve-comment` step of action.yml (line 462). The `suffix` env var was set from `inputs.report_title_suffix` (attacker-controlled) and used directly inside a double-quoted shell string in `gh run download -R "$GITHUB_REPOSITORY" "$GITHUB_RUN_ID" -n "check-spelling-comment-$suffix"`. Added a sanitization line `safe_suffix="${suffix//[^a-zA-Z0-9._-]/}"` at the start of the run block to strip any characters that could be used for command injection (keeping only alphanumerics, dots, hyphens, and underscores). The `gh run download` command now uses `$safe_suffix` instead of `$suffix`.

