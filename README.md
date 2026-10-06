# commit-messages

A Claude Code skill that makes Claude write accurate, well-structured git commit messages — grounded in the actual diff, not in memory of what it meant to do.

> It does not replace your judgment on *what* to commit. It makes sure the message that goes with it is true, specific, and in the right format, every time.

## The problem it solves

Left to its own habits, an AI agent committing on your behalf tends to:

- write generic subjects ("update files", "fix bug") that say nothing a future `git log` reader can use
- describe what it *meant* to do rather than what the diff actually contains
- skip the body on "small" changes, even when a one-line *why* would save someone time later
- drift between formats from one commit to the next
- quietly commit two unrelated changes together because splitting them felt like extra work

This skill closes all five, by forcing one rule before anything else: **read `git diff --staged` before writing a word of the message.** Session context tells Claude what it intended; the diff tells it what actually landed. The message is written from the diff.

## Format

```
<type>(<scope>): <subject>

- <bullet grounded in the diff>
- <bullet>
```

[Conventional Commits](https://www.conventionalcommits.org/) types (`feat`, `fix`, `refactor`, `perf`, `docs`, `style`, `test`, `chore`, `ci`), `!` + `BREAKING CHANGE:` for anything that breaks a contract, imperative mood, ~72-character soft limit on the subject — and a body with at least one bullet **every time**, even for a one-line fix. See [`SKILL.md`](SKILL.md) for the full rules, including what to do when staged changes mix unrelated things (hint: it proposes splitting them, see [`examples/02-mixed-changes.md`](examples/02-mixed-changes.md)).

## Usage

### As a Claude Code skill

```bash
mkdir -p ~/.claude/skills
cp -r commit-messages ~/.claude/skills/commit-messages
```

Claude Code picks it up automatically — no slash command needed. It triggers every time Claude is about to run `git commit`, whether you asked for a commit directly or it's committing as a step inside a larger task.

### As project guidance

Point your `CLAUDE.md` at it, or paste `SKILL.md`'s rules section directly into a project's commit guidelines for human contributors — the format and reasoning hold either way.

## Examples

Real output from running this skill against real diffs (see [`examples/`](examples/)):

- [`01-simple-fix.md`](examples/01-simple-fix.md) — a one-line bug fix still gets a grounded body, not just a bare subject
- [`02-mixed-changes.md`](examples/02-mixed-changes.md) — unrelated changes in two modules get flagged and a split is proposed, instead of being squashed into one commit
- [`03-breaking-change.md`](examples/03-breaking-change.md) — a contract-breaking change, formatted with `!` and a `BREAKING CHANGE:` footer

## License

MIT, see [LICENSE](LICENSE).
