<!-- markdownlint-disable -->

# Hardening Report: check-spelling--check-spelling/v0.0.22

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **check-spelling--check-spelling/v0.0.22** was hardened automatically. 8 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks in action.yml directly interpolate ${{ ... }} expressions inside shell command strings, violating sub-rule (a). This allows an attacker-controlled value to be executed as shell code before the shell ever sees it.

1. 'parse alternate engine' step: `echo "repo=$(echo '${{ inputs.alternate_engine }}' | perl -pe 's/\@.*//')" >> "$GITHUB_OUTPUT"` — inputs.alternate_engine is interpolated directly.
2. 'save sha' step: `cd "${{ inputs.experimental_path }}"` and `PRIVATE_SARIF_REF="refs/pull/${{ github.event.pull_request.number }}/merge"` — both inputs and github context interpolated directly.
3. 'perl configuration' step: `echo "${{ steps.hash-dictionaries.outputs.perl-libraries }}" |tr ...` — step output interpolated directly in shell.
4. 'install perl modules' step: `for module in ${{ steps.perl-config.outputs.perl-modules }}; do` — step output interpolated directly in a for loop.
5. 'Shim Sarif' step: `cd "${{ inputs.experimental_path }}"` — inputs interpolated directly.

Locations:

- `action.yml:271`
- `action.yml:272`
- `action.yml:310`
- `action.yml:312`
- `action.yml:356`
- `action.yml:381`
- `action.yml:430`

### github-env-injection (severity: high)

Multiple run: blocks write untrusted input values to $GITHUB_OUTPUT or $GITHUB_ENV without the required sanitization step (printf '%s' ... | tr -d '\n\r').

1. 'parse alternate engine' step writes ${{ inputs.alternate_engine }} (caller-controlled) directly to $GITHUB_OUTPUT: `echo "repo=$(echo '${{ inputs.alternate_engine }}' | perl ...)" >> "$GITHUB_OUTPUT"`. A newline in the input value can inject arbitrary output variables.
2. 'save sha' step writes a value derived from ${{ github.event.pull_request.number }} to $GITHUB_ENV: `echo "PRIVATE_SARIF_REF=$PRIVATE_SARIF_REF" >> "$GITHUB_ENV"` where PRIVATE_SARIF_REF contains the unsanitized PR number. A crafted PR number containing a newline could inject arbitrary environment variables.

Locations:

- `action.yml:271`
- `action.yml:272`
- `action.yml:312`
- `action.yml:313`

### unpinned-uses (severity: high)

All uses: references in action.yml use mutable version tags instead of immutable 40-character SHA digests, making the action vulnerable to supply-chain attacks if any referenced action is compromised or its tag is moved.

Failing references:
- uses: actions/checkout@v4 (appears twice)
- uses: check-spelling/actions-checkout@v4
- uses: check-spelling/checkout-merge@v0.0.4
- uses: actions/download-artifact@v3
- uses: actions/cache/restore@v3 (appears twice)
- uses: actions/cache/save@v3 (appears twice)
- uses: actions/upload-artifact@v3 (appears twice)
- uses: github/codeql-action/upload-sarif@v2

Locations:

- `action.yml:280`
- `action.yml:299`
- `action.yml:305`
- `action.yml:320`
- `action.yml:336`
- `action.yml:363`
- `action.yml:370`
- `action.yml:376`
- `action.yml:395`
- `action.yml:407`
- `action.yml:413`
- `action.yml:421`
- `action.yml:437`
- `action.yml:447`

### unsafe-shell (severity: high)

The 'install perl modules' run: block pipes remote content directly to a Perl interpreter without first downloading and verifying it: `curl -s -S -L https://cpanmin.us | perl - --sudo App::cpanminus`. If the remote server is compromised or the connection is intercepted, arbitrary code will be executed with sudo privileges on the runner.

Locations:

- `action.yml:386`

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

1. script-injection/static-inline-injection: Moved all ${{ inputs.alternate_engine }}, ${{ inputs.experimental_path }}, ${{ github.event.pull_request.number }}, ${{ steps.hash-dictionaries.outputs.perl-libraries }}, and ${{ steps.perl-config.outputs.perl-modules }} expressions from run: blocks into env: blocks, referencing them as plain shell variables.

2. github-env-injection: Added sanitization with `printf '%s' "$VAR" | tr -d '\n\r'` before writing values to $GITHUB_OUTPUT and $GITHUB_ENV in the 'parse alternate engine' and 'save sha' steps.

3. unpinned-uses: Pinned all 14 action references to full 40-character SHA digests using lookup_action_sha.

4. unsafe-shell: Replaced `curl ... | perl - --sudo App::cpanminus` with a safe pattern that downloads cpanmin.us to a temp file (mktemp), executes it separately, then removes the temp file.

### Iteration 2

**Fixes applied:** github-env-injection, script-injection

**Notes:**

Fixed two security findings in hardened/action/action.yml:

1. github-env-injection (line 478): In the 'perl configuration' step, sanitized perl_modules, installsitearch, and installsitelib values with `printf '%s' "$VAR" | tr -d '\n\r'` before writing to $GITHUB_OUTPUT to prevent newline injection.

2. script-injection (line 504): In the 'install perl modules' step, replaced unquoted $PERL_MODULES expansion in a for loop with safe xargs-based tokenization into a bash array. Changed cpan_modules from a string to an array, replaced backtick command substitution with $(...), and used proper quoted array expansion "${cpan_modules[@]}" in the final cpanm invocation.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in the 'install perl modules' step (action.yml ~line 430). The vulnerable pattern `perl -e "use $module; 1;"` interpolated a user-controlled value ($module, derived from inputs.extra_dictionaries via steps.perl-config.outputs.perl-modules) into a double-quoted shell string passed to perl -e. Replaced with `MODULE="$module" perl -e 'use $ENV{MODULE}; 1;'` — the module name is now passed as an environment variable and accessed via Perl's $ENV{MODULE}, so shell metacharacters in the value cannot inject code into the perl -e expression.

