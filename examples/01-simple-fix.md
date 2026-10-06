# Example 1 — a one-line fix still gets a grounded body

## The staged diff

```diff
diff --git a/src/auth.js b/src/auth.js
index ec84f69..7f78af5 100644
--- a/src/auth.js
+++ b/src/auth.js
@@ -1,4 +1,4 @@
 function isExpired(token) {
-  return token.expiresAt <= Date.now();
+  return token.expiresAt < Date.now();
 }
 module.exports = { isExpired };
```

## What an agent writes without this skill

```
fix bug
```

True in the sense that a bug was fixed. Useless to anyone scanning `git log` six months from now, or trying to figure out via `git bisect` which commit changed expiry behavior.

## What this skill produces

```
fix(auth): correct token expiry check off-by-one

- Session tokens were treated as expired one second early due to a
  `<=` vs `<` comparison against the expiry timestamp
```

The subject names the actual bug class (off-by-one), not just "a bug." The body exists even though the diff is a single line — because the *why* (the off-by-one in the comparison operator) isn't obvious from the subject alone, and it costs one sentence to include.
