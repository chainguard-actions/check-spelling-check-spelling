<!-- markdownlint-disable -->

# Hardening Report: check-spelling--check-spelling/v0.0.24

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **check-spelling--check-spelling/v0.0.24** was hardened automatically. 19 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): in the 'parse alternate engine' step, the run block puts the input expression straight into the shell script: echo "repo=$(echo '${{ inputs.alternate_engine }}' | perl -pe 's/\@.*//')" and the matching branch= line. A single quote in the input can break out of the quoting and run any command.

Locations:

- `action.yml:392`

### github-env-injection (severity: high)

The 'parse alternate engine' step writes values taken from inputs.alternate_engine to $GITHUB_OUTPUT (repo=..., branch=...). Newlines are not stripped with printf '%s' | tr -d '\n\r' before the write, so a crafted value can add extra outputs.

Locations:

- `action.yml:392`

### github-env-injection (severity: high)

The 'shim' step runs echo "THIS_GITHUB_JOB_ID=$THIS_GITHUB_JOB_ID" >> "$GITHUB_ENV", where THIS_GITHUB_JOB_ID comes from ${{ github.job }}. Newlines are not stripped with printf '%s' | tr -d '\n\r' before the write.

Locations:

- `action.yml:407`

### script-injection (severity: high)

Rule (a): in the 'save sha' step, the run block puts a GitHub expression straight into the shell script: PRIVATE_SARIF_REF="refs/pull/${{ github.event.pull_request.number }}/merge". It should be passed through an env: variable instead.

Locations:

- `action.yml:489`

### github-env-injection (severity: high)

The 'save sha' step runs echo "PRIVATE_SARIF_REF=$PRIVATE_SARIF_REF" >> "$GITHUB_ENV". PRIVATE_SARIF_REF is built from github.event.pull_request.number, and newlines are not stripped with printf '%s' | tr -d '\n\r' before the write.

Locations:

- `action.yml:490`

### unpinned-uses (severity: high)

Unpinned action reference: 'uses: actions/checkout@v4' points to a tag, not a full commit SHA.

Locations:

- `actions/checkout/action.yml:80`

### unpinned-uses (severity: high)

Unpinned action reference: 'uses: check-spelling/checkout-merge@v0.0.6' points to a tag, not a full commit SHA.

Locations:

- `actions/checkout-merge/action.yml:31`

### unpinned-uses (severity: high)

Unpinned action reference: 'uses: actions/upload-artifact@v4' points to a tag, not a full commit SHA.

Locations:

- `actions/upload-artifact/action.yml:37`

### unpinned-uses (severity: high)

Unpinned action reference: 'uses: github/codeql-action/upload-sarif@v3' points to a tag, not a full commit SHA.

Locations:

- `actions/upload-sarif/action.yml:55`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.alternate_engine }}" appears directly in run: block of step "parse alternate engine"; move to env: map

Locations:

- `action.yml:383`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.alternate_engine }}" appears directly in run: block of step "parse alternate engine"; move to env: map

Locations:

- `action.yml:384`

### static-unsanitized-env-write (severity: medium)

unsanitized write to $GITHUB_ENV: variable $THIS_GITHUB_JOB_ID in step "shim" comes from a ${{...}} expression and should be sanitized with printf/tr before writing

Locations:

- `action.yml:398`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index 764a087..d0e02e8 100644
--- a/action.yml
+++ b/action.yml
@@ -143,9 +143,7 @@ inputs:
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
@@ -326,7 +324,6 @@ inputs:
     description: "Per file spell checking time limit"
     required: false
     default: 30
-
 outputs:
   unknown_words:
     description: "Unrecognized words (should be added to expect.txt)"
@@ -355,10 +352,26 @@ outputs:
   docker_container:
     description: "Hack for nektos/act - pass to inputs.caller_container to give check-spelling a way to retrieve data it needs to report a comment"
     value: ${{ steps.spelling.outputs.docker_container }}
