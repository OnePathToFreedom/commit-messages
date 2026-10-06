# Example 2 — unrelated changes get flagged, not squashed together

## The staged diff

```diff
diff --git a/src/notifications/notify.js b/src/notifications/notify.js
index f053ebf..cc40cf6 100644
--- a/src/notifications/notify.js
+++ b/src/notifications/notify.js
@@ -1 +1,5 @@
-module.exports = {};
+function sendPush(userId, message) {
+  // new push notification channel
+  return { userId, message, channel: "push" };
+}
+module.exports = { sendPush };
diff --git a/src/payments/charge.js b/src/payments/charge.js
index bb2bf46..300bc8e 100644
--- a/src/payments/charge.js
+++ b/src/payments/charge.js
@@ -1,4 +1,5 @@
 function charge(amount) {
+  if (amount <= 0) throw new Error("invalid amount");
   return amount;
 }
 module.exports = { charge };
```

Two unrelated things ended up staged together: a validation fix in `payments`, and a brand-new feature in `notifications`.

## What an agent does without this skill

Picks whichever type seems to dominate (often whatever was mentioned last in the conversation) and commits everything under it — e.g. `feat(notifications): add push and fix charge validation`, burying the fix inside a feature commit where no one will think to look for it, or vice versa.

## What this skill does

It stops before committing and proposes a split instead:

> Your staged changes touch two unrelated things: a new `sendPush` feature in notifications and a validation fix in payments. Per the commit-messages skill, I'd split these into two commits rather than merge them under one type. Want me to:
> 1. Unstage everything (`git reset`), then stage and commit `src/payments/charge.js` first, then `src/notifications/notify.js` separately?
> 2. Or do you want them combined into a single commit anyway?

If the user agrees to split:

```
fix(payments): reject non-positive charge amounts

- `charge()` now throws "invalid amount" for amounts <= 0, preventing
  zero/negative charges from silently succeeding
```

```
feat(notifications): add push notification channel

- Implement `sendPush(userId, message)` returning a push-channel
  notification payload, exported from notify.js
```

Each commit now has one clean type, one clean scope, and a `git log` entry someone can actually trust.
