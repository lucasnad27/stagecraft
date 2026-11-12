# Setup

You are tasked with writing `STAGECRAFT.md` for this repo: the one committed file every stage reads at runtime. Detection does the work; the user confirms; you write one file. Nothing else is generated.

Supporting references, read when you reach the step that needs them:

- **`detection.md`** — what to read, what to extract, how to fill gaps (Steps 1 and 2)
- **`templates/STAGECRAFT.md`** — the file's shape and what each slot is for (Step 4)

## Before you start

If the tree is dirty or the user is on the default branch, suggest a branch so the new file lands as a reviewable diff. Don't insist.

## Step 1: Detect

Read `detection.md`, then inspect the project: language, framework, package manager, test/lint/format/build/typecheck commands, database tooling, issue tracker, forge, commit convention with a real example, and where a feature's files usually live.

**Then check for an existing installation:**

- `STAGECRAFT.md` exists → **upgrade**, see below.
- `.claude/skills/research-codebase/` or `.claude/.rpi-version` exists → **migration from research-plan-implement**, see below.
- Neither → fresh install; continue.

Present what you found as a compact list, marking anything you couldn't determine.

## Step 2: Fill the gaps

Ask for what you couldn't detect in as few turns as possible: one numbered block for free-text answers (commands, paths) with a realistic example per line and an explicit `skip`, and the harness's structured question tool for the tracker choice if it has one. Then present the complete configuration and get a yes before writing anything.

## Step 3: Draft the guidance

Detection gives you the facts. The **Constraints** and **Guidance** sections are judgment, and they are what make the workflow fit this repo. Draft them from what the repo already says about itself:

- `AGENTS.md`, `CLAUDE.md`, `CONTRIBUTING.md`, and the README for conventions that apply without being asked.
- Decision records (`docs/adr/`, `docs/decisions/`) for rules that fail late: a schema that isn't the deployment path, an env system with build-time inlining, a deploy order that forbids drop-and-rename.
- CI config for checks that run on every PR beyond the standard commands.
- Test layout for how manual verification is done: seeded personas, a testing environment, a command that prints credentials.

Write one sentence per finding in the slot that reads it. Don't invent guidance the repo doesn't support; an empty slot is honest. Show the draft and let the user cut or add before writing.

**Integrations** are opt-in. For each tool with a file under `reference/integrations/`, check `command -v <tool>`; if it's on the PATH, offer it in one line and list it only on a yes. Never list a tool the user didn't confirm.

## Step 4: Write the file

Write `STAGECRAFT.md` at the repo root from `templates/STAGECRAFT.md`, filled in the project's own vocabulary and stripped of every bracketed hint. Record this skill's version from `SKILL.md`'s frontmatter in the header comment.

Then:

1. Add `.stagecraft/` to `.gitignore` if it isn't there. If artifacts are already tracked, offer `git rm --cached -r .stagecraft/`.
2. Create `docs/stagecraft/agents/` only if the project has agents to put there; on Claude Code, offer a symlink from `.claude/agents/<name>.md` to each so they register by name.
3. Say what to run next: `/stagecraft guide`, then `/stagecraft research` on the first issue.

## Upgrade

`STAGECRAFT.md` exists. Compare the version in its header comment to this skill's. Read the changelog entries between them and tell the user, in a few lines, what changed in the stages and whether any slot in their file is new, renamed, or no longer read. Offer to add the new slots empty and update the version. Never rewrite their guidance; it is theirs.

## Migration from research-plan-implement

The repo has generated skills under `.claude/skills/` and agents under `.claude/agents/`. Everything project-specific the old install knew lives only inside that prose, so recover it before anything is deleted:

1. **Diff the installed skills against the templates they came from.** The tag is the version in `.claude/.rpi-version`; the templates are in the `research-plan-implement` plugin at that tag. Everything that isn't the template is either detected facts or project guidance.
2. **Sort each difference into a slot.** Commands, tracker, forge, and commit convention are facts. Sentences about how a stage should behave here go under the stage's guidance. Rules that appear in several stages go under **Constraints** once. A project-authored agent goes under **Agents**, and its prompt file moves to `docs/stagecraft/agents/`.
3. **Flag anything that doesn't fit a slot.** Show it to the user; it is either a generic improvement worth sending upstream or a case for a full stage override.
4. **Rename `.rpi/` to `.stagecraft/`** in the current worktree, update `.gitignore`, and list the other worktrees still carrying `.rpi/`.
5. **Write `STAGECRAFT.md`**, then offer to delete the generated skills, agents, `.claude/scripts/herdr-phase.sh`, and `.claude/.rpi-version`. Project skills that are not part of the workflow stay.
