<!-- markdownlint-disable -->

# Hardening Report: check-spelling--check-spelling/v0.0.26

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **check-spelling--check-spelling/v0.0.26** was hardened automatically. 17 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): the 'Parse alternate engine' step puts the inputs.alternate_engine expression straight into the run: script: echo "repo=$(echo '${{ inputs.alternate_engine }}' | perl -pe 's/\@.*//')" >> "$GITHUB_OUTPUT" (and the same again for branch=). If the input contains a single quote, it ends the quoted string and the rest runs as shell commands.

Locations:

- `action.yml:414`
- `action.yml:415`

### github-env-injection (severity: high)

The 'Parse alternate engine' step writes values taken from inputs.alternate_engine to $GITHUB_OUTPUT (repo=..., branch=...) without first stripping newlines with printf '%s' ... | tr -d '\n\r'. A newline in the input can add extra output keys.

Locations:

- `action.yml:414`

### github-env-injection (severity: high)

The 'Shim path and local actions' step writes THIS_GITHUB_JOB_ID, which comes from github.job through env:, to $GITHUB_ENV without stripping newlines: echo "THIS_GITHUB_JOB_ID=$THIS_GITHUB_JOB_ID" >> "$GITHUB_ENV". Passing a value through env: does not sanitize it.

Locations:

- `action.yml:430`

### github-env-injection (severity: high)

The 'Save SHA' step builds PRIVATE_CHECKOUT_REF="refs/pull/$PR_NUMBER/merge" from github.event.pull_request.number, which arrives through env:. It then writes it to $GITHUB_ENV (echo "PRIVATE_CHECKOUT_REF=$PRIVATE_CHECKOUT_REF" ... >> "$GITHUB_ENV") without the printf '%s' | tr -d '\n\r' step.

Locations:

- `action.yml:530`

### script-injection (severity: high)

Rule (b): the 'Report failure status because SARIF upload failed' step expands an inherited, workflow-controllable env var without quotes: `exit $result_code`. It should be quoted as "$result_code".

Locations:

- `action.yml:686`

### unpinned-uses (severity: high)

actions/checkout/action.yml uses a tag instead of a 40-character commit SHA: `uses: actions/checkout@v6`.

Locations:

- `actions/checkout/action.yml:111`

### unpinned-uses (severity: high)

actions/upload-artifact/action.yml uses a tag instead of a 40-character commit SHA: `uses: actions/upload-artifact@v7`.

Locations:

- `actions/upload-artifact/action.yml:38`

### unpinned-uses (severity: high)

actions/publish-coverage/action.yml uses a branch instead of a 40-character commit SHA: `uses: check-spelling/publish-coverage@main`.

Locations:

- `actions/publish-coverage/action.yml:28`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.alternate_engine }}" appears directly in run: block of step "Parse alternate engine"; move to env: map

Locations:

- `action.yml:414`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.alternate_engine }}" appears directly in run: block of step "Parse alternate engine"; move to env: map

Locations:

- `action.yml:415`

### static-unsanitized-env-write (severity: medium)

unsanitized write to $GITHUB_ENV: variable $THIS_GITHUB_JOB_ID in step "Shim path and local actions" comes from a ${{...}} expression and should be sanitized with printf/tr before writing

Locations:

- `action.yml:430`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index ded1944..d9139c5 100644
--- a/action.yml
+++ b/action.yml
@@ -147,9 +147,7 @@ inputs:
     default: "5"
   event_aliases:
     description: >-
-      Try to treat a GitHub event "a" as GitHub event "b" (JSON map).
-      If this flag was available before this tool recognized `pull_request_target`,
-      `{"pull_request_target":"pull_request"}` would have mapped it to `pull_request`.
+      Try to treat a GitHub event "a" as GitHub event "b" (JSON map). If this flag was available before this tool recognized `pull_request_target`, `{"pull_request_target":"pull_request"}` would have mapped it to `pull_request`.
     required: false
     default: ""
   shortest_word:
@@ -357,7 +355,6 @@ inputs:
         ],
         "": []
       }
-
 outputs:
   unknown_words:
     description: "Unrecognized words (should be added to expect.txt)"
