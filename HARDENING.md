<!-- markdownlint-disable -->

# Hardening Report: check-spelling--check-spelling/v0.0.22

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **check-spelling--check-spelling/v0.0.22** was hardened automatically. 14 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'parse alternate engine' run: block directly interpolates ${{ inputs.alternate_engine }} into shell commands. This allows an attacker-controlled input to inject arbitrary shell commands. Offending lines: `echo "repo=$(echo '${{ inputs.alternate_engine }}' | perl -pe 's/\@.*//')" >> "$GITHUB_OUTPUT"` and `echo "branch=$(echo '${{ inputs.alternate_engine }}' | perl -ne 'next unless s/.*\@//; print')" >> "$GITHUB_OUTPUT"`

Locations:

- `action.yml:295`

### script-injection (severity: high)

Sub-rule (a): The 'save sha' run: block directly interpolates ${{ inputs.experimental_path }} and ${{ github.event.pull_request.number }} into shell commands. Offending lines: `cd "${{ inputs.experimental_path }}"` and `PRIVATE_SARIF_REF="refs/pull/${{ github.event.pull_request.number }}/merge"`

Locations:

- `action.yml:375`

### script-injection (severity: high)

Sub-rule (a): The 'perl configuration' run: block directly interpolates ${{ steps.hash-dictionaries.outputs.perl-libraries }} into a shell command substitution: `echo "${{ steps.hash-dictionaries.outputs.perl-libraries }}" |tr " " "\n" |sort|xargs`. A step output can be attacker-controlled via PR content.

Locations:

- `action.yml:417`

### script-injection (severity: high)

Sub-rule (a): The 'install perl modules' run: block directly interpolates ${{ steps.perl-config.outputs.perl-modules }} into a shell for-loop: `for module in ${{ steps.perl-config.outputs.perl-modules }}; do`. This allows injection of arbitrary shell tokens.

Locations:

- `action.yml:454`

### script-injection (severity: high)

Sub-rule (a): The 'Shim Sarif' run: block directly interpolates ${{ inputs.experimental_path }} into a shell command: `cd "${{ inputs.experimental_path }}"`. An attacker-controlled input can inject shell metacharacters.

Locations:

- `action.yml:514`

### github-env-injection (severity: high)

The 'parse alternate engine' run: block writes ${{ inputs.alternate_engine }} (an attacker-controlled input) directly to $GITHUB_OUTPUT without sanitization (no `printf '%s' ... | tr -d '\n\r'` step). This allows newline injection to set arbitrary output variables.

Locations:

- `action.yml:295`

### github-env-injection (severity: high)

The 'save sha' run: block writes ${{ github.event.pull_request.number }} (attacker-influenced via PR) directly to $GITHUB_ENV via `echo "PRIVATE_SARIF_REF=$PRIVATE_SARIF_REF" >> "$GITHUB_ENV"` without sanitization. This allows newline injection to set arbitrary environment variables.

Locations:

- `action.yml:377`

### github-env-injection (severity: high)

The 'perl configuration' run: block writes ${{ steps.hash-dictionaries.outputs.perl-libraries }} (a step output that can be attacker-influenced) directly to $GITHUB_OUTPUT via `echo "perl-modules=$perl_modules"` without sanitization (no `tr -d '\n\r'`).

Locations:

- `action.yml:417`

### unpinned-uses (severity: high)

Multiple uses: references in action.yml use mutable tags/versions instead of pinned 40-character SHA digests, making the action vulnerable to supply-chain attacks if any referenced action is compromised or its tag is moved. Failing references: actions/checkout@v4, check-spelling/actions-checkout@v4, check-spelling/checkout-merge@v0.0.4, actions/download-artifact@v3, actions/cache/restore@v3 (×2), actions/cache/save@v3 (×3), actions/upload-artifact@v3 (×2), github/codeql-action/upload-sarif@v2.

Locations:

- `action.yml:300`
- `action.yml:310`
- `action.yml:330`
- `action.yml:395`
- `action.yml:405`
- `action.yml:430`
- `action.yml:435`
- `action.yml:470`
- `action.yml:480`
- `action.yml:490`
- `action.yml:500`
- `action.yml:530`

### unsafe-shell (severity: high)

The 'install perl modules' run: block pipes remote content directly to a Perl interpreter: `curl -s -S -L https://cpanmin.us | perl - --sudo App::cpanminus`. If the remote server is compromised or the connection is intercepted, arbitrary code will execute on the runner with sudo privileges.

Locations:

- `action.yml:459`

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

1. script-injection/static-inline-injection: Moved all ${{ }} expressions from run: shell blocks to env: blocks - inputs.alternate_engine, inputs.experimental_path, github.event.pull_request.number, steps.hash-dictionaries.outputs.perl-libraries, steps.perl-config.outputs.perl-modules.

2. github-env-injection: Added printf '%s' | tr -d '\n\r' sanitization before writing attacker-influenced values to $GITHUB_OUTPUT and $GITHUB_ENV in the 'parse alternate engine', 'save sha', and 'perl configuration' steps.

3. unpinned-uses: Pinned all 12 action references to full 40-character commit SHAs: actions/checkout@v4 (×2), check-spelling/actions-checkout@v4, check-spelling/checkout-merge@v0.0.4, actions/download-artifact@v3, actions/cache/restore@v3 (×3), actions/cache/save@v3 (×3), actions/upload-artifact@v3 (×2), github/codeql-action/upload-sarif@v2.

4. unsafe-shell: Fixed 'curl | perl' pipe in 'install perl modules' step by downloading cpanmin.us to a temp file first, then executing it separately with perl, then removing the temp file.

5. install perl modules list handling: Used xargs-based tokenization with null-delimited read loop to safely handle the space-separated perl-modules list without collapsing it into a single argument.

