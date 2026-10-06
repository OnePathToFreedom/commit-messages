# commit-messages

Agent-agnostic rules that make any AI coding tool write accurate, well-structured git commit messages — grounded in the actual diff, not in memory of what it meant to do.

> It does not replace your judgment on *what* to commit. It makes sure the message that goes with it is true, specific, and in the right format, every time — whichever AI (or human) is doing the committing.

## The problem it solves

Left to its own habits, an AI agent committing on your behalf tends to:

- write generic subjects ("update files", "fix bug") that say nothing a future `git log` reader can use
- describe what it *meant* to do rather than what the diff actually contains
- skip the body on "small" changes, even when a one-line *why* would save someone time later
- drift between formats from one commit to the next
- quietly commit two unrelated changes together because splitting them felt like extra work

The rules here close all five, by forcing one thing before anything else: **read `git diff --staged` before writing a word of the message.** Context tells an agent what it intended; the diff tells it what actually landed. The message is written from the diff.

## Format

```
<type>(<scope>): <subject>

- <bullet grounded in the diff>
- <bullet>
```

[Conventional Commits](https://www.conventionalcommits.org/) types (`feat`, `fix`, `refactor`, `perf`, `docs`, `style`, `test`, `chore`, `ci`), `!` + `BREAKING CHANGE:` for anything that breaks a contract, imperative mood, ~72-character soft limit on the subject — and a body with at least one bullet **every time**, even for a one-line fix. Full rules in [`AGENTS.md`](AGENTS.md), including what to do when staged changes mix unrelated things (hint: it proposes splitting them — see [`examples/02-mixed-changes.md`](examples/02-mixed-changes.md)).

## Three layers, so it works with whatever you're using

This isn't tied to one agent. Pick the layer(s) that match your setup — they're independent and stack fine.

| Layer | What it is | Works with |
|---|---|---|
| [`AGENTS.md`](AGENTS.md) | The rules, as an instructions file | Cursor, GitHub Copilot, Gemini CLI, Windsurf, Aider, OpenAI Codex, Zed, Devin, Warp, Amp — the 20+ tools that read the open [AGENTS.md](https://agents.md) convention |
| [`CLAUDE.md`](CLAUDE.md) | Symlink → `AGENTS.md` | Claude Code, which reads `CLAUDE.md` rather than `AGENTS.md` natively |
| [`SKILL.md`](SKILL.md) | The same rules as a Claude Code **skill** | Claude Code in skill mode — adds an explicit trigger ("use this every time before `git commit`") instead of relying on the agent to notice ambient repo instructions |
| [`hooks/commit-msg`](hooks/commit-msg) | A real git hook | Literally anything that runs `git commit` — any AI tool, any human, any CI step. Mechanical only: checks the message *looks* like Conventional Commits, can't check it's *true* to the diff (that needs judgment, which is what the other three layers are for) |

### Setup

**Repo-wide instructions (works with most AI coding tools):**
```bash
cp AGENTS.md /path/to/your-repo/AGENTS.md
ln -s AGENTS.md /path/to/your-repo/CLAUDE.md   # if you also use Claude Code
```

**As a Claude Code skill** (explicit trigger, independent of whether the repo has an AGENTS.md):
```bash
mkdir -p ~/.claude/skills
cp -r . ~/.claude/skills/commit-messages
```

**Format backstop (any tool, any repo):**
```bash
cp hooks/commit-msg /path/to/your-repo/.git/hooks/commit-msg
chmod +x /path/to/your-repo/.git/hooks/commit-msg
```
Blocks a commit whose subject line doesn't match `type(scope): subject`; warns (without blocking) on a missing body or an over-long subject. Bypass once with `git commit --no-verify`.

## Examples

Real output from running these rules against real diffs (see [`examples/`](examples/)):

- [`01-simple-fix.md`](examples/01-simple-fix.md) — a one-line bug fix still gets a grounded body, not just a bare subject
- [`02-mixed-changes.md`](examples/02-mixed-changes.md) — unrelated changes in two modules get flagged and a split is proposed, instead of being squashed into one commit
- [`03-breaking-change.md`](examples/03-breaking-change.md) — a contract-breaking change, formatted with `!` and a `BREAKING CHANGE:` footer

## License

MIT, see [LICENSE](LICENSE).
