<!-- markdownlint-disable -->

# Hardening Report: check-spelling--check-spelling/v0.0.24

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **check-spelling--check-spelling/v0.0.24** was hardened automatically. 8 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a) violation: The 'parse alternate engine' step directly interpolates ${{ inputs.alternate_engine }} inside a run: shell command string. An attacker-controlled value is embedded verbatim into the shell before quoting, enabling command injection. Offending lines: `echo "repo=$(echo '${{ inputs.alternate_engine }}' | perl -pe 's/\@.*//')" >> "$GITHUB_OUTPUT"` and `echo "branch=$(echo '${{ inputs.alternate_engine }}' | perl -ne 'next unless s/.*\@//; print')" >> "$GITHUB_OUTPUT"`

Locations:

- `action.yml:296`

### script-injection (severity: high)

Rule (a) violation: The 'save sha' step directly interpolates ${{ github.event.pull_request.number }} inside a run: shell command string. This GitHub context value is embedded verbatim into the shell before quoting, enabling command injection. Offending line: `PRIVATE_SARIF_REF="refs/pull/${{ github.event.pull_request.number }}/merge"`

Locations:

- `action.yml:371`

### github-env-injection (severity: high)

The 'parse alternate engine' step writes a value derived from ${{ inputs.alternate_engine }} (an untrusted input) directly to $GITHUB_OUTPUT without the required sanitization step (printf '%s' ... | tr -d '\n\r'). The expression is interpolated directly into the shell command and the result is echoed to $GITHUB_OUTPUT, allowing newline injection to set arbitrary output variables.

Locations:

- `action.yml:296`

### github-env-injection (severity: high)

The 'save sha' step writes a value derived from ${{ github.event.pull_request.number }} (a GitHub context value) to $GITHUB_ENV without the required sanitization step (printf '%s' ... | tr -d '\n\r'). The expression is interpolated directly into the shell variable PRIVATE_SARIF_REF, which is then echoed to $GITHUB_ENV, allowing newline injection to set arbitrary environment variables.

Locations:

- `action.yml:371`

### unpinned-uses (severity: high)

Multiple sub-action files use mutable tag-based references instead of pinned 40-character SHA digests, making them vulnerable to supply-chain attacks if the referenced tags are moved or overwritten. Failing references: `uses: actions/checkout@v4` (actions/checkout/action.yml), `uses: check-spelling/checkout-merge@v0.0.6` (actions/checkout-merge/action.yml), `uses: actions/upload-artifact@v4` (actions/upload-artifact/action.yml), `uses: github/codeql-action/upload-sarif@v3` (actions/upload-sarif/action.yml).

Locations:

- `actions/checkout/action.yml:48`
- `actions/checkout-merge/action.yml:28`
- `actions/upload-artifact/action.yml:28`
- `actions/upload-sarif/action.yml:44`

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

1. 'parse alternate engine' step (action.yml): Moved `${{ inputs.alternate_engine }}` to env var `ALTERNATE_ENGINE`, used `printf '%s' "$ALTERNATE_ENGINE" | perl ... | tr -d '\n\r'` to sanitize before writing repo/branch to $GITHUB_OUTPUT. Fixes script-injection, static-inline-injection (x2), and github-env-injection.

2. 'save sha' step (action.yml): Moved `${{ github.event.pull_request.number }}` to env var `PR_NUMBER`, sanitized with printf/tr before constructing PRIVATE_SARIF_REF and writing to $GITHUB_ENV. Fixes script-injection and github-env-injection.

3. 'shim' step (action.yml): Added `safe_job_id=$(printf '%s' "$THIS_GITHUB_JOB_ID" | tr -d '\n\r')` before writing to $GITHUB_ENV. Fixes static-unsanitized-env-write.

4. Pinned all 4 unpinned sub-action uses references to full 40-char SHAs: actions/checkout@34e114876b0b11c390a56381ad16ebd13914f8d5, check-spelling/checkout-merge@29407b1b8660c562313ed65c81feab3e3255c03c, actions/upload-artifact@ea165f8d65b6e75b540449e92b4886f43607fa02, github/codeql-action/upload-sarif@b7351df727350dca84cb9d725d57dcf5bc82ba26.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in the retrieve-comment step of action.yml (line 398). The $suffix variable (from inputs.report_title_suffix) was expanded inside a double-quoted string in the gh run download command, allowing command injection via $(...) or backtick expressions. Fix: added sanitization step `safe_suffix=$(printf '%s' "$suffix" | tr -d '\n\r$()`\\')` to strip injection-enabling characters, then constructed the artifact name as `artifact_name="check-spelling-comment-$safe_suffix"` and used `"$artifact_name"` in the command instead of the unsanitized `"check-spelling-comment-$suffix"`.

### Iteration 1

**Fixes applied:** unpinned-uses, unsafe-shell, script-injection

**Notes:**

Fixed all findings across three workflow files:

1. spelling.yml: Pinned actions/checkout@v4 (4 occurrences) to SHA 34e114876b0b11c390a56381ad16ebd13914f8d5 and check-spelling/checkout-merge@v0.0.6 (1 occurrence) to SHA 29407b1b8660c562313ed65c81feab3e3255c03c.

2. test.yml: Pinned actions/checkout@v4 (4 occurrences) to SHA 34e114876b0b11c390a56381ad16ebd13914f8d5 and actions/upload-artifact@v4 (2 occurrences) to SHA ea165f8d65b6e75b540449e92b4886f43607fa02. Fixed unsafe shell by replacing `curl -s -S -L https://cpanmin.us | perl - --sudo App::cpanminus` with a two-step download-then-execute pattern. Fixed script injection by replacing unquoted `for module in $required_perl_modules` with `IFS=' ' read -ra perl_module_list <<< "$required_perl_modules"` and `for module in "${perl_module_list[@]}"`.

3. test-server.yml: Pinned actions/checkout@v4 (1 occurrence) to SHA 34e114876b0b11c390a56381ad16ebd13914f8d5.

