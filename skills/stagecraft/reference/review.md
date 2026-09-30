# Review

You are tasked with getting a change ready for review: commit any outstanding work, run the project's checks, open a pull request, and land a **review guide** that points human attention at what matters. You don't replace code review — you make it faster by saying where to look, anchored to the lines it's about.

File writes are for the PR body, the review payload, and any source fix the user explicitly asks for during a walkthrough — never for unprompted changes.

This is the final step of the research → design → plan → implement → review workflow. **Run it in a fresh context, not the one that implemented the change.** A reviewer that inherits the implementation transcript inherits its optimism.

## Project context

Read `STAGECRAFT.md` at the repo root first. This stage uses **Commands**, **Tracker**, **Forge**, **Commits**, **Constraints**, **Guidance → Review**, and **Integrations**. If the file is missing, say so once, work from `git log` and the tree, and suggest `/stagecraft setup` at the end.

## Initial Response

Determine the mode from `$ARGUMENTS`:

- **No arguments** → **Create mode.** Commit outstanding changes on the current branch, push, open a new PR, land the guide.
- **A PR number (e.g. `123`)** → **Update mode.** Leave existing commits alone; write the guide onto that PR. Use this when a PR was created outside this skill loop.
- **A branch name** → **Create mode** targeting that branch's diff against the default branch.

Flags, anywhere in `$ARGUMENTS`:

- `--no-stops` — keep the whole guide in the PR description; post no inline comments. This is the default when **Forge** isn't GitHub or `gh` isn't available.
- `--walk` / `--no-walk` — force or suppress the Step 8 walkthrough without asking.

If the mode is ambiguous, state your assumption and proceed; don't stall on a question you can answer from context.

## The review guide is a numbered list of stops

One artifact, two places it appears, and an optional third:

- **Inline review comments on the PR** — the detail, anchored to the line it's about. This is its home.
- **A numbered index in the PR description** — one line per stop, so the guide is legible without opening the diff.
- **A walkthrough with the author**, if **Integrations** names a tool for it — the same stops, narrated in their terminal.

A stop is a file, a line or range, a type, and **a claim to test**. Not a description of what the code does — the reviewer is looking at the code. What they can't see is what you want them to *decide*.

> ✅ "The claim: a downgrade means 'the capability set shrank', derived from `OFFICE_CAPABILITIES` rather than a hardcoded ranking — so adding a fourth role can't silently produce a wrong answer."
>
> ❌ "This function computes whether a role change is a downgrade."

**The diff sets the number, not a target.** A PR with two things worth deciding gets two stops. Most PRs land between 3 and 8; past 10 nothing is emphasized, and that is a ceiling, not a goal. There is no floor — a mechanical change with green tests can earn one stop, or none, and "this is a rubber stamp, here's the one line I'd look at" is a complete and successful guide. Manufacturing stops to reach a count is the most common way this skill fails: it buries the two that mattered and spends the author's afternoon.

Types carry intent:

- `issue` — you believe something is wrong
- `note` — verify this is correct
- `suggestion` — decide whether you accept this
- `yagni` — an abstraction, config, or layer that has one caller and could be inlined until it has two

Separate "verify this is correct" from "decide whether you accept this." Load-bearing conventions with nothing enforcing them, and gaps in test coverage, are judgment calls — that's where a human's time pays off.

## Process

### Step 1: Establish the diff and gather context

1. **Determine the default branch:** `git remote show origin | sed -n '/HEAD branch/s/.*: //p'`, falling back to `main`.

2. **Determine the diff source:**
   - Create mode, current branch: `git diff [default-branch]...HEAD`
   - Create mode, named branch: `git diff [default-branch]...[branch]`
   - Update mode: `gh pr diff [number]`

3. **Read the full diff.** It is the source of truth.

4. **Check for supporting context (all optional):**
   - A design doc (`.stagecraft/design-*.md`) matching the recent dates or the branch name — the *intent* behind the change, and any concrete reference artifact it links.
   - The plan (`.stagecraft/plan-*.md`) this branch implemented. Its per-phase `### Completion` blocks record what was actually built versus specified — deviations, waivers, anything left unproven. `.stagecraft/` is working state that dies with the worktree, so the PR is the only durable home for anything that outlives the merge.
   - Review metadata at **the plan's name with `plan-` swapped for `review-`**. If you found a plan, that path is deterministic — read it directly rather than globbing; fall back to matching recent dates only when there's no plan in hand. Its per-file Critical / Mechanical / Tests triage was written by the agent that wrote the code, so it picks stops far better than the diff can.
   - The issue the branch encodes, fetched with the **Tracker** command. Link it in the PR body the way **Forge** says to.
   - Any decision record the change implements or contradicts, if the project keeps them. A change that contradicts a binding decision needs a stop of its own, not a footnote.
   - These sharpen the guide, but it works from the diff alone — which is exactly the case when preparing a PR for someone else's branch.

