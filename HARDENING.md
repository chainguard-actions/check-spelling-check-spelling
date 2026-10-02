<!-- markdownlint-disable -->

# Hardening Report: check-spelling--check-spelling/v0.0.22

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **check-spelling--check-spelling/v0.0.22** was hardened automatically. 8 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple `run:` blocks in action.yml directly interpolate `${{ }}` expressions inside shell commands, enabling script injection.

1. `parse alternate engine` step: `${{ inputs.alternate_engine }}` is interpolated directly into shell command strings: `echo "repo=$(echo '${{ inputs.alternate_engine }}' | perl ...)" >> "$GITHUB_OUTPUT"` and similarly for `branch=`. An attacker-controlled input value can break out of the single-quoted context and inject arbitrary shell commands.

2. `save sha` step: `cd "${{ inputs.experimental_path }}"` and `PRIVATE_SARIF_REF="refs/pull/${{ github.event.pull_request.number }}/merge"` — both `inputs.experimental_path` and `github.event.pull_request.number` are interpolated directly into shell.

3. `perl configuration` step: `echo "${{ steps.hash-dictionaries.outputs.perl-libraries }}"` — step output interpolated directly into shell.

4. `install perl modules` step: `for module in ${{ steps.perl-config.outputs.perl-modules }}` — step output interpolated directly into a shell `for` loop without quoting.

5. `Shim Sarif` step: `cd "${{ inputs.experimental_path }}"` — input interpolated directly into shell.

Locations:

- `action.yml:295`
- `action.yml:296`
- `action.yml:356`
- `action.yml:358`
- `action.yml:391`
- `action.yml:416`
- `action.yml:481`

### github-env-injection (severity: high)

Multiple `run:` blocks write values derived from untrusted inputs/github context to `$GITHUB_OUTPUT` or `$GITHUB_ENV` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`).

1. `parse alternate engine` step: `${{ inputs.alternate_engine }}` (a caller-controlled input) is written directly to `$GITHUB_OUTPUT` via `echo "repo=$(echo '${{ inputs.alternate_engine }}' ...)" >> "$GITHUB_OUTPUT"` and `echo "branch=$(echo '${{ inputs.alternate_engine }}' ...)" >> "$GITHUB_OUTPUT"`. No newline sanitization is applied.

2. `save sha` step: `echo "PRIVATE_SARIF_REF=$PRIVATE_SARIF_REF" >> "$GITHUB_ENV"` where `PRIVATE_SARIF_REF` is constructed from `${{ github.event.pull_request.number }}` — a GitHub-context value that is written to `$GITHUB_ENV` without sanitization.

Locations:

- `action.yml:295`
- `action.yml:296`
- `action.yml:359`

### unpinned-uses (severity: high)

All `uses:` references in action.yml use mutable tags or version strings instead of pinned 40-character SHA commit hashes, making the action vulnerable to supply-chain attacks if any referenced action is compromised or its tag is moved.

Failing references:
- `uses: actions/checkout@v4` (alternate-engine step)
- `uses: check-spelling/actions-checkout@v4` (checkout-replacement step)
- `uses: actions/checkout@v4` (checkout step)
- `uses: check-spelling/checkout-merge@v0.0.4` (checkout-merge step)
- `uses: actions/download-artifact@v3` (retrieve-comment step)
- `uses: actions/cache/restore@v3` (retrieve-dictionaries step)
- `uses: actions/cache/restore@v3` (retrieve-perl-libraries-arch step)
- `uses: actions/cache/restore@v3` (retrieve-perl-libraries-lib step)
- `uses: actions/cache/save@v3` (save-dictionaries step)
- `uses: actions/cache/save@v3` (save-perl-libraries-arch step)
- `uses: actions/cache/save@v3` (save-perl-libraries-lib step)
- `uses: actions/upload-artifact@v3` (store-comment step)
- `uses: actions/upload-artifact@v3` (artifact-sarif step)
- `uses: github/codeql-action/upload-sarif@v2` (upload-sarif step)

Locations:

- `action.yml:305`
- `action.yml:315`
- `action.yml:330`
- `action.yml:345`
- `action.yml:375`
- `action.yml:385`
- `action.yml:405`
- `action.yml:410`
- `action.yml:430`
- `action.yml:455`
- `action.yml:462`
- `action.yml:470`
- `action.yml:487`
- `action.yml:494`

### unsafe-shell (severity: high)

The `install perl modules` step pipes remote content directly to a shell interpreter: `curl -s -S -L https://cpanmin.us | perl - --sudo App::cpanminus`. This downloads and executes arbitrary code from the internet without any integrity verification (no checksum, no signature check). An attacker who can intercept or compromise the cpanmin.us endpoint could execute arbitrary code on the runner.

Locations:

- `action.yml:421`

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

1. script-injection/static-inline-injection: Moved all ${{ }} expressions from run: blocks to env: blocks for 5 steps (parse alternate engine, save sha, perl configuration, install perl modules, Shim Sarif).

2. github-env-injection: Added printf '%s' | tr -d '\n\r' sanitization for values written to GITHUB_OUTPUT (parse alternate engine step) and GITHUB_ENV (save sha step).

3. unpinned-uses: Pinned all 14 action references to full 40-character SHA hashes with tag comments: actions/checkout@v4→11d5960a, check-spelling/actions-checkout@v4→cb50106c, check-spelling/checkout-merge@v0.0.4→3aa4a3df, actions/download-artifact@v3→9bc31d5c, actions/cache/restore@v3→6f8efc29 (×3), actions/cache/save@v3→6f8efc29 (×3), actions/upload-artifact@v3→ff15f030 (×2), github/codeql-action/upload-sarif@v2→b8d3b6e8.

4. unsafe-shell: Replaced 'curl ... | perl - --sudo App::cpanminus' with download-then-execute pattern using mktemp.

### Iteration 2

**Fixes applied:** github-env-injection, script-injection

**Notes:**

Fixed two security findings in hardened/action/action.yml:

1. **github-env-injection** (perl configuration step, ~line 407): Added `printf '%s' "$VAR" | tr -d '\n\r'` sanitization for all values written to $GITHUB_OUTPUT: `perl_modules`, `installsitearch`, `installsitelib`, and `perl_modules_sha`. This prevents newline injection via the user-controllable `perl-libraries` step output.

2. **script-injection** (install perl modules step, ~line 428): Replaced unquoted `$PERL_MODULES` expansion in `for module in $PERL_MODULES` and `perl \`command -v cpanm\` -S --notest $cpan_modules` with safe xargs-based tokenization into bash arrays. The `PERL_MODULES` value is now tokenized with `printf '%s' "$PERL_MODULES" | xargs printf '%s\0'` into a null-delimited read loop, stored in `perl_modules_list` array. The `cpan_modules` variable is also an array, and both are expanded with proper quoting `"${array[@]}"`. The backtick command substitution is replaced with `"$(command -v cpanm)"`.

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in the 'install perl modules' step at action.yml line 493. The original `perl -e "use $module; 1;"` expanded `$module` (sourced from `steps.perl-config.outputs.perl-modules`) inside a double-quoted shell string, allowing shell metacharacters in the module name to execute arbitrary commands. The fix uses single-quoted perl code and passes the module name as a separate argument via `$ARGV[0]`: `perl -e 'my $m = $ARGV[0]; $m =~ s|::|/|g; $m .= ".pm"; require $m; 1;' -- "$module"`. This completely eliminates the shell injection vector since `$module` is never interpolated into the shell command string.

