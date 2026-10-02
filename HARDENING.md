<!-- markdownlint-disable -->

# Hardening Report: check-spelling--check-spelling/v0.0.22

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **check-spelling--check-spelling/v0.0.22** was hardened automatically. 8 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks in action.yml directly interpolate ${{ ... }} expressions into shell commands (rule a), enabling script injection. (1) 'parse alternate engine' step: echo "repo=$(echo '${{ inputs.alternate_engine }}' | perl ...)" >> "$GITHUB_OUTPUT" — inputs.alternate_engine is attacker-controlled and interpolated directly into the shell. (2) 'save sha' step: cd "${{ inputs.experimental_path }}" and PRIVATE_SARIF_REF="refs/pull/${{ github.event.pull_request.number }}/merge" — both inputs.experimental_path and github.event.pull_request.number are interpolated directly. (3) 'perl configuration' step: echo "${{ steps.hash-dictionaries.outputs.perl-libraries }}" — step output interpolated directly into shell. (4) 'install perl modules' step: for module in ${{ steps.perl-config.outputs.perl-modules }}; do — step output interpolated directly into a for-loop, allowing word-splitting and shell metacharacter injection.

Locations:

- `action.yml:248`
- `action.yml:249`
- `action.yml:296`
- `action.yml:298`
- `action.yml:340`
- `action.yml:360`

### github-env-injection (severity: high)

Unsanitized untrusted values are written to $GITHUB_OUTPUT and $GITHUB_ENV without the required printf '%s' ... | tr -d '\n\r' sanitization step. (1) 'parse alternate engine' step: echo "repo=$(echo '${{ inputs.alternate_engine }}' | perl ...)" >> "$GITHUB_OUTPUT" and the branch echo — inputs.alternate_engine (attacker-controlled) is written to GITHUB_OUTPUT without newline sanitization. (2) 'save sha' step: echo "PRIVATE_SARIF_REF=$PRIVATE_SARIF_REF" >> "$GITHUB_ENV" where PRIVATE_SARIF_REF is derived from ${{ github.event.pull_request.number }} — written to GITHUB_ENV without sanitization.

Locations:

- `action.yml:248`
- `action.yml:249`
- `action.yml:298`

### unsafe-shell (severity: high)

The 'install perl modules' step pipes remote content directly to a Perl interpreter: `curl -s -S -L https://cpanmin.us | perl - --sudo App::cpanminus`. If the remote URL is compromised or the connection is intercepted, arbitrary code will be executed on the runner with sudo privileges.

Locations:

- `action.yml:365`

### unpinned-uses (severity: high)

All uses: references in action.yml use mutable tags instead of pinned 40-character SHA digests, making the action vulnerable to supply-chain attacks. Failing references: actions/checkout@v4 (x2), check-spelling/actions-checkout@v4, check-spelling/checkout-merge@v0.0.4, actions/download-artifact@v3, actions/cache/restore@v3 (x3), actions/cache/save@v3 (x3), actions/upload-artifact@v3 (x2), github/codeql-action/upload-sarif@v2.

Locations:

- `action.yml:270`
- `action.yml:278`
- `action.yml:285`
- `action.yml:292`
- `action.yml:305`
- `action.yml:316`
- `action.yml:326`
- `action.yml:333`
- `action.yml:346`
- `action.yml:352`
- `action.yml:358`
- `action.yml:375`
- `action.yml:393`
- `action.yml:400`

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

Fixed all findings in action.yml:

1. script-injection/static-inline-injection: Moved all ${{ }} expressions from run: blocks to env: maps. 'parse alternate engine' uses ALTERNATE_ENGINE env var; 'save sha' uses EXPERIMENTAL_PATH and PR_NUMBER; 'perl configuration' uses PERL_LIBRARIES; 'install perl modules' uses PERL_MODULES with xargs-based bash array tokenization; 'Shim Sarif' uses EXPERIMENTAL_PATH.

2. github-env-injection: Added printf '%s' | tr -d '\n\r' sanitization in 'parse alternate engine' before writing to GITHUB_OUTPUT, and in 'save sha' before writing to GITHUB_ENV.

3. unsafe-shell: Replaced 'curl ... | perl - --sudo App::cpanminus' with download-to-tempfile-then-execute pattern using mktemp and curl -o.

4. unpinned-uses: Pinned all 14 action references to full 40-character SHA digests: actions/checkout@v4 (x2) → 11d5960..., check-spelling/actions-checkout@v4 → cb50106..., check-spelling/checkout-merge@v0.0.4 → 3aa4a3d..., actions/download-artifact@v3 → 9bc31d5..., actions/cache/restore@v3 (x3) → 6f8efc2..., actions/cache/save@v3 (x3) → 6f8efc2..., actions/upload-artifact@v3 (x2) → ff15f03..., github/codeql-action/upload-sarif@v2 → b8d3b6e...

