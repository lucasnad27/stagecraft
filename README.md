# Stagecraft

Research → design → plan → implement → review, as one skill you install once and run from any
coding agent. The workflow never changes per repo; a committed `STAGECRAFT.md` tells it how your
project builds, tests, tracks issues, and reviews.

Successor to [`research-plan-implement`](https://github.com/lucasnad27/claude-plugins), built on
HumanLayer's RPI.

## Install

```bash
npx skills add lucasnad27/stagecraft          # this repo, or -g for every repo
/plugin marketplace add lucasnad27/stagecraft # Claude Code marketplace
```

Then, in a project:

```
/stagecraft setup     # writes STAGECRAFT.md from what it finds
/stagecraft guide     # where am I, what's next
```

## Verbs

| Verb        | What it does                                                              |
| ----------- | ------------------------------------------------------------------------- |
| `setup`     | Detects the project and writes `STAGECRAFT.md`; upgrades an existing one |
| `research`  | Documents how the code works today, with parallel sub-agents where the harness has them |
| `design`    | Settles the approach through discussion, with a concrete reference artifact |
| `plan`      | Vertical phases with per-phase verification                               |
| `implement` | Executes phase by phase, pausing for manual checks                         |
| `review`    | Commits, opens the PR, lands a numbered review guide                       |
| `guide`     | Orientation and topic deep-dives                                          |

## Layout

```
skills/stagecraft/      the skill: SKILL.md router, reference/ per verb, agents/, scripts/
.claude-plugin/         Claude Code plugin and marketplace manifests
```

MIT.
