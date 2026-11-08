<!-- markdownlint-disable -->

# Hardening Report: check-spelling--check-spelling/v0.0.22

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **check-spelling--check-spelling/v0.0.22** was hardened automatically. 15 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): in step 'parse alternate engine', inputs.alternate_engine is inserted straight into the run script: echo "repo=$(echo '${{ inputs.alternate_engine }}' | perl -pe 's/\@.*//')" and the matching branch= line. A single quote in the input breaks out of the string and runs commands.

Locations:

- `action.yml:353`
- `action.yml:354`

### script-injection (severity: high)

Rule (a): step 'save sha' inserts expressions into the run script: cd "${{ inputs.experimental_path }}" and PRIVATE_SARIF_REF="refs/pull/${{ github.event.pull_request.number }}/merge".

Locations:

- `action.yml:433`
- `action.yml:435`

### script-injection (severity: high)

Rule (a): step 'perl configuration' inserts a step output into the run script: echo "${{ steps.hash-dictionaries.outputs.perl-libraries }}" | tr ...

Locations:

- `action.yml:476`

### script-injection (severity: high)

Rule (a): step 'install perl modules' inserts a step output into the run script: for module in ${{ steps.perl-config.outputs.perl-modules }}; do. Rule (b): it also expands $module and $cpan_modules unquoted (echo $module; cpanm ... $cpan_modules), and both come from that output.

Locations:

- `action.yml:507`
- `action.yml:513`

### script-injection (severity: high)

Rule (a): step 'Shim Sarif' inserts an input into the run script: cd "${{ inputs.experimental_path }}".

Locations:

- `action.yml:564`

### github-env-injection (severity: high)

Step 'parse alternate engine' writes values taken from inputs.alternate_engine to $GITHUB_OUTPUT (repo=..., branch=...) without the printf '%s' | tr -d '\n\r' step, so a value containing newlines can add extra outputs.

Locations:

- `action.yml:353`
- `action.yml:354`

### github-env-injection (severity: high)

Step 'shim-path' writes values built from inherited process env vars ($GITHUB_ACTION_PATH via realpath, and $PATH) to $GITHUB_ENV without stripping newlines: echo "THIS_ACTION_PATH=$THIS_ACTION_PATH" >> "$GITHUB_ENV"; echo "PATH=$THIS_ACTION_PATH/wrappers:$PATH" >> "$GITHUB_ENV".

Locations:

- `action.yml:374`
- `action.yml:375`

### github-env-injection (severity: high)

Step 'save sha' writes PRIVATE_SARIF_REF, which is built from ${{ github.event.pull_request.number }}, to $GITHUB_ENV without stripping newlines.

Locations:

- `action.yml:436`

### github-env-injection (severity: high)

Step 'perl configuration' writes perl-modules=$perl_modules to $GITHUB_OUTPUT. The value comes from steps.hash-dictionaries.outputs.perl-libraries and is not passed through printf '%s' | tr -d '\n\r'.

Locations:

- `action.yml:484`

### unsafe-shell (severity: high)

Step 'install perl modules' pipes a remote script straight into an interpreter: curl -s -S -L https://cpanmin.us | perl - --sudo App::cpanminus

Locations:

- `action.yml:512`

### unpinned-uses (severity: high)

These uses: references in action.yml point to a tag, not a commit SHA: actions/checkout@v4 (lines 357, 403), check-spelling/actions-checkout@v4 (388), check-spelling/checkout-merge@v0.0.4 (419), actions/download-artifact@v3 (447), actions/cache/restore@v3 (465, 490, 497), actions/cache/save@v3 (528, 536, 543), actions/upload-artifact@v3 (553, 569), github/codeql-action/upload-sarif@v2 (578).

Locations:

- `action.yml:357`
- `action.yml:388`
- `action.yml:403`
- `action.yml:419`
- `action.yml:447`
- `action.yml:465`
- `action.yml:490`
- `action.yml:497`
- `action.yml:528`
- `action.yml:536`
- `action.yml:543`
- `action.yml:553`
- `action.yml:569`
- `action.yml:578`

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

**Fixes applied:** script-injection, static-inline-injection, github-env-injection, unsafe-shell, unpinned-uses

**Notes:**

All fixes are in action.yml. Nothing was run, so none of the changes have been tested at runtime.

- **Pinned actions:** every uses: reference now points to a full commit SHA, with the tag kept as a comment. This covers checkout v4, check-spelling/actions-checkout v4, checkout-merge v0.0.4, download-artifact v3, cache restore/save v3, upload-artifact v3 and codeql upload-sarif v2.
- **Expressions moved out of run: scripts:** in the steps 'parse alternate engine', 'save sha', 'Shim Sarif', 'perl configuration' and 'install perl modules', each ${{ }} value now comes in through env: and is used as a quoted shell variable.
- **Newlines stripped before writes:** the values written to $GITHUB_OUTPUT and $GITHUB_ENV (repo, branch, THIS_ACTION_PATH, PATH, PRIVATE_SARIF_REF, perl-modules) now pass through printf '%s' | tr -d '\n\r' first.
- **Perl modules step:** the module list is split on whitespace into a bash array (read -r -d '' -a), which matches upstream's unquoted for-loop. Missing modules are collected in an array and passed to cpanm quoted.
- **cpanminus installer:** it is now downloaded from https://cpanmin.us to a temp file and run with 'perl "$file" --sudo App::cpanminus', instead of being piped from curl into perl.

The module name is still inserted into perl -e "use $module; 1;", as upstream had it. No finding covered that line. I tried adding a name check there but took it out, because I couldn't confirm what format unknown-words.sh produces for these names (the file is too large to read), and a strict check could reject valid names.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

In action.yml act-windows-shim step, quoted $THIS_ACTION_PATH in chmod and perl commands ("$THIS_ACTION_PATH"/*.sh etc.), keeping globs unquoted so expansion behavior is unchanged.