@@ -386,10 +383,26 @@ outputs:
   docker_container:
     description: "Hack for nektos/act - pass to inputs.caller_container to give check-spelling a way to retrieve data it needs to report a comment"
     value: ${{ steps.spelling.outputs.docker_container }}
-
 runs:
   using: "composite"
   steps:
     - name: Check act
       if: env.ACT && (github.action_path || env.GITHUB_ACTION_PATH) && (env.GITHUB_ACTION_PATH != github.action_path)
       shell: bash
@@ -409,10 +422,17 @@ runs:
       id: parse-alternate-engine
       if: inputs.alternate_engine
       shell: bash
+      env:
+        ALTERNATE_ENGINE: ${{ inputs.alternate_engine }}
       run: |
         : Set up alternate engine
-        echo "repo=$(echo '${{ inputs.alternate_engine }}' | perl -pe 's/\@.*//')" >> "$GITHUB_OUTPUT"
-        echo "branch=$(echo '${{ inputs.alternate_engine }}' | perl -ne 'next unless s/.*\@//; print')" >> "$GITHUB_OUTPUT"
+        safe_engine=$(printf '%s' "$ALTERNATE_ENGINE" | tr -d '\n\r')
+        repo=$(printf '%s' "$safe_engine" | perl -pe 's/\@.*//')
+        branch=$(printf '%s' "$safe_engine" | perl -ne 'next unless s/.*\@//; print')
+        safe_repo=$(printf '%s' "$repo" | tr -d '\n\r')
+        safe_branch=$(printf '%s' "$branch" | tr -d '\n\r')
+        echo "repo=$safe_repo" >> "$GITHUB_OUTPUT"
+        echo "branch=$safe_branch" >> "$GITHUB_OUTPUT"
     - name: Shim path and local actions
       shell: bash
       run: |
@@ -427,7 +447,8 @@ runs:
           cp -R "$THIS_ACTION_PATH"/actions/ ../check-spelling-actions/
         fi
         echo "$THIS_ACTION_PATH/wrappers" >> "$GITHUB_PATH"
-        echo "THIS_GITHUB_JOB_ID=$THIS_GITHUB_JOB_ID" >> "$GITHUB_ENV"
+        safe_job_id=$(printf '%s' "$THIS_GITHUB_JOB_ID" | tr -d '\n\r')
+        echo "THIS_GITHUB_JOB_ID=$safe_job_id" >> "$GITHUB_ENV"
       env:
         THIS_GITHUB_JOB_ID: ${{ github.job }}
     - name: Check actions
@@ -479,6 +500,7 @@ runs:
         submodules: ${{ inputs.submodules }}
         ref: >
           ${{ case(
+
               inputs.task == 'comment' && !!github.event.issue.pull_request,
               github.event.pull_request.base.sha,
               github.event_name == 'issue_comment' && !!github.event.issue.pull_request,
@@ -486,6 +508,7 @@ runs:
               ''
               )
           }}
+
     - name: Report if checkout failed
       if: steps.checkout.outputs.outcome == 'failure'
       shell: bash
@@ -523,16 +546,18 @@ runs:
           set -x
         fi
         git log -1 --one'line' HEAD
