<!-- markdownlint-disable -->

# Hardening Report: check-spelling--check-spelling/v0.0.26

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **check-spelling--check-spelling/v0.0.26** was hardened automatically. 6 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Parse alternate engine' step in action.yml directly interpolates `${{ inputs.alternate_engine }}` inside a `run:` shell command string. An attacker-controlled value for `inputs.alternate_engine` is embedded verbatim into the shell command before the shell ever sees it, enabling arbitrary command injection. The offending lines are:
  echo "repo=$(echo '${{ inputs.alternate_engine }}' | perl -pe 's/\@.*//')" >> "$GITHUB_OUTPUT"
  echo "branch=$(echo '${{ inputs.alternate_engine }}' | perl -ne 'next unless s/.*\@//; print')" >> "$GITHUB_OUTPUT"

Locations:

- `action.yml:370`

### github-env-injection (severity: high)

The 'Parse alternate engine' step writes values derived from `${{ inputs.alternate_engine }}` (an attacker-controlled input) directly to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A newline embedded in `inputs.alternate_engine` can inject arbitrary key=value pairs into GITHUB_OUTPUT, poisoning subsequent steps. The offending lines are:
  echo "repo=$(echo '${{ inputs.alternate_engine }}' | perl -pe 's/\@.*//')" >> "$GITHUB_OUTPUT"
  echo "branch=$(echo '${{ inputs.alternate_engine }}' | perl -ne 'next unless s/.*\@//; print')" >> "$GITHUB_OUTPUT"

Locations:

- `action.yml:370`

### unpinned-uses (severity: high)

Multiple sub-actions reference external actions by mutable tag or branch instead of a full 40-character commit SHA, making them vulnerable to supply-chain attacks if the referenced tag/branch is moved or compromised:
- `actions/checkout/action.yml`: uses: actions/checkout@v6 (tag)
- `actions/upload-artifact/action.yml`: uses: actions/upload-artifact@v7 (tag)
- `actions/publish-coverage/action.yml`: uses: check-spelling/publish-coverage@main (branch)

Locations:

- `actions/checkout/action.yml:63`
- `actions/upload-artifact/action.yml:29`
- `actions/publish-coverage/action.yml:21`

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

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, static-inline-injection, static-unsanitized-env-write

**Notes:**

Fixed all 6 findings:
1. Pinned actions/checkout@v6 to SHA d23441a48e516b6c34aea4fa41551a30e30af803 in actions/checkout/action.yml
2. Pinned actions/upload-artifact@v7 to SHA 043fb46d1a93c77aae656e7c1c64a875d1fc6a0a in actions/upload-artifact/action.yml
3. Pinned check-spelling/publish-coverage@main to SHA e4b5547dea6a84ea9981fbd60c8a8601f62823f3 in actions/publish-coverage/action.yml
4. Fixed script injection in 'Parse alternate engine' step: moved ${{ inputs.alternate_engine }} to env block as ALTERNATE_ENGINE, rewrote run block to use $ALTERNATE_ENGINE
5. Fixed github-env-injection in 'Parse alternate engine' step: sanitized value with tr -d '\n\r' before writing to $GITHUB_OUTPUT using printf
6. Fixed unsanitized env write in 'Shim path and local actions' step: sanitized THIS_GITHUB_JOB_ID with tr -d '\n\r' before writing to $GITHUB_ENV using printf

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection finding in the 'Save SHA' step of action.yml. Added sanitization of PRIVATE_CHECKOUT_REF before writing to $GITHUB_ENV: added `safe_ref=$(printf '%s' "$PRIVATE_CHECKOUT_REF" | tr -d '\n\r')` and changed the echo to use `$safe_ref` instead of the raw `$PRIVATE_CHECKOUT_REF`. This prevents newline injection attacks via crafted PR number values in the event payload.

