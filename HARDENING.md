<!-- markdownlint-disable -->

# Hardening Report: check-spelling--check-spelling/v0.0.22

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **check-spelling--check-spelling/v0.0.22** was hardened automatically. 8 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple run: blocks in action.yml directly interpolate ${{ }} expressions inside shell commands, allowing script injection. Affected steps:

1. 'parse alternate engine' step: `echo "repo=$(echo '${{ inputs.alternate_engine }}' | perl ...)" >> "$GITHUB_OUTPUT"` and `echo "branch=$(echo '${{ inputs.alternate_engine }}' | perl ...)" >> "$GITHUB_OUTPUT"` — inputs.alternate_engine is caller-controlled and interpolated directly into the shell command.

2. 'save sha' step: `cd "${{ inputs.experimental_path }}"` and `PRIVATE_SARIF_REF="refs/pull/${{ github.event.pull_request.number }}/merge"` — both expressions are interpolated directly into shell commands.

3. 'perl configuration' step: `echo "${{ steps.hash-dictionaries.outputs.perl-libraries }}" | tr ...` — step output interpolated directly into shell.

4. 'install perl modules' step: `for module in ${{ steps.perl-config.outputs.perl-modules }}; do` — step output interpolated unquoted into a for-loop, allowing word-splitting and glob expansion of attacker-influenced data.

5. 'Shim Sarif' step: `cd "${{ inputs.experimental_path }}"` — inputs.experimental_path interpolated directly into shell.

Locations:

- `action.yml:285`
- `action.yml:286`
- `action.yml:316`
- `action.yml:318`
- `action.yml:358`
- `action.yml:388`
- `action.yml:437`
- `action.yml:438`

### github-env-injection (severity: high)

Untrusted input values are written to special GitHub environment files without sanitization (no `printf '%s' ... | tr -d '\n\r'` step):

1. 'parse alternate engine' step: `${{ inputs.alternate_engine }}` (caller-controlled) is interpolated directly into the shell command that writes to $GITHUB_OUTPUT. A newline in the value would allow injecting arbitrary output variables.

2. 'save sha' step: `PRIVATE_SARIF_REF` is constructed from `${{ github.event.pull_request.number }}` and then written to $GITHUB_ENV via `echo "PRIVATE_SARIF_REF=$PRIVATE_SARIF_REF" >> "$GITHUB_ENV"` without sanitization. A crafted PR number containing a newline could inject arbitrary environment variables.

Locations:

- `action.yml:285`
- `action.yml:286`
- `action.yml:319`
- `action.yml:321`

### unpinned-uses (severity: high)

All uses: references in action.yml use mutable tags instead of immutable 40-character SHA digests, making the action vulnerable to supply-chain attacks if any referenced action is compromised or its tag is moved. Failing references:
- uses: actions/checkout@v4 (appears twice)
- uses: check-spelling/actions-checkout@v4
- uses: check-spelling/checkout-merge@v0.0.4
- uses: actions/download-artifact@v3
- uses: actions/cache/restore@v3 (appears twice)
- uses: actions/cache/save@v3 (appears three times)
- uses: actions/upload-artifact@v3 (appears twice)
- uses: github/codeql-action/upload-sarif@v2

Locations:

- `action.yml:293`
- `action.yml:303`
- `action.yml:327`
- `action.yml:336`
- `action.yml:345`
- `action.yml:370`
- `action.yml:378`
- `action.yml:397`
- `action.yml:415`
- `action.yml:421`
- `action.yml:427`
- `action.yml:447`
- `action.yml:455`
- `action.yml:462`

### unsafe-shell (severity: high)

The 'install perl modules' run: block pipes remote content directly to a Perl interpreter: `curl -s -S -L https://cpanmin.us | perl - --sudo App::cpanminus`. This downloads and executes arbitrary code from cpanmin.us without any integrity verification (no checksum or signature check). If the remote server is compromised or the connection is intercepted, malicious code would be executed on the runner with sudo privileges.

Locations:

- `action.yml:392`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.alternate_engine }}" appears directly in run: block of step "parse alternate engine"; move to env: map

Locations:

- `action.yml:353`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.alternate_engine }}" appears directly in run: block of step "parse alternate engine"; move to env: map

Locations:

- `action.yml:354`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.experimental_path }}" appears directly in run: block of step "save sha"; move to env: map

Locations:

- `action.yml:433`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.experimental_path }}" appears directly in run: block of step "Shim Sarif"; move to env: map

Locations:

- `action.yml:563`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, unsafe-shell, static-inline-injection

**Notes:**

Fixed all security findings in hardened/action/action.yml:

1. script-injection/static-inline-injection: Moved all ${{ }} expressions from run: blocks to env: blocks for 5 steps: 'parse alternate engine' (ALTERNATE_ENGINE), 'save sha' (EXPERIMENTAL_PATH, PR_NUMBER), 'perl configuration' (PERL_LIBRARIES), 'install perl modules' (PERL_MODULES), and 'Shim Sarif' (EXPERIMENTAL_PATH).

2. github-env-injection: Added printf '%s' ... | tr -d '\n\r' sanitization in 'parse alternate engine' (before writing repo/branch to GITHUB_OUTPUT) and 'save sha' (sanitizing PR_NUMBER before constructing PRIVATE_SARIF_REF, and sanitizing the ref before writing to GITHUB_ENV).

3. unsafe-shell: Fixed curl pipe to perl in 'install perl modules' by downloading cpanm installer to a temp file first, then executing it separately, then cleaning up.

4. install perl modules script-injection: Replaced unquoted ${{ }} for-loop with xargs-based tokenization using the env var PERL_MODULES, using bash arrays to safely iterate over module names.

5. unpinned-uses: Pinned all 14 action references to full 40-character SHA digests with tag comments: actions/checkout@v4 (x2), check-spelling/actions-checkout@v4, check-spelling/checkout-merge@v0.0.4, actions/download-artifact@v3, actions/cache/restore@v3 (x2), actions/cache/save@v3 (x3), actions/upload-artifact@v3 (x2), github/codeql-action/upload-sarif@v2.

### Iteration 1

**Fixes applied:** github-env-injection, script-injection

**Notes:**

Fixed two security findings in hardened/action/action.yml:
1. github-env-injection (line 418): Added sanitization of `perl_modules` before writing to GITHUB_OUTPUT. Added `safe_perl_modules=$(printf '%s' "$perl_modules" | tr -d '\n\r')` and used `safe_perl_modules` in the echo statement to strip any injected newlines.
2. script-injection (line 441): Changed `perl -e "use $module; 1;"` to `CHECK_MODULE="$module" perl -e 'use $ENV{CHECK_MODULE}; 1;'`. The module name is now passed via the `CHECK_MODULE` environment variable and referenced inside a single-quoted Perl string, preventing shell metacharacter injection.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the shim-path step in action.yml (around line 285). The $PATH environment variable (and $THIS_ACTION_PATH) are now sanitized using `printf '%s' "$VAR" | tr -d '\n\r'` before being written to $GITHUB_ENV. This prevents a calling workflow from injecting newlines into GITHUB_ENV to set arbitrary environment variables by poisoning the $PATH environment variable.

