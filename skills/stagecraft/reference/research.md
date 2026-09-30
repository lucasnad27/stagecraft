# Research

You are tasked with conducting comprehensive research across the codebase to answer user questions by spawning parallel sub-agents and synthesizing their findings.

**Your job is to document and explain the codebase as it exists today** — what exists, where it lives, how it works, and how components interact. You are creating a technical map of the existing system. Improvements, critiques, root cause analysis, and refactoring proposals are out of scope unless the user explicitly asks for them.

## Project context

Read `STAGECRAFT.md` at the repo root first. This stage uses **Project** (layout, search excludes), **Tracker** (how to fetch an issue), **Forge** (permalinks), **Guidance → Research**, and **Agents**. If the file is missing, say so once, work from what `git` and the tree show, and suggest `/stagecraft setup` at the end.

## Initial Setup

Check whether the user gave you something to research (a question, an issue reference, or a file in `$ARGUMENTS`):

**If the user provided a research question or issue**, read any mentioned files (see step 1) and proceed.

**If the user provided nothing**, don't immediately ask them to type a question — they're almost always on a branch cut for a specific issue, and that issue is the thing they want researched. Try to detect it first:

1. Spawn the **branch-ticket-detector** agent. It reads the branch and worktree, extracts an issue key in the shape **Tracker** describes, and fetches the issue.
2. Branch on what it returns:
   - **Issue found** — confirm before committing to it, since a wrong guess wastes a full research pass:
     ```txt
     It looks like you're working on PROJ-1478 — "Add SSO support to the login flow" (detected from your branch). Want me to research the codebase against this issue? (yes / or tell me what to research instead)
     ```
     On confirmation, treat the fetched issue body as the research input and continue (it goes through query planning in step 2 like any other issue).
   - **Nothing found, ambiguous, or fetch failed** — fall back to asking:
     ```txt
     I'm ready to research the codebase. Please provide your research question or area of interest, and I'll analyze it thoroughly by exploring relevant components and connections.
     ```
     Then wait for the user's research query.

Detection is a convenience, not a constraint — the user can always override the detected issue with their own question.

## Steps to follow after receiving the research query

1. **Read any directly mentioned files first:**

   - If the user mentions specific files (issues, docs, JSON), read them FULLY first
   - **IMPORTANT**: read entire files, not excerpts
   - **CRITICAL**: Read these files yourself in the main context before spawning any sub-tasks — this ensures you have full context before decomposing the research

2. **Generate research questions via query planning:**

   - If you have an issue or task description — whether the user provided it or the **branch-ticket-detector** fetched it during Initial Setup (not just a bare question) — use a **query-planner** agent to decompose it into focused, objective research questions
   - The query-planner reads the issue and generates 3-8 specific questions that will cause research agents to explore all relevant code
   - **CRITICAL**: The query-planner strips out information about what is being built — research agents receive ONLY the questions, not the original issue. This keeps research findings objective and factual.
   - If the user provided a direct research question (e.g., "How does authentication work?"), skip query planning and use the question directly
   - Track the sub-tasks so nothing is dropped

3. **Spawn parallel sub-agent tasks for comprehensive research:**

   - **Fan out across research questions** — when you have N distinct questions or areas to investigate, spawn N sub-agents in a single response so they run concurrently. This is the whole point of the research phase: parallel exploration, not sequential drilling. On a harness with no sub-agents, run the same prompts inline from `reference/degraded/` and say so once.
   - **When to spawn vs. reason directly**: spawn sub-agents for file discovery, cross-file tracing, and pattern surveys — work that would flood the main context with search output. Reason directly only when a single targeted read answers the question.
   - **IMPORTANT**: If query planning was used, pass ONLY the research questions to agents — do NOT include the original issue or task description
   - Give every agent the **Project** layout and search excludes from `STAGECRAFT.md`.

   **For codebase research:**

   - **codebase-locator** — find WHERE files and components live
   - **codebase-analyzer** — understand HOW specific code works
   - **codebase-pattern-finder** — find examples of existing patterns

   **For prior work under `.stagecraft/`:**

   - **artifact-locator** — discover what documents exist about the topic
   - **artifact-analyzer** — extract key insights from specific documents (only the most relevant ones)

   **For the project's own sources:** **Guidance → Research → extra sources** names tracked material outside `.stagecraft/` (decision records, reference docs, package READMEs) and **Agents** names any project agents that cover it. Spawn those alongside the codebase agents. When a committed decision and the code disagree, report both and say which you verified; the gap is itself a finding.

   **For web research (only if user explicitly asks):**

   - **web-search-researcher** — external documentation and resources. Instruct it to return LINKS with its findings, and include those links in your final report.

   **For issue context (if relevant):** fetch related issues yourself with the **Tracker** fetch command — a single lookup doesn't need a sub-agent.

   Use these agents intelligently: start with locator agents to find what exists, then use analyzer agents on the most promising findings. Run multiple agents in parallel when they're searching for different things. Each agent knows its job — tell it what you're looking for, not how to search.

