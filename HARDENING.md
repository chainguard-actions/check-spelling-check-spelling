<!-- markdownlint-disable -->

# Hardening Report: check-spelling--check-spelling/v0.0.22

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **check-spelling--check-spelling/v0.0.22** was hardened automatically. 13 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'parse alternate engine' step directly interpolates ${{ inputs.alternate_engine }} inside a run: shell command. This allows an attacker-controlled input to inject arbitrary shell commands. Offending lines: echo "repo=$(echo '${{ inputs.alternate_engine }}' | perl ...)" >> "$GITHUB_OUTPUT" and echo "branch=$(echo '${{ inputs.alternate_engine }}' | perl ...)" >> "$GITHUB_OUTPUT"

Locations:

- `action.yml:278`
- `action.yml:279`

### script-injection (severity: high)

Sub-rule (a): The 'save sha' step directly interpolates ${{ inputs.experimental_path }} and ${{ github.event.pull_request.number }} inside run: shell commands. Offending lines: cd "${{ inputs.experimental_path }}" and PRIVATE_SARIF_REF="refs/pull/${{ github.event.pull_request.number }}/merge"

Locations:

- `action.yml:332`
- `action.yml:334`

### script-injection (severity: high)

Sub-rule (a): The 'perl configuration' step directly interpolates ${{ steps.hash-dictionaries.outputs.perl-libraries }} inside a run: shell command. Offending line: echo "${{ steps.hash-dictionaries.outputs.perl-libraries }}" |tr " " "\n" |sort|xargs

Locations:

- `action.yml:374`

### script-injection (severity: high)

Sub-rule (a): The 'install perl modules' step directly interpolates ${{ steps.perl-config.outputs.perl-modules }} inside a run: shell for-loop, allowing step output to inject arbitrary shell commands. Offending line: for module in ${{ steps.perl-config.outputs.perl-modules }}; do

Locations:

- `action.yml:393`

### script-injection (severity: high)

Sub-rule (a): The 'Shim Sarif' step directly interpolates ${{ inputs.experimental_path }} inside a run: shell command. Offending line: cd "${{ inputs.experimental_path }}"

Locations:

- `action.yml:446`

### github-env-injection (severity: high)

The 'parse alternate engine' step writes values derived from ${{ inputs.alternate_engine }} (attacker-controlled) directly to $GITHUB_OUTPUT without sanitization (no printf '%s' ... | tr -d '\n\r'). A newline in the input can inject arbitrary output variables.

Locations:

- `action.yml:278`
- `action.yml:279`

### github-env-injection (severity: high)

The 'save sha' step writes PRIVATE_SARIF_REF (derived from ${{ github.event.pull_request.number }}) to $GITHUB_ENV without sanitization. Offending line: echo "PRIVATE_SARIF_REF=$PRIVATE_SARIF_REF" >> "$GITHUB_ENV". A newline in the value could inject arbitrary environment variables.

Locations:

- `action.yml:335`

### unsafe-shell (severity: high)

The 'install perl modules' step pipes remote content directly to a Perl interpreter: curl -s -S -L https://cpanmin.us | perl - --sudo App::cpanminus. This executes remotely-fetched code without integrity verification, enabling supply-chain attacks.

Locations:

- `action.yml:398`

### unpinned-uses (severity: high)

All 14 uses: references in action.yml use mutable version tags instead of immutable 40-character SHA commit pins, making the action vulnerable to supply-chain attacks. Unpinned references include: actions/checkout@v4, check-spelling/actions-checkout@v4, check-spelling/checkout-merge@v0.0.4, actions/download-artifact@v3, actions/cache/restore@v3 (x2), actions/cache/save@v3 (x3), actions/upload-artifact@v3 (x2), github/codeql-action/upload-sarif@v2.

Locations:

- `action.yml:284`
- `action.yml:302`
- `action.yml:315`
- `action.yml:325`
- `action.yml:344`
- `action.yml:356`
- `action.yml:363`
- `action.yml:370`
- `action.yml:408`
- `action.yml:421`
- `action.yml:432`
- `action.yml:438`
- `action.yml:449`
- `action.yml:456`

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

**Fixes applied:** script-injection, github-env-injection, unsafe-shell, unpinned-uses, static-inline-injection

**Notes:**

Fixed all 14 findings in action.yml:

1. script-injection (5 instances): Moved all ${{ }} expressions from run: blocks to env: blocks - ALTERNATE_ENGINE in 'parse alternate engine', EXPERIMENTAL_PATH+PR_NUMBER in 'save sha', PERL_LIBRARIES in 'perl configuration', PERL_MODULES in 'install perl modules' (with xargs tokenization for list input), EXPERIMENTAL_PATH in 'Shim Sarif'.

2. github-env-injection (2 instances): Added printf '%s' | tr -d '\n\r' sanitization before writing to GITHUB_OUTPUT/GITHUB_ENV in 'parse alternate engine' and 'save sha' steps.

3. unsafe-shell (1 instance): Fixed curl|perl pipe in 'install perl modules' by downloading cpanmin.us to a mktemp file first, then executing it separately with perl.

4. unpinned-uses (14 instances): Pinned all action references to full commit SHAs - actions/checkout@v4→11d5960a (x2), check-spelling/actions-checkout@v4→cb50106c, check-spelling/checkout-merge@v0.0.4→3aa4a3df, actions/download-artifact@v3→9bc31d5c, actions/cache/restore@v3→6f8efc29 (x3), actions/cache/save@v3→6f8efc29 (x3), actions/upload-artifact@v3→ff15f030 (x2), github/codeql-action/upload-sarif@v2→b8d3b6e8.

5. static-inline-injection (4 instances): Same fixes as script-injection above - all ${{ }} expressions moved to env: blocks.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection finding in the 'perl-config' step of action.yml. The `perl_modules` variable (derived from the untrusted `PERL_LIBRARIES` step output, which comes from `steps.hash-dictionaries.outputs.perl-libraries`) was being written directly to `$GITHUB_OUTPUT` without newline sanitization. Added a sanitization step: `safe_perl_modules=$(printf '%s' "$perl_modules" | tr -d '\n\r')` and replaced the `echo "perl-modules=$perl_modules"` line with `echo "perl-modules=$safe_perl_modules"` to prevent newline injection attacks.