5. **Sweep for deferred work — a backstop, not the mechanism.** Reading the plan is how you find deferred work; this catches the pieces that live somewhere else — a "does not prove" caveat in review metadata, a design doc's out-of-scope list, something you and the user agreed to punt during a walkthrough.

   ```bash
   grep -rniE "defer|smoke test|owed|can'?t be verified|until (it )?(merges|deploys)|follow-up (ticket|issue)|out of scope" \
     --include='plan-*.md' --include='design-*.md' --include='review-*.md' .stagecraft 2>/dev/null
   ```

   It matches on wording, so it misses phrasings nobody anticipated. A hit is a pointer to go read the surrounding paragraph; silence is not evidence that nothing was deferred.

### Step 2: Commit outstanding changes (create mode only)

Skip this entire step in update mode.

1. **Check the working tree:** `git status --short`.

2. **If there are uncommitted changes:**
   - Group them into a sensible commit, or a few if the work is clearly separable.
   - Run the verification commands from **Commands** first and report results — don't commit on top of a red build without flagging it. If the project has a separate format gate, run it and include the result in the commit.
   - Propose a commit message in the convention **Commits** records, matching its example. Confirm it still holds against `git log --no-merges --format=%s -10`, and follow the history if it has moved.
   - **Confirm the message with the user**, then commit.
   - Trailers follow **Commits**. Absent a rule there, never add a "Co-authored-by" or "Generated with" trailer unless the repo's own history shows that convention.

3. **If the tree is clean**, say so and move on.

**Don't push yet.** Pushing happens in Step 6, so the branch and its guide go up together.

### Step 3: Run the project's checks

**Guidance → Review → checks** lists what this project runs on every PR beyond the standard commands — a security audit, a compliance scan, a dependency check. Run each on the same diff, before the PR exists, so a blocking finding is caught before anything is pushed.

Their output has one home: a collapsed `<details>` section per check at the end of the PR description, named for the check. A **blocking finding** — whatever the check calls its highest severity — becomes an `issue` stop anchored to the offending line and a ⛔ line at the top of the description. It gets an anchor, not just a paragraph.

With nothing listed, skip this step.

### Step 4: Build the stop list

This is the work. Everything downstream is formatting.

1. **Size the review before picking stops.** Ask what could actually go wrong here, and answer honestly. A change that only moves values inside an established contract — a token swap, a copy edit, a dependency bump, a rename — is low-risk by construction, and its guide should be short enough to read in the PR description. A change to auth, money, data shape, or an external contract is not, however small the diff. **Green tests plus an established pattern is evidence for fewer stops, not a gap to write stops about.** Say which of these the PR is before you write anything; if it's the first, cap yourself at three and pick the best three.

2. **Pick the stops.** Start from the review metadata's "Needs careful review" entries; fall back to reading the diff. A stop is earned by a decision a reviewer has to make — not by a file being interesting, and not by your having something accurate to say about it. Business logic, auth/permissions/validation/secrets, a data model or migration, an external contract, a deviation from the plan: each of those is a decision. **Guidance → Review → always a stop** names what earns one in this project without exception. A `yagni` stop earns its place when the diff introduces an abstraction with one caller.

   **Missing test coverage is one stop for the whole PR, not one per file** — it belongs under **Not tested** in the description, and only becomes a stop when the untested thing is also the risky thing. The same goes for a fact you verified and found correct: that is a reason to stay quiet, not to write a stop asking someone to re-verify it. Pattern conformance is a `note` stop anchored to the line, not a standing section.

3. **Verify every anchor.** A stop's line number must fall inside a diff hunk, or GitHub rejects the whole payload in Step 7. Read the hunk headers — `@@ -old,+new @@` — from the same diff you're describing:

   ```bash
   git diff [default-branch]...HEAD -- <file>          # create mode: no PR exists yet
   gh api repos/{owner}/{repo}/pulls/<N>/files --jq '.[] | "\(.filename)\n\(.patch)"'   # update mode
   ```

   Confirm each stop's line falls in a new-side range. For a stop about a *deleted* line, use the old-side number with `side: "LEFT"`. A stop whose subject isn't in any hunk moves to the nearest in-hunk line, and says so in its text.

