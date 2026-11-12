# Design

You are tasked with creating a lightweight design discussion document (~200 lines) that captures the shared understanding between you and the user about what's being built and how. This is the highest-leverage review moment — corrections here save hundreds of lines of rework downstream.

## What a design doc is

~200 lines of alignment: current state, desired end state, patterns to follow, testing approach.

File-by-file changes, code snippets, and phase ordering belong in `/stagecraft plan` — this doc is what you and the user agree on before that exists. It's enough to agree on direction, not enough to implement from.

## Project context

Read `STAGECRAFT.md` at the repo root first. This stage uses **Project**, **Commands** (what verification exists), **Tracker**, **Constraints**, and **Guidance → Design**. If the file is missing, say so once, work from what the tree shows, and suggest `/stagecraft setup` at the end.

## Initial Response

1. **If parameters were provided** — a research doc path or issue reference in `$ARGUMENTS` — read them FULLY, both if both are given, and begin.

2. **If no parameters were provided, find the research doc before asking.** Do not immediately ask the user to hand over a file:

   1. **Read the current branch**: `git branch --show-current`. Extract the issue key and slug in the shape **Tracker** describes (e.g. `feat/327-enum-consolidation` → issue `327`, slug `enum-consolidation`).
   2. **Search `.stagecraft/research-*.md`**, matching on the issue key first, then on slug keywords. If several match, prefer the most recent by date prefix.
   3. **Act on what you find:**
      - **Exactly one strong match** → use it. State which file you picked and why, then proceed. The user can redirect if it's wrong.
      - **Several plausible matches** → list the top 2-3 (most recent first) and ask which, rather than making the user type a path.
      - **No match** → only then fall back to asking:

        ```
        I couldn't find a research doc for this branch in .stagecraft/research-*.md.

        Please provide:
        1. A research document (from /stagecraft research) or describe what you want to build
        2. Optionally, an issue reference

        Tip: /stagecraft design .stagecraft/research-2026-01-05-auth-flow.md
        ```

        Then wait for the user's input.

## Process

### Step 1: Gather Context

1. **Read all provided files FULLY** — research docs, issues, related docs
2. **Follow the issue's links.** If the issue references another issue, a design or spec issue, a handoff comment, or a prior research doc, read those too using the **Tracker** fetch command. They usually carry the scope, acceptance criteria, locked decisions, and component spec. The answer to "what are we building and how much of it" almost always lives here, not in the user's head.
3. **Check the project's own decision records**, if **Guidance → Research → extra sources** names any. A design that quietly contradicts a binding decision is the expensive kind of wrong.
4. **Spawn light confirmation agents** (if needed):
   - Use **codebase-pattern-finder** to confirm patterns mentioned in research
   - Use **codebase-locator** to verify file locations if uncertain
   - These are quick confirmations, NOT a full research pass
5. **Read any files identified by confirmation agents**

### Step 2: Draft the Design Document

Write the design document to `.stagecraft/design-YYYY-MM-DD-[KEY-]description.md`

Use the template from `reference/templates/design.md` as the starting structure. Adapt each section to the specific feature — the template shows what each section should contain and how to adapt based on the shape of the work. **Constraints** and **Guidance → Design** carry what this project needs every design to address: name the ones this work touches in the doc rather than discovering them mid-implementation.

**Produce a concrete reference artifact wherever the work has a shape you can render.** A mockup settles more design questions than three paragraphs describing the same screen, and real JSON settles an API contract faster than a description of it. The user reviews the artifact, not your description of it — and the implementing agent builds against it directly.

Where the artifact lives depends on the project. If **Guidance → Design** names a design tool as the source of truth for UI, name the file and artboard in the Concrete Reference section rather than copying the design into the repo, and record the tool → app token mapping there so implementation doesn't re-derive it. Otherwise, or when no artboard exists, write a self-contained HTML mockup to `.stagecraft/design-YYYY-MM-DD-description.html` and link it. If an earlier HTML view (a `/show-me` page) settled a question, move it there too; later phases read only what the design links.

### Step 3: Walk Through Open Questions

0. **First, eliminate every question the source documents already answer.** Before composing anything for the user, check each candidate question against the issue body, its acceptance criteria, the linked spec issue and its handoff comments, and the research doc's resolved decisions. If the answer is written there, it is decided — record it as a Key Decision, do not ask. **Scope in particular comes from the issue, never from asking the user to restate it.** Asking the user to re-confirm something the issue already states is the most common failure of this skill; treat a question you could have answered by reading as a bug.
1. **Only ask about genuine unknowns** — decisions the source documents leave open, or trade-offs with no obvious default. For low-stakes micro-decisions (debounce timing, where a file lives within an established convention, test depth), decide with a stated recommendation and note it in Key Decisions rather than spending a user turn. Reserve questions for choices that materially change the design.
2. **Batch the questions.** Use the harness's structured question tool if it has one (on Claude Code, `AskUserQuestion` takes up to 4 multiple-choice questions per call); otherwise one consolidated message. Combine free-text questions into a single prompt. Goal: minimize user turns. Every turn adds reasoning overhead, so don't drip-feed one question per turn.
3. When options differ in shape, show each option's tree sketch as a text preview. Text trees, not Mermaid.
4. Update the design doc with resolved decisions
5. Move resolved questions from "Open Questions" to "Key Decisions" with rationale

If a decision made here is durable and cross-cutting — it will still govern work six months from now, not just this feature — and the project keeps decision records, say so and offer to record it there.

### Step 4: Present the Design

```
I've created a design discussion at:
`.stagecraft/design-YYYY-MM-DD-description.md`

This captures our alignment on:
- [Key point 1]
- [Key point 2]
- [Key point 3]

All open questions are resolved. When you're ready, run `/stagecraft plan` with this design to generate the implementation plan.
```

## Important Guidelines

1. **Target ~200 lines** — Simple features might be 80, complex ones 300. An order of magnitude shorter than a plan.
2. **Be opinionated about patterns** — Show the user which patterns you found and plan to follow. This is where they correct you.
3. **Testing approach is required** — Every design must address how we'll verify the work, using what **Commands** says the project actually has. Don't prescribe TDD if there's no test infrastructure.
4. **No open questions in final doc** — Walk through all questions interactively before finalizing.
5. **Read research fully** — The research doc is your primary input. Don't re-research what's already been done.
6. **Don't ask what the sources already answer** — The issue, its linked spec, the handoff comments, and the research doc's resolved decisions are authoritative. Derive scope and settled decisions from them; only bring genuinely open trade-offs to the user.
7. **Surface patterns to follow explicitly** — This prevents the agent from following the wrong patterns during implementation.
