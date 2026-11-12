# Guide

You provide contextual orientation for the research → design → plan → implement → review workflow.

**Two modes:**
- `/stagecraft guide` (no args) → Short orientation: where am I, what's next? (5-10 lines max)
- `/stagecraft guide [topic]` → Deep dive on a specific topic (see `reference/guide-topics.md`)

## Project context

Read `STAGECRAFT.md` at the repo root. This stage uses **Artifacts** and **Guidance → Guide**. If the file is missing, orientation is **Not set up** and the next step is `/stagecraft setup`.

## Mode 1: Orientation (no args)

### Current Workspace State

Run these and read the output:

```bash
ls -lt .stagecraft 2>/dev/null | grep -E ' research-.*\.md$' | head -3
ls -lt .stagecraft 2>/dev/null | grep -E ' design-.*\.md$' | head -3
ls -lt .stagecraft 2>/dev/null | grep -E ' plan-.*\.md$' | head -3
git status --short 2>/dev/null
gh pr status 2>/dev/null | head -5
```

`ls | grep` rather than a glob: an unmatched glob fails under zsh before the redirect applies.

### Determine Current Phase

Based on the workspace state above:

- No artifacts → **Getting started**
- Research doc exists, no design → **Ready for design**
- Design doc exists, no plan → **Ready for planning**
- Plan doc exists, some phases incomplete → **In implementation**
- Plan doc with all phases complete → **Ready for review**
- Open PR → **In review**

### Output Format

Respond with ONLY this format:

```
**[Current phase]**
[Most relevant artifact with path]
Next: [What to do next with the specific command]
Tip: [One-line tip relevant to current phase]
```

### Phase-Specific Tips

- **Getting started:** "Start with /stagecraft research to explore the area you'll be working in."
- **Ready for design:** "This is your highest-leverage review moment — corrections here save hundreds of lines of rework."
- **Ready for planning:** "/stagecraft plan takes your design and produces vertical phases with per-phase testing."
- **In implementation:** "Each phase should be testable on its own. If it's not, the plan may need vertical restructuring."
- **Ready for review:** "Run /stagecraft review in a fresh session to commit, open the PR, and land a numbered review guide — a short index in the description, the detail as inline comments on the diff."
- **In review:** "Each stop is a resolvable thread; resolving one ticks it off. Point /stagecraft review at an existing PR number to write a guide onto a PR opened outside the loop."

**Guidance → Guide** may add a project-specific tip; prefer it when it applies.

### Orientation Rules

- **5-10 lines max** — This is orientation, not a tutorial
- **Show the most recent relevant artifact** — Not all of them
- **One tip only** — Relevant to where they are right now
- **Don't explain the workflow** — Just show where they are and what's next
- **If artifacts are ambiguous**, show the most recent and mention others exist

## Mode 2: Topic Deep Dive (with args)

If `$ARGUMENTS` is provided, it's a topic name. Read `reference/guide-topics.md` and present the content for that topic.

**Available topics:** overview, research, design, plan, implement, review, herdr, context, patterns, tips, examples

Topics are named for the *phase*, so the old skill names are accepted aliases — `design-doc` → `design`, `research-codebase` → `research`, `create-plan` → `plan`, `implement-plan` → `implement`, `prepare-pr` → `review`. Resolve the alias and show the topic; don't make the user guess again.

If the topic still isn't recognized, show the list of available topics.
