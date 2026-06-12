<!-- markdownlint-disable -->

# Hardening Report: check-spelling--check-spelling/v0.0.22

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **check-spelling--check-spelling/v0.0.22** was hardened automatically. 8 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple `run:` blocks in action.yml directly interpolate `${{ ... }}` expressions into shell commands (sub-rule a), allowing script injection.

1. `parse alternate engine` step: `echo "repo=$(echo '${{ inputs.alternate_engine }}' | perl ...)" >> "$GITHUB_OUTPUT"` — `inputs.alternate_engine` is interpolated directly into the shell command.
2. `save sha` step: `cd "${{ inputs.experimental_path }}"` and `PRIVATE_SARIF_REF="refs/pull/${{ github.event.pull_request.number }}/merge"` — both expressions are interpolated directly into shell.
3. `perl configuration` step: `echo "${{ steps.hash-dictionaries.outputs.perl-libraries }}"` — step output interpolated directly into shell.
4. `install perl modules` step: `for module in ${{ steps.perl-config.outputs.perl-modules }}` — step output interpolated directly into a for-loop without quoting.
5. `Shim Sarif` step: `cd "${{ inputs.experimental_path }}"` — input interpolated directly into shell.

Locations:

- `action.yml:313`
- `action.yml:390`
- `action.yml:440`
- `action.yml:470`
- `action.yml:540`

### github-env-injection (severity: high)

Multiple `run:` blocks write untrusted input values to GitHub special environment files without sanitization.

1. `parse alternate engine` step: `${{ inputs.alternate_engine }}` (caller-controlled) is interpolated directly into the shell and its processed output is written to `$GITHUB_OUTPUT` without `printf '%s' ... | tr -d '\n\r'` sanitization.
2. `save sha` step: A shell variable derived from `${{ github.event.pull_request.number }}` is written to `$GITHUB_ENV` (`echo "PRIVATE_SARIF_REF=$PRIVATE_SARIF_REF" >> "$GITHUB_ENV"`) without sanitization. Similarly `${{ inputs.experimental_path }}` is used in a `cd` command in the same step.

Locations:

- `action.yml:313`
- `action.yml:390`

### unpinned-uses (severity: high)

All `uses:` references in action.yml use mutable tag or version refs instead of pinned 40-character SHA digests, making the action vulnerable to supply-chain attacks if any referenced action is compromised or a tag is moved.

Failing references:
- `uses: actions/checkout@v4` (appears twice)
- `uses: check-spelling/actions-checkout@v4`
- `uses: check-spelling/checkout-merge@v0.0.4`
- `uses: actions/download-artifact@v3`
- `uses: actions/cache/restore@v3` (appears twice)
- `uses: actions/cache/save@v3` (appears three times)
- `uses: actions/upload-artifact@v3` (appears twice)
- `uses: github/codeql-action/upload-sarif@v2`

Locations:

- `action.yml:313`
- `action.yml:350`
- `action.yml:360`
- `action.yml:380`
- `action.yml:400`
- `action.yml:430`
- `action.yml:435`
- `action.yml:460`
- `action.yml:500`
- `action.yml:510`
- `action.yml:520`
- `action.yml:530`
- `action.yml:555`
- `action.yml:570`

### unsafe-shell (severity: high)

The `install perl modules` step pipes remote content directly to a Perl interpreter: `curl -s -S -L https://cpanmin.us | perl - --sudo App::cpanminus`. This downloads and executes arbitrary remote code without any integrity verification (no checksum, no pinned version), which is a supply-chain attack vector.

Locations:

- `action.yml:475`

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

Fixed all security findings in action.yml:

1. **script-injection / static-inline-injection**: Moved all ${{ }} expressions from run: shell blocks into env: maps for 5 steps: 'parse alternate engine' (inputs.alternate_engine), 'save sha' (inputs.experimental_path, github.event.pull_request.number), 'perl configuration' (steps.hash-dictionaries.outputs.perl-libraries), 'install perl modules' (steps.perl-config.outputs.perl-modules), 'Shim Sarif' (inputs.experimental_path).

2. **github-env-injection**: In 'parse alternate engine', used printf '%s' | tr -d '\n\r' to sanitize values before writing to $GITHUB_OUTPUT. In 'save sha', sanitized EXPERIMENTAL_PATH and PR_NUMBER before using in cd and writing PRIVATE_SARIF_REF to $GITHUB_ENV.

3. **unpinned-uses**: Pinned all 14 action references to full SHA digests:
   - actions/checkout@v4 → @34e114876b0b11c390a56381ad16ebd13914f8d5 (×2)
   - check-spelling/actions-checkout@v4 → @cb50106c205b30e86652e685b759a4fd92e25fb5
   - check-spelling/checkout-merge@v0.0.4 → @3aa4a3dfa942930fc6d88a77e55f6e0001445cdc
   - actions/download-artifact@v3 → @9bc31d5ccc31df68ecc42ccf4149144866c47d8a
   - actions/cache/restore@v3 → @6f8efc29b200d32929f49075959781ed54ec270c (×3)
   - actions/cache/save@v3 → @6f8efc29b200d32929f49075959781ed54ec270c (×3)
   - actions/upload-artifact@v3 → @ff15f0306b3f739f7b6fd43fb5d26cd321bd4de5 (×2)
   - github/codeql-action/upload-sarif@v2 → @b8d3b6e8af63cde30bdc382c0bc28114f4346c88

4. **unsafe-shell**: Replaced `curl -s -S -L https://cpanmin.us | perl - --sudo App::cpanminus` with download-to-file-then-execute pattern: curl downloads to /tmp/cpanm-installer, then perl executes the file, then the file is removed.

### Iteration 2

**Fixes applied:** github-env-injection, script-injection

**Notes:**

Fixed two high-severity findings in action.yml:

1. github-env-injection (line 440, 'perl configuration' step): Added sanitization using `printf '%s' "$var" | tr -d '\n\r'` for all four values written to $GITHUB_OUTPUT (perl_modules, installsitearch, installsitelib, perl_modules_sha). The safe_ prefixed variables are now used in the echo statements.

2. script-injection (line 460, 'install perl modules' step): Replaced the unquoted `for module in $PERL_MODULES` word-splitting loop with a safe `while IFS= read -r module` loop reading from a process substitution. Changed cpan_modules from an unquoted string to a bash array, and used properly quoted array expansion `"${cpan_modules[@]}"` when passing modules to cpanm. Also replaced backtick command substitution with `"$(command -v cpanm)"` for safety.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed the script injection vulnerability in the 'install perl modules' step at action.yml line 509. Replaced `perl -e "use $module; 1;"` with `perl -M"$module" -e 1`. The `-M` flag passes the module name as a separate shell argument to perl rather than embedding it inside a double-quoted shell string, preventing shell metacharacters in `$module` from being interpreted by the shell and enabling command injection.

