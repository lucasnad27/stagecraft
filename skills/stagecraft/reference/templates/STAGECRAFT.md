# Stagecraft

<!-- stagecraft: [version] · written [YYYY-MM-DD] -->

[This file is the only thing about the workflow that is specific to this repo. Every stage
reads it at runtime. Keep it readable in one sitting; a sentence in the right slot beats a
paragraph. Delete any bracketed hint once its slot is filled, and drop a slot that has nothing
in it.]

## Project

[Two or three sentences: language, framework, package manager, database, hosting. Where the
project's own conventions live (AGENTS.md, CONTRIBUTING.md).]

| Path | Holds |
| ---- | ----- |
| `src/...` | [what lives there] |

[Directories to skip when searching: build output, vendored code, generated files.]

## Commands

| Purpose   | Command |
| --------- | ------- |
| test      | `...`   |
| lint      | `...`   |
| format    | `...`   |
| typecheck | `...`   |
| build     | `...`   |
| dev       | `...`   |
| database  | `...`   |

[The order a phase should run them, anything slow enough to run once per phase, anything that
only works in CI. Test infrastructure by kind: unit, integration, e2e, and where each lives.]

## Tracker

[System and key shape (`PROJ-123`, `#123`, none). The fetch command or tool. Branch and
worktree naming, if the key is encoded there. What a bare number means if it isn't a key.]

## Forge

[GitHub, GitLab, other. Owner/repo. Permalink form. Whether inline review comments are
available. Anything unusual: a mirror, a policy on closing keywords.]

## Commits

[Convention, with one real example from this repo's history. Scopes in use. Trailer rules.
Anything that must never appear in a commit or PR body.]

## Artifacts

`.stagecraft/`, gitignored, per worktree. [Issue key shape in filenames. Where the tracked
record lives, if the project keeps one, and that it is never this feature's design doc.]

## Constraints

[Rules every stage applies without being asked, because they fail late: a schema that isn't
the deployment path, an env var that inlines at build time, a design tool that is the source
of truth for UI, a dev server the user has to start. One bullet each, with the command or file
that proves it and a pointer to the decision that made it a rule.]

## Guidance

### Research

[Extra sources beyond the code: decision records, reference docs, package READMEs, and which
project agent covers them. Extra sections the research doc should carry. Where a feature's
surface area usually starts in this codebase.]

### Design

[Decision records to check first. Patterns to name. Where UI reference artifacts live.
Testing options by kind of work. Forks worth stating explicitly in this codebase.]

### Plan

[The reuse ladder: where a "new" helper most often already exists. Changes that get a phase
of their own. Sections this project adds to the plan template.]

### Implement

[How manual verification is done: personas, seeded environments, the command that prints
credentials. A comment example in this project's language. Known ceilings worth a Completion
note.]

### Review

[Checks to run before the PR exists, with the name of their blocking severity. What always
earns a stop in this repo. PR body: issue link form, renamed or extra sections, steps on
create (milestone, labels, reviewers). Walkthrough specifics.]

### Guide

[A project-specific orientation tip, if one applies.]

## Integrations

[Optional tools this project has opted into, one per line, by the name the skill knows them
by: `herdr`, `tuicr`. Omit the section if there are none; nothing runs unless listed here.]

## Agents

[Project-owned agent prompts, one bullet each: name, file path (`docs/stagecraft/agents/<name>.md`),
what it does, which stage spawns it. Omit the section if there are none.]
