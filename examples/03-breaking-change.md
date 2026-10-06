# Example 3 — a change that breaks a contract

## The staged diff (abridged)

```diff
diff --git a/src/export/csv.js b/src/export/csv.js
--- a/src/export/csv.js
+++ b/src/export/csv.js
@@ -4,11 +4,9 @@
-function exportToCsv(rows) {
-  const buffer = rows.map(toCsvLine).join("\n");
-  return buffer;
-}
+function exportToCsv(rows) {
+  return createReadStream(rows); // returns a Readable now, not a string
+}
```

Plus a new `--chunk-size` CLI flag wired up elsewhere in the diff.

## What this skill produces

```
feat(export)!: switch CSV export to streaming writer

- Replace in-memory buffer with a streaming writer so exports no
  longer load the full dataset into RAM
- Add `--chunk-size` CLI flag to control batch size
- BREAKING CHANGE: `exportToCsv()` now returns a stream instead of
  a string; callers awaiting a string result must update
```

Two things to note:

- The `!` after the type/scope is the fast visual signal (shows up in `git log --oneline`); the `BREAKING CHANGE:` footer is what tools like `semantic-release` and `conventional-changelog` actually parse to bump a major version and list it prominently in the changelog.
- The breaking change is one bullet among others, not the whole message — the commit still describes the full diff (the streaming rewrite *and* the new flag), because both are really there.
