<!-- markdownlint-disable -->

# Hardening Report: check-spelling--check-spelling/v0.0.22

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **check-spelling--check-spelling/v0.0.22** was hardened automatically. 8 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple `run:` blocks in action.yml directly interpolate `${{ ... }}` expressions into shell commands, violating sub-rule (a). An attacker who controls the interpolated value can inject arbitrary shell commands.

1. `parse alternate engine` step: `echo "repo=$(echo '${{ inputs.alternate_engine }}' | perl ...)" >> "$GITHUB_OUTPUT"` — `inputs.alternate_engine` is interpolated directly into the shell command string.

2. `save sha` step: `cd "${{ inputs.experimental_path }}"` and `PRIVATE_SARIF_REF="refs/pull/${{ github.event.pull_request.number }}/merge"` — both `inputs.experimental_path` and `github.event.pull_request.number` are interpolated directly.

3. `perl configuration` step: `echo "${{ steps.hash-dictionaries.outputs.perl-libraries }}" | tr ...` — a step output is interpolated directly.

4. `install perl modules` step: `for module in ${{ steps.perl-config.outputs.perl-modules }}; do` — a step output is interpolated directly into a for-loop, allowing word-splitting and shell metacharacter injection.

5. `Shim Sarif` step: `cd "${{ inputs.experimental_path }}"` — `inputs.experimental_path` is interpolated directly.

Locations:

- `action.yml:270`
- `action.yml:271`
- `action.yml:323`
- `action.yml:325`
- `action.yml:374`
- `action.yml:399`
- `action.yml:456`

### github-env-injection (severity: high)

Two `run:` blocks write untrusted values to GitHub special environment files without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`).

1. `parse alternate engine` step: `echo "repo=$(echo '${{ inputs.alternate_engine }}' | perl ...)" >> "$GITHUB_OUTPUT"` and `echo "branch=$(echo '${{ inputs.alternate_engine }}' | perl ...)" >> "$GITHUB_OUTPUT"` — the user-controlled `inputs.alternate_engine` value is written directly to `$GITHUB_OUTPUT`. A newline in the input can inject additional key=value pairs into the output file.

2. `save sha` step: `PRIVATE_SARIF_REF="refs/pull/${{ github.event.pull_request.number }}/merge"` followed by `echo "PRIVATE_SARIF_REF=$PRIVATE_SARIF_REF" >> "$GITHUB_ENV"` — the PR number from the GitHub event context is written to `$GITHUB_ENV` without sanitization. Although PR numbers are typically integers, the value flows through template substitution before the shell sees it.

Locations:

- `action.yml:270`
- `action.yml:271`
- `action.yml:325`
- `action.yml:326`

### unpinned-uses (severity: high)

All `uses:` references in action.yml use mutable version tags instead of immutable 40-character commit SHA hashes. If any of these upstream actions is compromised or the tag is moved, the action will silently execute attacker-controlled code. Failing references:
- `actions/checkout@v4` (appears twice)
- `check-spelling/actions-checkout@v4`
- `check-spelling/checkout-merge@v0.0.4`
- `actions/download-artifact@v3`
- `actions/cache/restore@v3` (appears twice)
- `actions/cache/save@v3` (appears twice)
- `actions/upload-artifact@v3` (appears twice)
- `github/codeql-action/upload-sarif@v2`

Locations:

- `action.yml:278`
- `action.yml:295`
- `action.yml:308`
- `action.yml:320`
- `action.yml:340`
- `action.yml:355`
- `action.yml:365`
- `action.yml:380`
- `action.yml:390`
- `action.yml:415`
- `action.yml:425`
- `action.yml:435`
- `action.yml:445`
- `action.yml:465`

### unsafe-shell (severity: high)

The `install perl modules` step in action.yml pipes remote content directly to a Perl interpreter: `curl -s -S -L https://cpanmin.us | perl - --sudo App::cpanminus`. If the remote URL is compromised (via DNS hijacking, MITM, or supply-chain attack), arbitrary Perl code will be executed with sudo privileges on the runner. The script should be downloaded to a file first, its integrity verified (e.g., via checksum), and then executed separately.

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

Fixed all security findings in hardened/action/action.yml:

1. script-injection + github-env-injection (parse alternate engine): Moved inputs.alternate_engine to env block as ALTERNATE_ENGINE, sanitized with tr -d '\n\r' before writing to GITHUB_OUTPUT.

2. script-injection + github-env-injection (save sha): Moved inputs.experimental_path and github.event.pull_request.number to env block, sanitized PR_NUMBER before writing to GITHUB_ENV.

3. script-injection (perl configuration): Moved steps.hash-dictionaries.outputs.perl-libraries to env block as PERL_LIBRARIES.

4. script-injection (install perl modules): Moved steps.perl-config.outputs.perl-modules to env block as PERL_MODULES, used xargs-based null-delimited tokenization loop.

5. unsafe-shell (install perl modules): Replaced 'curl ... | perl - --sudo App::cpanminus' with download-to-tempfile then execute pattern.

6. script-injection (Shim Sarif): Moved inputs.experimental_path to env block as EXPERIMENTAL_PATH.

7. unpinned-uses: Pinned all 14 action references to full commit SHAs: actions/checkout@v4 (x2) → 11d5960a..., check-spelling/actions-checkout@v4 → cb50106c..., check-spelling/checkout-merge@v0.0.4 → 3aa4a3df..., actions/download-artifact@v3 → 9bc31d5c..., actions/cache/restore@v3 (x3) → 6f8efc29..., actions/cache/save@v3 (x3) → 6f8efc29..., actions/upload-artifact@v3 (x2) → ff15f030..., github/codeql-action/upload-sarif@v2 → b8d3b6e8...

### Iteration 2

**Fixes applied:** github-env-injection, script-injection

**Notes:**

Fixed two findings in hardened/action/action.yml:
1. github-env-injection (line 430): Added `safe_perl_modules=$(printf '%s' "$perl_modules" | tr -d '\n\r')` to sanitize the perl_modules value before writing to $GITHUB_OUTPUT. The sanitized variable `safe_perl_modules` is now used in the `echo "perl-modules=..."` line instead of the unsanitized `perl_modules`.
2. script-injection (line 455): Changed `perl -e "use $module; 1;"` to `MODULE="$module" perl -e 'eval "use $ENV{MODULE}; 1" or die $@;'`. The module name is now passed via an environment variable (MODULE) rather than being shell-interpolated into the perl -e string. The eval happens inside Perl, so shell metacharacters in the module name cannot cause shell command injection.

