<!-- markdownlint-disable -->

# Hardening Report: check-spelling--check-spelling/v0.0.22

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **check-spelling--check-spelling/v0.0.22** was hardened automatically. 8 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple run: blocks directly interpolate ${{ ... }} expressions into shell commands without routing through env: variables.

1. 'parse alternate engine' step: `echo "repo=$(echo '${{ inputs.alternate_engine }}' | perl ...)" >> "$GITHUB_OUTPUT"` and `echo "branch=$(echo '${{ inputs.alternate_engine }}' | perl ...)" >> "$GITHUB_OUTPUT"` — inputs.alternate_engine is directly interpolated in shell.

2. 'save sha' step: `cd "${{ inputs.experimental_path }}"` and `PRIVATE_SARIF_REF="refs/pull/${{ github.event.pull_request.number }}/merge"` — both inputs.experimental_path and github.event.pull_request.number are directly interpolated.

3. 'perl configuration' step: `echo "${{ steps.hash-dictionaries.outputs.perl-libraries }}" |tr ...` — steps output directly interpolated in shell.

4. 'install perl modules' step: `for module in ${{ steps.perl-config.outputs.perl-modules }}; do` — steps output directly interpolated in an unquoted for-loop (sub-rule b: unquoted expansion).

5. 'Shim Sarif' step: `cd "${{ inputs.experimental_path }}"` — inputs.experimental_path directly interpolated.

Locations:

- `action.yml:275`
- `action.yml:276`
- `action.yml:334`
- `action.yml:336`
- `action.yml:374`
- `action.yml:397`
- `action.yml:450`

### github-env-injection (severity: high)

Multiple run: blocks write values derived from untrusted inputs/github context to $GITHUB_OUTPUT or $GITHUB_ENV without the required sanitization step (printf '%s' ... | tr -d '\n\r').

1. 'parse alternate engine' step writes ${{ inputs.alternate_engine }} (caller-controlled) directly to $GITHUB_OUTPUT via echo without sanitization. An attacker-controlled value containing newlines could inject additional key=value pairs.

2. 'save sha' step: constructs PRIVATE_SARIF_REF using ${{ github.event.pull_request.number }} and writes it to $GITHUB_ENV without sanitization. Also writes PRIVATE_SARIF_SHA to $GITHUB_ENV.

3. 'perl configuration' step writes ${{ steps.hash-dictionaries.outputs.perl-libraries }} (derived from caller-controlled inputs) to $GITHUB_OUTPUT without sanitization.

Locations:

- `action.yml:275`
- `action.yml:276`
- `action.yml:336`
- `action.yml:337`
- `action.yml:374`

### unpinned-uses (severity: high)

All uses: references in action.yml use mutable tags or version strings instead of pinned 40-character SHA digests, making the action vulnerable to supply-chain attacks if any referenced action is compromised or its tag is moved.

Failing references:
- uses: actions/checkout@v4 (alternate-engine step)
- uses: check-spelling/actions-checkout@v4 (checkout-replacement step)
- uses: actions/checkout@v4 (checkout step)
- uses: check-spelling/checkout-merge@v0.0.4 (checkout-merge step)
- uses: actions/download-artifact@v3 (retrieve-comment step)
- uses: actions/cache/restore@v3 (retrieve-dictionaries step)
- uses: actions/cache/restore@v3 (retrieve-perl-libraries-arch step)
- uses: actions/cache/restore@v3 (retrieve-perl-libraries-lib step)
- uses: actions/cache/save@v3 (save-dictionaries step)
- uses: actions/cache/save@v3 (save-perl-libraries-arch step)
- uses: actions/cache/save@v3 (save-perl-libraries-lib step)
- uses: actions/upload-artifact@v3 (store-comment step)
- uses: actions/upload-artifact@v3 (artifact-sarif step)
- uses: github/codeql-action/upload-sarif@v2 (upload-sarif step)

Locations:

- `action.yml:278`
- `action.yml:290`
- `action.yml:302`
- `action.yml:316`
- `action.yml:348`
- `action.yml:358`
- `action.yml:385`
- `action.yml:390`
- `action.yml:413`
- `action.yml:417`
- `action.yml:423`
- `action.yml:430`
- `action.yml:455`
- `action.yml:462`

### unsafe-shell (severity: high)

The 'install perl modules' step pipes remote content directly to a Perl interpreter: `curl -s -S -L https://cpanmin.us | perl - --sudo App::cpanminus`. This downloads and executes arbitrary code from a remote URL without any integrity verification (no checksum, no pinned version). If cpanmin.us is compromised or the response is tampered with in transit, malicious code would be executed with sudo privileges on the runner.

Locations:

- `action.yml:403`

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

Fixed all findings in action.yml:

1. script-injection/static-inline-injection: Moved all ${{ }} expressions from run: blocks to env: variables. 'parse alternate engine' uses ALTERNATE_ENGINE; 'save sha' uses EXPERIMENTAL_PATH and PR_NUMBER; 'perl configuration' uses PERL_LIBRARIES; 'install perl modules' uses PERL_MODULES (also fixed unquoted for-loop using xargs+read loop); 'Shim Sarif' uses EXPERIMENTAL_PATH.

2. github-env-injection: Added tr -d '\n\r' sanitization for all values written to GITHUB_OUTPUT and GITHUB_ENV: repo/branch in parse-alternate-engine, PRIVATE_SARIF_REF/PRIVATE_SARIF_SHA in save-sha, and all four outputs in perl-config.

3. unpinned-uses: Pinned all 14 action references to full 40-character SHA digests using lookup_action_sha: actions/checkout@v4, check-spelling/actions-checkout@v4, check-spelling/checkout-merge@v0.0.4, actions/download-artifact@v3, actions/cache/restore@v3 (3x), actions/cache/save@v3 (3x), actions/upload-artifact@v3 (2x), github/codeql-action/upload-sarif@v2.

4. unsafe-shell: Replaced 'curl ... | perl - --sudo App::cpanminus' with download-then-execute pattern: curl to temp file, then perl executes the file, then temp file is removed.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed two script-injection findings in hardened/action/action.yml:
1. raise-merge-failure step (line ~371): Changed `echo ::error "::$MESSAGE"` to `echo "::error ::${MESSAGE}"` so the entire argument is double-quoted and $MESSAGE cannot cause word-splitting or injection.
2. install perl modules step (line ~438): Changed `perl -e "use $module; 1;"` to `PERL_MODULE_NAME="$module" perl -e 'my $m=$ENV{PERL_MODULE_NAME}; (my $f=$m)=~s|::|/|g; require "$f.pm"; 1;'` — the module name is passed via an environment variable and accessed safely in Perl via $ENV{PERL_MODULE_NAME}, with a single-quoted -e string to prevent shell interpolation. Used `require` instead of `use` since `use` is compile-time and cannot accept a runtime expression.