-
 runs:
   using: "composite"
   steps:
     - name: act-broken
       if: env.ACT && (github.action_path || env.GITHUB_ACTION_PATH) && (env.GITHUB_ACTION_PATH != github.action_path)
       shell: bash
@@ -378,10 +391,13 @@ runs:
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
+        echo "repo=$(printf '%s' "$safe_engine" | perl -pe 's/\@.*//')" >> "$GITHUB_OUTPUT"
+        echo "branch=$(printf '%s' "$safe_engine" | perl -ne 'next unless s/.*\@//; print')" >> "$GITHUB_OUTPUT"
     - name: shim
       shell: bash
       run: |
@@ -395,7 +411,8 @@ runs:
           echo "USE_RSYNC_FOR_LOCAL_CHECK_SPELLING_ACTIONS=1" >> "$GITHUB_ENV"
         fi
         echo "$THIS_ACTION_PATH/wrappers" >> "$GITHUB_PATH"
-        echo "THIS_GITHUB_JOB_ID=$THIS_GITHUB_JOB_ID" >> "$GITHUB_ENV"
+        safe_job_id=$(printf '%s' "$THIS_GITHUB_JOB_ID" | tr -d '\n\r')
+        echo "THIS_GITHUB_JOB_ID=$safe_job_id" >> "$GITHUB_ENV"
       env:
         THIS_GITHUB_JOB_ID: ${{ github.job }}
     - name: check-actions
@@ -446,13 +463,7 @@ runs:
         fetch-depth: ${{ !contains(inputs.check_commit_messages, 'commits') && '1' || '0' }}
         filter: ${{ contains(inputs.check_commit_messages, 'commits') && 'tree:0' || '' }}
         token: ${{ inputs.checkout-token || inputs.GITHUB_TOKEN }}
-        ref:
-          ${{
-            github.event_name == 'issue_comment' &&
-            github.event.issue.pull_request &&
-            github.event.pull_request.head.sha ||
-            ''
-          }}
+        ref: ${{ github.event_name == 'issue_comment' && github.event.issue.pull_request && github.event.pull_request.head.sha || '' }}
     - name: report if checkout failed
       if: steps.checkout.outcome == 'failure'
       shell: bash
@@ -474,11 +485,15 @@ runs:
       if: (contains(github.event_name, 'pull_request')) && (inputs.checkout && inputs.checkout != 'false' && !inputs.custom_task && !inputs.task) && inputs.use_sarif && steps.checkout-merge.outputs.message == ''
       shell: bash
       working-directory: ${{ inputs.experimental_path }}
+      env:
+        PR_NUMBER: ${{ github.event.pull_request.number }}
       run: |
         : Save SHA
         git show HEAD
-        PRIVATE_SARIF_REF="refs/pull/${{ github.event.pull_request.number }}/merge"
-        echo "PRIVATE_SARIF_REF=$PRIVATE_SARIF_REF" >> "$GITHUB_ENV"
+        safe_pr_number=$(printf '%s' "$PR_NUMBER" | tr -d '\n\r')
+        PRIVATE_SARIF_R
... (truncated)
```

Locations:

- `action.yml`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed actions/checkout/action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/actions/checkout/action.yml b/actions/checkout/action.yml
index 6a222a2..ccfbaa9 100644
--- a/actions/checkout/action.yml
+++ b/actions/checkout/action.yml
@@ -79,7 +79,7 @@ runs:
   - name: Checkout
     id: action
     continue-on-error: true
-    uses: actions/checkout@v4
+    uses: actions/checkout@11d5960a326750d5838078e36cf38b85af677262 # v4
     with:
       clean: ${{ inputs.clean }}
       fetch-depth: ${{ inputs.fetch-depth }}

```

Locations:

- `actions/checkout/action.yml`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed actions/checkout-merge/action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/actions/checkout-merge/action.yml b/actions/checkout-merge/action.yml
index 3c05709..eaa18d0 100644
--- a/actions/checkout-merge/action.yml
+++ b/actions/checkout-merge/action.yml
@@ -29,7 +29,7 @@ runs:
   steps:
   - name: Merge
     id: action