4. **Cut the list.** Read the stops back and delete any that a competent reviewer would reach on their own from the diff plus the description. Deleting down to two is a good outcome; deleting to zero and saying so in one line is also a good outcome. Then **record the list** in your scratchpad as `n`, `path`, `line` (or `start_line`–`line`), `side`, `type`, and the claim. Steps 5, 7, and 8 all render from it, and they must not drift apart.

### Step 5: Write the PR description

**Under 60 lines**, not counting collapsed check output. If it's longer, the detail belongs in a stop.

```markdown
## Summary
[≤5 sentences: what this change does, why, and the approach. Link the issue the way Forge says to.]

[Structural changes only: one tree diff, or a Mermaid sequence diagram if the forge
renders Mermaid, ≤12 lines. Reuse the design doc's shape diff, updated to what landed.]

## Review guide (N stops)
Each stop is an inline comment on the line it names.

1. `path/to/file.ext:52` — [the claim to test, one line]
2. `path/to/file.ext:118-140` — [the claim to test, one line]
...
The remaining N files are [generated migrations, import moves, copy tweaks].

[With nothing worth stopping on, drop this heading entirely and write one line under Summary —
"Mechanical: <what it is>. Green on <the check>. Nothing here needs a second opinion." An empty
or padded guide is worse than none.]

## Test coverage
**Tested:** [behavior] — `tests/path` · [behavior] — `tests/path`
**Not tested:** [behavior] — [why]

## Deferred
[Work this PR cannot verify before merge — cloud-only surfaces, anything needing a deployed
environment. When the plan has such a section, carry its wording verbatim rather than
summarizing. Omit this heading entirely when there is nothing deferred; an empty section
trains readers to skip it. Guidance → Review → PR body may rename this heading or add
sections; follow it.]

<details><summary>[Check name]</summary>
[Full output from each Step 3 check, one block per check.]
</details>
```

Everything else stays out. The mechanical files get **one line** at the end of the guide — not an inventory. A reviewer who wants the file list opens the Files tab. The numbering is the review order.

**With `--no-stops`**, expand each numbered index line into its full stop text under the same heading, drop the "Each stop is an inline comment" line, and skip Step 7. The description gets longer; nothing else changes.

### Step 6: Push and create the PR

Do this **before** posting the stops — inline comments need a PR to attach to.

**Create mode:**

1. Derive a concise PR title following the repo's convention.
2. Show the title and description to the user and confirm.
3. Apply any PR-creation steps **Guidance → Review → PR body** lists (a milestone to set, a label to add, a reviewer to request).
4. Push with upstream tracking: `git push -u origin HEAD`.
5. `gh pr create --title "..." --body-file <file>` against the default branch. Use `--body-file` to preserve formatting.
6. Return the PR URL.

**Update mode:** the PR exists — don't create anything. Push any local commits so the branch on GitHub matches the diff you described, then publish the description with `gh pr edit [number] --body-file <file>`. Leave the title alone unless it's clearly wrong; ask before changing it.

### Step 7: Post the stops as a review

Skip on `--no-stops`.

**In update mode, check for a guide you already posted** before adding a second one:

```bash
gh api repos/{owner}/{repo}/pulls/<N>/reviews --jq '.[] | "\(.id) \(.user.login) \(.state)"'
```

Write the payload, one entry per stop, in guide order:

```json
{
  "event": "COMMENT",
  "body": "Review guide — 7 stops, numbered in reading order. Each names a claim to test, not a description of the code.",
  "comments": [
    {
      "path": "src/hooks.server.ts",
      "line": 52,
      "side": "RIGHT",
      "body": "**STOP 1 · note** — The claim: the guard runs before session refresh, so a stale cookie can't reach the handler.\n\nWhat to verify: `refreshSession` is called on the branch below, not above.\nRisk: an expired session reads as valid for one request."
    },
    {
      "path": "src/lib/server/db/schema.ts",
      "start_line": 118,
      "line": 140,
      "side": "RIGHT",
      "body": "**STOP 2 · issue** — ..."
    }
  ]
}
```

**Confirm before posting** — the review lands on the PR under the authenticated account (`gh auth status`). Then:

