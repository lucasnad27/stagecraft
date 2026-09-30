---
name: stagecraft
description: Research → design → plan → implement → review for a codebase, with a human checkpoint at each stage and a written artifact between them. Use when the user wants to research how something works before changing it, align on a design, turn a design into a phased plan, implement a plan phase by phase, or prepare a pull request with a review guide. Also `setup` to write STAGECRAFT.md for a repo and `guide` for orientation.
user-invocable: true
argument-hint: "[setup|research|design|plan|implement|review|guide] [args]"
license: MIT
metadata:
  version: 0.1.0
---

# Stagecraft

One workflow, five stages, a human at every seam. Each stage reads the artifact the previous one wrote, does its work in a fresh context, and leaves the next artifact behind. The friction is the point: understanding is created at the checkpoints, and the artifacts carry it forward.

The workflow never changes per project. **`STAGECRAFT.md` at the repo root** carries everything project-specific — commands, tracker, forge, commit convention, constraints, and per-stage guidance — and every stage reads it at runtime. `setup` writes it.

## Setup

1. `<skill-dir>` below means the directory containing this SKILL.md. Resolve `reference/` and `agents/` paths relative to it.
2. Read `STAGECRAFT.md` at the repo root. Each stage's reference says which sections it uses. If the file is missing, run with what you can detect, say so once, and suggest `/stagecraft setup` when the stage ends.
3. Load the verb's reference from the table and follow it. Load nothing else up front.

## Commands

| Verb | Description | Reference | Model hint |
|---|---|---|---|
| `setup` | Detect the project and write or upgrade `STAGECRAFT.md` | [reference/setup.md](reference/setup.md) | strongest, high effort |
| `research [question \| issue \| file]` | Document how the code works today, fanning out across sub-agents | [reference/research.md](reference/research.md) | strongest, highest effort |
| `design [research doc \| issue]` | Settle the approach in ~200 lines plus a concrete reference artifact | [reference/design.md](reference/design.md) | strongest, highest effort |
| `plan [design doc]` | Vertical phases with per-phase verification and a Completion block each | [reference/plan.md](reference/plan.md) | strongest, highest effort |
| `implement [plan]` | Execute phase by phase; pause for manual checks | [reference/implement.md](reference/implement.md) | strongest, highest effort |
| `review [PR number \| branch] [--no-stops] [--walk\|--no-walk]` | Commit, open the PR, land a numbered review guide | [reference/review.md](reference/review.md) | strongest, high effort |
| `guide [topic]` | Where am I, what's next; topic deep dives | [reference/guide.md](reference/guide.md) | smallest, low effort |

Routing:

- **A verb as the first word of the arguments:** load its reference and follow it. `$ARGUMENTS` in a reference means the rest of the arguments after the verb.
- **No argument:** run `guide`.
- **Old names** (`research-codebase`, `design-doc`, `create-plan`, `implement-plan`, `prepare-pr`) map to the verbs in order. Accept them silently.
- **Otherwise:** state which verb you think they mean and run it.

The model hint is advisory. Set the model and effort if the harness lets you; ignore it if not.

## Sub-agents

`agents/<name>.md` is the complete prompt for each research agent: `codebase-locator`, `codebase-analyzer`, `codebase-pattern-finder`, `artifact-locator`, `artifact-analyzer`, `query-planner`, `web-search-researcher`, `branch-ticket-detector`. A project may add its own under `STAGECRAFT.md` → **Agents**, pointing at prompt files in the repo.

- **Harness with sub-agents** (Claude Code's Agent tool, Codex's `spawn_agent`, OpenCode's task tool): spawn each with its file as the prompt, plus the task. Spawn fresh; never fork the current conversation into a reviewer or researcher. Run independent agents in parallel in one turn.
- **Harness without sub-agents** (pi and others): read `reference/degraded/<name>.md` instead. It is the same prompt with a preamble for running the role inline, sequentially. Say once per stage that you're running the fan-out inline.
- On Claude Code the plugin also registers the agents by name, so `codebase-locator` resolves as a named agent with its tool restrictions. The prompt file is still the source of truth.

## Artifacts

Everything the stages write goes to `.stagecraft/` at the repo root, flat, gitignored: `{research,design,plan,review}-YYYY-MM-DD-[KEY-]description.md`. The type prefix is load-bearing — `implement` and `review` find the review metadata by swapping `plan-` for `review-`. Design mockups sit beside their doc as `.html`. `.stagecraft/` is per-worktree working state; anything that has to outlive the merge goes in the PR.

## Integrations

`STAGECRAFT.md` → **Integrations** lists optional tools the project has opted into. Each has a reference under `reference/integrations/`; read it only when the file lists the tool.

- `herdr` — marks each stage on the agent's tab. Run the script in [reference/integrations/herdr.md](reference/integrations/herdr.md) at the start of every stage.
- `tuicr` — walks the review guide with the author in their terminal. `review` reads [reference/integrations/tuicr.md](reference/integrations/tuicr.md) for its walkthrough step.

Nothing listed means nothing runs. `setup` offers each tool it finds on the PATH.

## Harness notes

- Read files whole. Where a reference says to read something FULLY, it means no offset or limit.
- "Ask the user" means one batched message, or the harness's structured question tool if it has one. Never one question per turn.
- "Track the sub-tasks" means whatever task list the harness offers, or a short list in your reply if it has none.
- Frontmatter `model` and `tools` on agent files are hints. Harnesses that don't read them ignore them.
- `review` posts inline comments through `gh`. Without `gh` or off GitHub, `--no-stops` is the default and the guide lives in the PR description.