-    uses: check-spelling/checkout-merge@v0.0.6
+    uses: check-spelling/checkout-merge@29407b1b8660c562313ed65c81feab3e3255c03c # v0.0.6
     with:
       base_ref: ${{ inputs.base_ref }}
       do_not_report: ${{ inputs.do_not_report }}

```

Locations:

- `actions/checkout-merge/action.yml`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed actions/upload-artifact/action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/actions/upload-artifact/action.yml b/actions/upload-artifact/action.yml
index f624cb5..e29f181 100644
--- a/actions/upload-artifact/action.yml
+++ b/actions/upload-artifact/action.yml
@@ -35,7 +35,7 @@ runs:
   - name: Upload Artifact
     id: action
     continue-on-error: true
-    uses: actions/upload-artifact@v4
+    uses: actions/upload-artifact@ea165f8d65b6e75b540449e92b4886f43607fa02 # v4
     with:
       compression-level: ${{ inputs.compression-level }}
       if-no-files-found: ${{ inputs.if-no-files-found }}

```

Locations:

- `actions/upload-artifact/action.yml`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed actions/upload-sarif/action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/actions/upload-sarif/action.yml b/actions/upload-sarif/action.yml
index 2c71638..bd5ebee 100644
--- a/actions/upload-sarif/action.yml
+++ b/actions/upload-sarif/action.yml
@@ -51,7 +51,7 @@ runs:
   - name: Upload SARIF
     id: action
     continue-on-error: true
-    uses: github/codeql-action/upload-sarif@v3
+    uses: github/codeql-action/upload-sarif@1190a975f95ce23525efb6a3fc21ea29567c1b52 # v3
     with:
       category: ${{ inputs.category }}
       checkout_path: ${{ inputs.checkout_path }}

```

Locations:

- `actions/upload-sarif/action.yml`

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

**Fixes applied:** script-injection, github-env-injection, static-inline-injection, static-unsanitized-env-write, unpinned-uses, prior-fix

**Notes:**

action.yml: moved inputs.alternate_engine into env ALTERNATE_ENGINE and github.event.pull_request.number into env PR_NUMBER. Newlines are now stripped before writing to GITHUB_OUTPUT/GITHUB_ENV (repo, branch, THIS_GITHUB_JOB_ID, PRIVATE_SARIF_REF). Pinned actions/checkout, check-spelling/checkout-merge, actions/upload-artifact and github/codeql-action/upload-sarif to SHAs looked up with lookup_action_sha. upload-sarif now points at the current v3 SHA (9418214...), which is newer than the previous harden's 1190a97.... Not re-applied: the previous harden's YAML reformatting (flattened folded strings/expressions, which isn't security-related) and its swap of the t/with-dictionaries/*.php symlinks for regular files. Those are test fixtures, not part of the distributed action, and the swap isn't a security fix.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection, static-inline-injection

**Notes:**

In the 'parse alternate engine' step, inputs.alternate_engine moved into env ALTERNATE_ENGINE, which the script reads as "$ALTERNATE_ENGINE" (double-quoted) through printf. The perl parsing is unchanged. Captures use '|| true' to keep upstream's echo-wrapped behaviour where a failure did not stop the step. repo and branch have newlines stripped (printf | tr -d '\n\r') before being written to GITHUB_OUTPUT.

### Iteration 3

**Fixes applied:** github-env-injection, script-injection

**Notes:**

Both fixes are in action.yml; nothing was run or tested.

shim step: the result of realpath now goes into raw_action_path first. THIS_ACTION_PATH is then set by stripping newlines from it with printf '%s' | tr -d '\n\r'. Because the realpath call is still a plain assignment on its own line, a realpath failure still stops the step. The cleaned THIS_ACTION_PATH is the value written to both GITHUB_ENV and GITHUB_PATH.

act-windows-shim step: the variable is now quoted and the globs stay outside the quotes ("$THIS_ACTION_PATH"/*.sh and so on) in both the chmod and perl commands.