```bash
gh api repos/{owner}/{repo}/pulls/<N>/reviews --input stops.json
```

Four things to know:

- **`event` is always `COMMENT`.** Never `APPROVE` or `REQUEST_CHANGES` — the guide is navigation, not a verdict, and it's the author's own PR.
- **`line` is the line in the file, on the side you name**, and must be inside a diff hunk. Ranges use `start_line`–`line`. This is what Step 4.3 verified.
- **A bad anchor rejects the whole payload with a 422** that names the offending comment. Fix that stop's line and re-post — but **check for a review that landed before you retry**, using the `reviews` call above. Never blind-retry.
- **Each stop becomes a resolvable thread.** That's the point: a reviewer ticks stops off as they go, and a walkthrough tool that renders review threads hides resolved ones too.

Report the review URL alongside the PR URL.

### Step 8: Walk the PR with the author (optional)

The guide you just posted is exactly the input a guided walkthrough needs — ordered stops, anchored to lines, each naming a claim to test. If **Integrations** names a walkthrough tool and someone is there to answer, project it into their terminal instead of leaving it in a browser tab.

**Read that tool's file under `reference/integrations/` and follow it.** The shape is the same whatever the tool: the stops are already on the PR as review threads, you narrate, they drive, and anything you find becomes a commit on the open PR before another reviewer is asked to look.

Before the first fix, know the baseline — test count and typecheck errors from **Commands**. The Test coverage section usually states it; if not, capture it now, because "nothing regressed" is only a claim you can make against a number you recorded.

Offer it once and wait:

```
PR is up: <url> — 7 stops posted as inline comments. Want to walk them?
I'll narrate each one and we can fix anything we find before anyone else looks.

Open the PR in your walkthrough tool in another pane, or say "skip" and it's ready for review as-is.
```

Skip straight to Step 9 if **Integrations** names no walkthrough tool, if the user declines, if the tool isn't on the PATH (say so once, don't belabor it), if the diff is trivially mechanical, or if you're running unattended.

### Step 9: Reconcile

Only when Step 8 produced commits. Otherwise you're done — return the PR URL.

1. **Recompute every line reference.** Walk commits shift them all. An index pointing at `:270` when the code now lives at `:306` is worse than no line number, and GitHub marks the moved threads outdated.
2. **Add the new commits to the story.** A short note near the top for anyone who reviewed the earlier push: what moved, and — just as important — what didn't.
3. **Fold findings into the right place.** A behavior change that came out of a fix belongs in the stop it came from (reply to that thread); anything the fix left unexercised belongs under **Not tested**.
4. **Write down what you agreed to defer** under **Deferred**, added here if the original description didn't need it.
5. **Re-run any Step 3 check** whose scope the walk touched. Otherwise the earlier output still stands.
6. **Re-publish the description** with `gh pr edit [number] --body-file <file>`.

## Important Guidelines

1. **Guide attention, don't find bugs.** This is navigation, not a verdict. The stop says what to decide; the human decides it.
2. **A short guide is a good guide.** The measure is whether the author's attention went to the right place, never how much you found. If the honest answer is "this is fine, merge it", say that in a sentence and stop — you are not being paid by the stop, and an author who has to argue their way out of ten stops on a green token swap will stop running this skill.
3. **A stop and its index line carry the same claim.** If a stop needs something the description doesn't say, the description is missing it — not the other way around.
4. **Be specific about lines.** "Review auth.ts" is useless. A stop anchored to `auth.ts:52` naming what could go wrong is the whole product.
5. **Mechanical doesn't mean unimportant** — it means it follows an established pattern and doesn't need creative review. It gets one line, not a section.
6. **Works without metadata.** With no design doc or review metadata, build the stop list from the diff. Less precise, still useful.
7. **Trust in-phase triage over reconstructed triage.** A metadata section headed `(reconstructed from the diff — not authored in-phase)` was written by an agent reading the diff, not the one that wrote the code. Check those claims against the diff before turning them into stops.
8. **The project's checks are not optional.** They run on every PR, including someone else's branch, and before anything is pushed.
9. **Confirm before outward-facing or irreversible actions** — committing, pushing, creating the PR, posting the review. The guide itself is cheap; these are not.
10. **The walkthrough edits source only on request.** When the user does ask for a fix, verify it against the pre-walk baseline, commit it separately, and push it so the session can see it.
11. **Match the repo's conventions** — commit style, PR title format, issue linking. **Commits** and **Forge** record them; read recent history when they're silent.