4. **Wait for all sub-agents to complete and synthesize findings:**

   - Wait for ALL sub-agent tasks to complete before proceeding
   - Compile all sub-agent results (codebase, project sources, and artifact findings)
   - Prioritize live codebase findings as primary source of truth; project sources supply committed decisions and `.stagecraft/` supplies historical working notes
   - Connect findings across different components, with specific file paths and line numbers
   - Verify all artifact paths are within the current repo's `.stagecraft/` tree
   - Highlight patterns, connections, and architectural decisions
   - Apply anything else **Guidance → Research** asks for in the report
   - Answer the user's specific questions with concrete evidence

5. **Gather metadata for the research document:**

   - Identify the date, current commit hash, current branch name, and topic

   - Filename: `.stagecraft/research-YYYY-MM-DD-[KEY-]description.md`
     - `research-` is the type prefix, and it's what distinguishes this file from the design and plan that follow it
     - YYYY-MM-DD is today's date
     - KEY is the issue key in the shape **Artifacts** describes (omit if no issue)
     - description is a brief kebab-case description of the research topic
     - Examples: `.stagecraft/research-2025-01-08-PROJ-1478-parent-child-tracking.md`, `.stagecraft/research-2025-01-08-authentication-flow.md`

   Gather this before writing the document — never write it with placeholder values.

6. **Generate research document:**

   - Structure the document with YAML frontmatter followed by content:

     ```markdown
     ---
     date: [Current date and time with timezone in ISO format]
     git_commit: [Current commit hash]
     branch: [Current branch name]
     repository: [Repository name]
     topic: "[User's Question/Topic]"
     ---

     # Research: [User's Question/Topic]

     **Date**: [Current date and time with timezone from step 5]
     **Git Commit**: [Current commit hash from step 5]
     **Branch**: [Current branch name from step 5]

     ## Research Question

     [Original user query]

     ## Summary

     [High-level documentation of what was found, answering the user's question by describing what exists]

     [A call tree or shallow file tree of the main flow as it exists, with real paths. No diff; changes belong to /stagecraft design.]

     ## Detailed Findings

     ### [Component/Area 1]

     - Description of what exists ([file.ext:line](link))
     - How it connects to other components
     - Current implementation details

     ### [Component/Area 2]

     ...

     ## Code References

     - `path/to/file.py:123` - Description of what's there
     - `another/file.ts:45-67` - Description of the code block

     ## Architecture Documentation

     [Current patterns, conventions, and design implementations found in the codebase]

     [Any section Guidance → Research asks for, such as committed decisions from the project's own sources, goes here with references]

     ## Historical Context (from .stagecraft/)

     [Relevant insights from the .stagecraft/ directory with references]

     - `.stagecraft/design-2025-11-02-rate-limits.md` - Historical decision about X
     - `.stagecraft/research-2025-09-18-auth-refactor.md` - Past exploration of Y

     ## Related Research

     [Links to other research documents in .stagecraft/]

     ## Open Questions

     [Any areas that need further investigation]
     ```

   Keep frontmatter fields consistent across research documents, and use snake_case for multi-word field names.

7. **Add permalinks (if applicable):**

   - Check if on the default branch or if the commit is pushed: `git branch --show-current` and `git status`
   - If so, build permalinks in the form **Forge** gives (for GitHub: `https://github.com/{owner}/{repo}/blob/{commit}/{file}#L{line}`)
   - Replace local file references with permalinks in the document

8. **Present findings:**

   - Present a concise summary of findings to the user, starting with the Summary's tree
   - Include the path to the research document, and steer toward the open questions:
     ```
     Research doc: `.stagecraft/research-YYYY-MM-DD-description.md`. Scan over it to make sure you and I are aligned before moving to design — often, running through the open questions is all you really need to do.

     Want to go through them now?
     ```
   - The open questions are the point of this step, not the document. Don't ask the user to
     read it thoroughly or wait for them to finish reviewing — offer the questions directly.
   - Walk them interactively, batched into as few turns as possible; resolving them is what unblocks design
   - **Next step in the workflow**: once the open questions are resolved, the next step is
     `/stagecraft design` — not `/stagecraft plan` and not direct implementation. Research feeds a
     design discussion for alignment before any plan exists. Recommend it explicitly:
     ```
     When you're ready, run `/stagecraft design` with this research doc to align on the approach
     before we plan the implementation.
     ```
     Do not offer to create an implementation plan or implement directly from the research doc.

9. **Handle follow-up questions:**
   - Add `last_updated_note: "Added follow-up research for [brief description]"` to frontmatter
   - Add a new section: `## Follow-up Research [timestamp]`
   - Spawn new sub-agents as needed for additional investigation
   - Continue updating the document

## Important notes

- Always run fresh codebase research — never rely solely on existing documents. Project sources and `.stagecraft/` supplement live findings; they don't replace them.
- Research documents should be self-contained, with concrete file paths and line numbers, and permalinks where possible
- Keep the main agent focused on synthesis, not deep file reading — that's what the sub-agents are for
- Explore all of `.stagecraft/`, not just the `research-*.md` files
- **Path handling**: only reference paths within the current repo's `.stagecraft/` directory. `.stagecraft/` is hidden, so an unscoped search skips it — name it explicitly. Never broaden searches to parent dirs, sibling worktrees, or home-directory paths.
