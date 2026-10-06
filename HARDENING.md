<!-- markdownlint-disable -->

# Hardening Report: check-spelling--check-spelling/v0.0.22

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **check-spelling--check-spelling/v0.0.22** was hardened automatically. 8 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks in action.yml directly interpolate ${{ ... }} expressions into shell commands (sub-rule a), enabling script injection.

1. 'parse alternate engine' step: `echo "repo=$(echo '${{ inputs.alternate_engine }}' | perl ...)" >> "$GITHUB_OUTPUT"` — inputs.alternate_engine is caller-controlled and interpolated directly into the shell command string.

2. 'save sha' step: `cd "${{ inputs.experimental_path }}"` and `PRIVATE_SARIF_REF="refs/pull/${{ github.event.pull_request.number }}/merge"` — both inputs.experimental_path and github.event.pull_request.number are interpolated directly.

3. 'perl configuration' step: `echo "${{ steps.hash-dictionaries.outputs.perl-libraries }}" | tr ...` — a step output is interpolated directly into the shell.

4. 'install perl modules' step: `for module in ${{ steps.perl-config.outputs.perl-modules }}; do` — a step output is interpolated directly into a for-loop.

5. 'Shim Sarif' step: `cd "${{ inputs.experimental_path }}"` — inputs.experimental_path interpolated directly.

Locations:

- `action.yml:313`
- `action.yml:314`
- `action.yml:340`
- `action.yml:342`
- `action.yml:380`
- `action.yml:399`
- `action.yml:453`

### github-env-injection (severity: high)

Multiple run: blocks write values derived from untrusted inputs to $GITHUB_OUTPUT or $GITHUB_ENV without the required sanitization step (printf '%s' ... | tr -d '\n\r').

1. 'parse alternate engine' step writes ${{ inputs.alternate_engine }}-derived values (repo= and branch=) directly to $GITHUB_OUTPUT with no newline sanitization. An attacker-controlled input containing newlines could inject arbitrary key=value pairs.

2. 'save sha' step constructs PRIVATE_SARIF_REF from `${{ github.event.pull_request.number }}` and writes it to $GITHUB_ENV without sanitization: `echo "PRIVATE_SARIF_REF=$PRIVATE_SARIF_REF" >> "$GITHUB_ENV"`.

Locations:

- `action.yml:313`
- `action.yml:314`
- `action.yml:342`

### unpinned-uses (severity: high)

All uses: references in action.yml use mutable version tags instead of pinned 40-character SHA digests, making the action vulnerable to supply-chain attacks.

Failing references:
- actions/checkout@v4 (appears twice)
- check-spelling/actions-checkout@v4
- check-spelling/checkout-merge@v0.0.4
- actions/download-artifact@v3
- actions/cache/restore@v3 (appears twice)
- actions/cache/save@v3 (appears three times)
- actions/upload-artifact@v3 (appears twice)
- github/codeql-action/upload-sarif@v2

Locations:

- `action.yml:320`
- `action.yml:325`
- `action.yml:330`
- `action.yml:355`
- `action.yml:360`
- `action.yml:375`
- `action.yml:385`
- `action.yml:390`
- `action.yml:415`
- `action.yml:420`
- `action.yml:425`
- `action.yml:430`
- `action.yml:440`
- `action.yml:460`

### unsafe-shell (severity: high)

The 'install perl modules' run: block pipes remote content directly to a Perl interpreter without first saving it to a file: `curl -s -S -L https://cpanmin.us | perl - --sudo App::cpanminus`. This allows arbitrary code from cpanmin.us to execute immediately in the runner with no integrity verification.

Locations:

- `action.yml:404`

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

Fixed all findings in hardened/action/action.yml:

1. script-injection/static-inline-injection: Moved all ${{ }} expressions from run: blocks to env: blocks - inputs.alternate_engine→ALTERNATE_ENGINE, inputs.experimental_path→EXPERIMENTAL_PATH, github.event.pull_request.number→PR_NUMBER, steps.hash-dictionaries.outputs.perl-libraries→PERL_LIBRARIES, steps.perl-config.outputs.perl-modules→PERL_MODULES.

2. github-env-injection: Added sanitization (printf '%s' | tr -d '\n\r') for values written to GITHUB_OUTPUT (repo, branch in parse-alternate-engine) and GITHUB_ENV (PRIVATE_SARIF_REF in save-sha).

3. unpinned-uses: Pinned all 14 action references to full 40-character SHA digests with tag comments: actions/checkout@v4→11d5960a (×2), check-spelling/actions-checkout@v4→cb50106c, check-spelling/checkout-merge@v0.0.4→3aa4a3df, actions/download-artifact@v3→9bc31d5c, actions/cache/restore@v3→6f8efc29 (×2), actions/cache/save@v3→6f8efc29 (×3), actions/upload-artifact@v3→ff15f030 (×2), github/codeql-action/upload-sarif@v2→b8d3b6e8.

4. unsafe-shell: Replaced `curl ... | perl - --sudo App::cpanminus` with safe pattern: download to mktemp file, execute separately, then remove the temp file.

### Iteration 2

**Fixes applied:** github-env-injection, script-injection

**Notes:**

Fixed two security findings in hardened/action/action.yml:

1. github-env-injection (line 394): In the 'perl configuration' step, added `safe_perl_modules=$(printf '%s' "$perl_modules" | tr -d '\n\r')` before writing to $GITHUB_OUTPUT, and used `$safe_perl_modules` in the echo statement to prevent newline injection.

2. script-injection (lines 419, 420, 430): In the 'install perl modules' step:
   - Replaced unquoted `for module in $PERL_MODULES` with a guarded xargs-based tokenization into a bash array (`modules=()`), using `while IFS= read -r -d '' t; do modules+=("$t"); done < <(printf '%s' "$PERL_MODULES" | xargs printf '%s\0')` to safely split the space-separated module list
   - Changed `cpan_modules` from a string to a bash array (`cpan_modules=()`), appending modules with `cpan_modules+=("$module")`
   - Changed the condition from `[ -n "$cpan_modules" ]` to `[ "${#cpan_modules[@]}" -gt 0 ]`
   - Replaced backtick `perl \`command -v cpanm\`` with `perl "$(command -v cpanm)"` and expanded the array with `"${cpan_modules[@]}"` instead of unquoted `$cpan_modules`

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in the 'install perl modules' step (action.yml ~line 543). The vulnerable `perl -e "use $module; 1;"` (double-quoted, shell-expanded) was replaced with `MODULE="$module" perl -e 'use $ENV{MODULE}; 1;'` (single-quoted perl script, module name passed via environment variable). This prevents any shell metacharacters in the module name from being interpreted by the shell, as the value is now accessed purely within Perl via `$ENV{MODULE}`.

