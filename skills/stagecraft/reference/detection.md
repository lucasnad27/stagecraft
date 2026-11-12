# Project Detection

Reference for Step 1 (analyze) and Step 2 (fill gaps) of the setup skill.

## What to read

Read whichever of these exist, and infer the stack from them:

| File | Tells you |
|---|---|
| `package.json` | Node/TypeScript; scripts block usually has every command you need |
| `Cargo.toml` | Rust |
| `go.mod` | Go |
| `pyproject.toml`, `requirements.txt` | Python; check for poetry/uv/pdm |
| `Makefile` | Often the real entry point — check for `test`, `lint`, `build` targets |
| `.git/config` | Remote host hints at the issue tracker (github.com, gitlab.com) |

Also note whether `.stagecraft/` or an older `.rpi/` artifacts directory already exists (see "Detecting an existing installation").

## What to extract

- Primary language and framework
- Package manager and available scripts
- Test commands — unit, integration, e2e (they're often distinct)
- Lint, format, build, and type-check commands
- Database tooling and its migration workflow (Prisma, SQLAlchemy/Alembic, Django ORM, Diesel, Drizzle)
- Directory structure conventions
- Issue tracker
- Commit message convention (see below)
- Existing artifacts directory, if any
- Optional tools on the PATH that have a file under `reference/integrations/`

## Detecting the issue tracker

Check for CLIs (`which linear`, `which gh`, `which glab`), ticket files (`.stagecraft/ticket-*.md`), MCP tools for the tracker, and the git remote host. Note both what the project uses *and* whether the CLI or tool is actually available — the stages depend on it.

Install hints if a CLI is missing:
- Linear: `npm install -g @linear/cli`
- GitHub: https://cli.github.com
- GitLab: https://gitlab.com/gitlab-org/cli

## Detecting the commit convention

`/stagecraft review` writes commit messages, and inferring the convention fresh on every PR is where it
gets this wrong — Conventional Commits is the common guess and plenty of repos don't use it.
Resolve it once, here.

```bash
git log --no-merges --format=%s -30
```

Read the subjects and record **which convention, plus one real example from this repo**. The
example is the part that carries; a label alone still leaves room to guess wrong.

- `feat(scope): lowercase summary` → Conventional Commits. Note whether scopes are actually used.
- `Add the fallow health check` → imperative sentence case, no type prefix.
- Anything with a ticket key (`PROJ-12: …`, `[#451] …`) → record the position and bracket style.

If the last 30 subjects disagree with each other, say so rather than picking a winner, and ask.
A repo with no convention is a real answer — write "no convention; match surrounding history."

## Presenting findings

Show what you found as a compact list — language, framework, package manager, test/lint/format/build/typecheck commands, database, issue tracking, the commit convention with its example, and the artifacts directory. Mark anything you couldn't determine as needing input rather than guessing.

## Filling gaps

Ask for everything you couldn't detect **in as few turns as possible** — every extra turn costs the user more than answering a batched prompt once.

- Command and path gaps are free-text. Present them as a single numbered block the user fills in one pass, with a realistic example per line and an explicit opt-out (`none` / `skip`).
- The issue-tracker choice is multiple choice (Linear / GitHub / GitLab / local files / none). Use the harness's structured question tool for it if there is one, and batch any follow-up into the same call.
- Only split into a second round when a later question genuinely depends on an earlier answer.

For a detected database with an unclear migration workflow, ask for two things: how schema changes are applied during development, and how formal migrations are created.

## Confirming

Present the complete resolved configuration and get a yes before writing anything. This is the last cheap moment to correct a wrong assumption.

## Detecting an existing installation

- `STAGECRAFT.md` at the repo root → **upgrade**. Its header comment carries the version it was written against.
- `.claude/skills/research-codebase/SKILL.md` or `.claude/.rpi-version` → **migration from research-plan-implement**. `.rpi-version` gives the version; without it, `.claude/agents/artifact-locator.md` means 5.0+ and `.claude/skills/prepare-pr/` alone means 3.x to 4.1.
- Neither → **fresh install**, continue with Step 2.

Both paths are described in `setup.md`. An RPI install may write artifacts somewhere other than `.rpi/`; read the root from the `path:` scope in `.claude/agents/artifact-locator.md` and cross-check against what's on disk before renaming anything.

## When detection fails

If you can't determine the project type at all, ask directly: what language/framework, how tests run, how linting and formatting work. If there's no test command, offer to skip test-related sections rather than inventing one.
