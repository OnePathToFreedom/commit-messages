---
name: commit-messages
description: Write accurate, well-structured git commit messages. Use this skill every single time you are about to run `git commit` — whether the user explicitly asked for a commit, or you're committing as a step inside a larger task. Do not skip this for "quick" or "obvious" commits; a one-line fix still needs a message that reflects what the diff actually contains, not a guess from memory. Also use it when the user asks to rewrite, clean up, split, or review a commit message or a batch of commits.
---

# Commit Messages

Git commit messages exist for people who are not you, reading them months later with zero context — scanning `git log --oneline`, writing a changelog, or running `git bisect` trying to find which commit broke something. A message is only useful to them if it's true, specific, and found in the right place by anyone scanning the log quickly.

This skill has one job: make sure every commit message is **grounded in what actually changed**, not in what you remember doing or intended to do.

## The core rule: read the diff first, every time

Before writing a single word of the message, run:

```bash
git diff --staged
```

Not `git status` alone (that only shows filenames), and not relying on your memory of the conversation. Session context tells you *intent* — the diff tells you *what actually landed in this commit*. They drift apart more often than you'd expect: a refactor leaves debug code behind, a "simple fix" touches three unrelated files, an edit you thought you reverted is still staged.

Write the message from the diff. If something in the diff surprises you — a file you don't remember touching, a change bigger or smaller than expected — stop and look closer before describing it. Never describe a change you have not actually seen in the diff output.

## Format: Conventional Commits, strictly

```
<type>(<scope>): <subject>

- <bullet explaining what changed and, where it's not obvious, why>
- <bullet>
- <bullet>
```

**Type** — pick the one that matches the diff's primary effect:

| Type | When |
|---|---|
| `feat` | a new capability for the end user |
| `fix` | a bug fix |
| `refactor` | code restructured, no behavior change |
| `perf` | a performance improvement |
| `docs` | documentation only |
| `style` | formatting, whitespace, no logic change |
| `test` | adding or fixing tests only |
| `chore` | tooling, deps, build config, no source behavior change |
| `ci` | CI/CD pipeline changes |

Add `!` after the type (or scope) and a `BREAKING CHANGE:` line in the body for anything that breaks a public API, config format, or CLI contract — e.g. `feat(api)!:`.

**Scope** — the module, package, or area touched (`auth`, `parser`, `cli`, the directory name — whatever the project's convention is). Skip scope only if the change is genuinely repo-wide and no single scope fits; don't skip it out of laziness.

**Subject** — imperative mood ("add", not "added" or "adds"), no period at the end, describes the net effect of the diff. Soft limit **~72 characters**. If the honest, specific subject runs longer, let it — a true 80-character line beats a vague 60-character one. Never cut the subject down by making it vaguer.

**Body — always include it, even for small commits.** This is a deliberate departure from the common advice to skip the body on trivial changes: a one-line bullet of *why* costs almost nothing to write and saves real time for whoever reads the log later wondering what problem this solved. Every commit gets at least one bullet. For commits touching several things, one bullet per logical change, each grounded in something actually visible in the diff.

## Examples

**Small fix — still gets a body:**
```
fix(auth): correct token expiry check off-by-one

- Session tokens were treated as expired one second early due to a
  `<=` vs `<` comparison against the expiry timestamp
```

**Larger feature:**
```
feat(export)!: switch CSV export to streaming writer

- Replace in-memory buffer with a streaming writer so exports no
  longer load the full dataset into RAM
- Add `--chunk-size` CLI flag to control batch size
- BREAKING CHANGE: `exportToCsv()` now returns a stream instead of
  a string; callers awaiting a string result must update
```

**What NOT to do:**
```
update files
fix bug
improve code
```
These tell a future reader nothing. If you catch yourself about to write something this generic, go back to `git diff --staged` — the specific words are in there.

## When staged changes mix unrelated things

If `git diff --staged` shows changes that don't belong to one logical unit — say, a bug fix in the payment module alongside a new feature in the notifications module — don't force them into one commit with a type that only half-fits. Stop before committing and tell the user what you see, proposing the split with draft messages for each piece. Only commit everything together if the user says that's fine, or if the pieces genuinely are one logical change (e.g., a rename that touches many files).

To actually split: `git reset` to unstage everything, then re-stage and commit each logical group on its own.
- If the groups line up cleanly with whole files, `git add <path>` per group is enough.
- If a single file mixes both (e.g. one function fixed, a different function extended), use `git add -p <path>` to stage only the relevant hunks for each commit, or edit the file's working copy to temporarily hold out the other group's lines if the hunks aren't cleanly separable.

## Language

Always write commit messages in English, regardless of what language the conversation is in. This is independent of the project's existing commit history — if the repo's past commits are in another language, that's their convention to change if they want; don't silently switch based on old commits, and don't ask every time — just use English by default.

## Quick checklist before running `git commit`

1. Ran `git diff --staged` and actually read the output
2. Type matches the diff's primary effect, scope names the real area touched
3. Subject is imperative, true to the diff, ~72 chars or less unless precision needs more
4. Body has at least one bullet grounded in something visible in the diff
5. If the diff mixes unrelated changes, flagged that to the user before committing everything as one