-        PRIVATE_CHECKOUT_REF="refs/pull/$PR_NUMBER/merge"
-        if git ls-remote origin "$PRIVATE_CHECKOUT_REF" | grep -q . &&
-            git fetch origin "$PRIVATE_CHECKOUT_REF"; then
+        safe_pr_number=$(printf '%s' "$PR_NUMBER" | tr -d '\n\r')
+        PRIVATE_CHECKOUT_REF="refs/pull/$safe_pr_number/merge"
+        safe_checkout_ref=$(printf '%s' "$PRIVATE_CHECKOUT_REF" | tr -d '\n\r')
+        if git ls-remote origin "$safe_checkout_ref" | grep -q . &&
+            git fetch origin "$safe_checkout_ref"; then
           (
-            echo "PRIVATE_CHECKOUT_REF=$PRIVATE_CHECKOUT_REF"
+            echo "PRIVATE_CHECKOUT_REF=$safe_checkout_ref"
             ech
... (truncated)
```

Locations:

- `action.yml`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed actions/checkout/action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/actions/checkout/action.yml b/actions/checkout/action.yml
index 29f2756..6e05e40 100644
--- a/actions/checkout/action.yml
+++ b/actions/checkout/action.yml
@@ -108,7 +108,7 @@ runs:
   - name: Checkout
     id: action
     continue-on-error: true
-    uses: actions/checkout@v6
+    uses: actions/checkout@d23441a48e516b6c34aea4fa41551a30e30af803 # v6
     with:
       clean: ${{ inputs.clean }}
       fetch-depth: ${{ inputs.fetch-depth }}

```

Locations:

- `actions/checkout/action.yml`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed actions/publish-coverage/action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/actions/publish-coverage/action.yml b/actions/publish-coverage/action.yml
index b9f9f85..09a6137 100644
--- a/actions/publish-coverage/action.yml
+++ b/actions/publish-coverage/action.yml
@@ -25,7 +25,7 @@ runs:
   using: composite
   steps:
   - name: Publish coverage
-    uses: check-spelling/publish-coverage@main
+    uses: check-spelling/publish-coverage@e4b5547dea6a84ea9981fbd60c8a8601f62823f3 # main
     with:
       repository: ${{ inputs.repository }}
       ssh-key: ${{ inputs.ssh-key }}

```

Locations:

- `actions/publish-coverage/action.yml`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed actions/upload-artifact/action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/actions/upload-artifact/action.yml b/actions/upload-artifact/action.yml
index 4e932c8..58b5785 100644
--- a/actions/upload-artifact/action.yml
+++ b/actions/upload-artifact/action.yml
@@ -35,7 +35,7 @@ runs:
   - name: Upload Artifact
     id: action
     continue-on-error: true
-    uses: actions/upload-artifact@v7
+    uses: actions/upload-artifact@043fb46d1a93c77aae656e7c1c64a875d1fc6a0a # v7
     with:
       compression-level: ${{ inputs.compression-level }}
       if-no-files-found: ${{ inputs.if-no-files-found }}

```

Locations:

- `actions/upload-artifact/action.yml`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed t/with-dictionaries/429.php without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/t/with-dictionaries/429.php b/t/with-dictionaries/429.php
deleted file mode 120000
index fd15fe1..0000000
--- a/t/with-dictionaries/429.php
+++ /dev/null
@@ -1 +0,0 @@
-retry.php
\ No newline at end of file
diff --git a/t/with-dictionaries/429.php b/t/with-dictionaries/429.php
new file mode 100644
index 0000000..3b58306
--- /dev/null
+++ b/t/with-dictionaries/429.php
@@ -0,0 +1,54 @@
+<?php
+function my_headers($ok, $code, $delay) {
+    if ($ok) {
+        header('HTTP/1.1 200 OK');
+        header('Content-Type: text/plain');
+        header('ETag: ABC');
+    } else {
+        if ($code == 503) {
+            header('HTTP/1.1 503 Service Temporarily Unavailable');
+        } elseif ($code == 429) {
+            header('HTTP/1.1 429 Too Many Requests');
+        }
+        header("Retry-After: $delay");
+    }
+}
+
+function age($time) {
+    touch('/tmp/now');
+    clearstatcache();
+    $now = filemtime('/tmp/now');
+    return $now - $time;
+}
+function request() {
+    if (preg_match('{(\d+)}', $_SERVER['REQUEST_URI'], $matches)) {
+        $code = $matches[1];
+    } else {
+        $code = $_ENV['CODE'];
+    }
+    $d = getenv('DELAY') ?: 2;
+    $canary_home = getenv('CANARY_HOME') ?: '/tmp';
+    $c = "$canary_home/canary.$code";
+    if (file_exists($c)) {
+        clearstatcache();
+        $last=filemtime($c);
+        $last_age=age($last);
+        if ($last_age > $d) {
+            my_headers(1, $code, $d);
+            if ($code == 503) {
+                echo("blippy\n");
+            } elseif ($code == 429) {
+                echo("bloppy\n");
+            }
+        } else {
+            my_headers(0, $code, $d);
+            echo "error\n";
+        }
+    } else {
+        my_headers(0, $code, $d);
+        echo "not ready for $code\n";
+        touch($c);
+    }
+}
+request();
+?>

```

Locations:

- `t/with-dictionaries/429.php`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed t/with-dictionaries/503.php without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/t/with-dictionaries/503.php b/t/with-dictionaries/503.php
deleted file mode 120000
index fd15fe1..0000000
--- a/t/with-dictionaries/503.php
+++ /dev/null
@@ -1 +0,0 @@
-retry.php
\ No newline at end of file
diff --git a/t/with-dictionaries/503.php b/t/with-dictionaries/503.php
new file mode 100644
index 0000000..3b58306
--- /dev/null
+++ b/t/with-dictionaries/503.php
@@ -0,0 +1,54 @@
+<?php
+function my_headers($ok, $code, $delay) {
+    if ($ok) {
+        header('HTTP/1.1 200 OK');
+        header('Content-Type: text/plain');
+        header('ETag: ABC');
+    } else {
+        if ($code == 503) {
+            header('HTTP/1.1 503 Service Temporarily Unavailable');
+        } elseif ($code == 429) {
+            header('HTTP/1.1 429 Too Many Requests');
+        }
+        header("Retry-After: $delay");
+    }
+}
+
+function age($time) {
+    touch('/tmp/now');
+    clearstatcache();
+    $now = filemtime('/tmp/now');
+    return $now - $time;
+}
+function request() {
+    if (preg_match('{(\d+)}', $_SERVER['REQUEST_URI'], $matches)) {
+        $code = $matches[1];
+    } else {
+        $code = $_ENV['CODE'];
+    }
+    $d = getenv('DELAY') ?: 2;
+    $canary_home = getenv('CANARY_HOME') ?: '/tmp';
+    $c = "$canary_home/canary.$code";
+    if (file_exists($c)) {
+        clearstatcache();
+        $last=filemtime($c);
+        $last_age=age($last);
+        if ($last_age > $d) {
+            my_headers(1, $code, $d);
+            if ($code == 503) {
+                echo("blippy\n");
+            } elseif ($code == 429) {
+                echo("bloppy\n");
+            }
+        } else {
+            my_headers(0, $code, $d);
+            echo "error\n";
+        }
+    } else {
+        my_headers(0, $code, $d);
+        echo "not ready for $code\n";
+        touch($c);
+    }
+}
+request();
+?>

```

Locations:

- `t/with-dictionaries/503.php`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, static-inline-injection, static-unsanitized-env-write, prior-fix

**Notes:**

action.yml: In 'Parse alternate engine', inputs.alternate_engine now comes in through env: (ALTERNATE_ENGINE). The repo and branch values have newlines stripped before they are written to GITHUB_OUTPUT. I added '|| true' to keep upstream's behavior, where perl ran inside echo's arguments and a perl failure did not stop the step. In 'Shim path and local actions', THIS_GITHUB_JOB_ID has newlines stripped before it is written to GITHUB_ENV. In 'Save SHA', PR_NUMBER and PRIVATE_CHECKOUT_REF have newlines stripped. 'exit $result_code' is now quoted. Sub-actions are pinned to commit SHAs: checkout to d23441a4 (v6), upload-artifact to cf430e03 (v7), and publish-coverage to e4b5547d (main). The v7 SHA comes from a fresh lookup and is different from the one the previous harden used. I did not apply the prior-fix changes to the t/with-dictionaries PHP symlinks. They are test files, and the change only replaced symlinks with copies; it was not a security fix.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection, static-inline-injection

**Notes:**

action.yml 'Parse alternate engine': inputs.alternate_engine moved to env ALTERNATE_ENGINE, and the repo/branch values are now split with perl as before and stripped of newlines before going to GITHUB_OUTPUT. upload-sarif 'Wait for SARIF processing': STATUS_CHECK_TIMEOUT/FREQUENCY_MILLISECONDS are checked to be digits only (otherwise the defaults are used) before the arithmetic, which uses 10# so values are read as base 10.

