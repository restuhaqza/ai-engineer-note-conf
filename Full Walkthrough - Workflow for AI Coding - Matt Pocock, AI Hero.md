# Full Walkthrough: Workflow for AI Coding

| Field | Details |
| --- | --- |
| **Speaker** | Matt Pocock |
| **Affiliation** | AI Hero |
| **Type** | Talk |
| **Date** | 2026-04-24 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=-QFHIoCo-Ko) |
| **Tags** | `architecture` `coding-and-developer-tools` `safety-and-governance` `agent-engineering` |

## Summary

Matt Pocock's thesis is that AI is a new paradigm, but software engineering fundamentals work just
as well with AI as with people. In this long, hands-on session he walks one ambiguous feature
request — "add gamification to Cadence," a course platform where students drop off — from idea all
the way to implementation and QA.

The constraint everything is built around is context. A model does its best work in a "smart zone"
at the start of a conversation and degrades in a "dumb zone" as tokens pile up (Pocock treats
roughly 100K tokens as the practical marker). So the workflow keeps tasks small, prefers clean
session resets over compaction, and delegates expensive exploration to subagents in separate windows.

His pipeline: grill a human until both sides share the same design concept; summarize the destination
in a PRD (product requirements document); break it into a Kanban board of vertical-slice issues; let
agents implement "AFK" (away from keyboard) with test-driven development and automated review; then
keep humans on QA, architecture, and taste.

The closing thread is that codebases shape agent quality. Deep modules — a small interface with
substantial behavior inside — are easier to test and easier to delegate, while bad codebases make bad
agents. Plans are working artifacts, not permanent truth; stale PRDs can mislead future agents, a
problem he calls doc rot.

## Key Takeaways

- **Treat context as a budget.** The smart zone is roughly the first ~100K tokens regardless of the
  advertised window; coding degrades before the window fills.
- **Prefer a clean reset to compaction.** A fresh session starts from a predictable system prompt,
  while compaction carries an imperfect summary forward.
- **Delegate exploration to subagents.** In the demo an Explore subagent burned about 94K tokens in
  its own window while the main session stayed small.
- **Alignment is human-in-the-loop and cannot be looped.** The tiny Grill Me skill interviews you one
  question at a time, each with a recommended answer.
- **Don't over-polish the PRD.** It captures the destination; the leverage is in alignment and QA,
  not in writing the perfect document.
- **Use tracer bullets, not horizontal layers.** Agents default to building the database, then the
  API, then the front end, which delays all feedback.
- **TDD keeps agents honest.** Writing a failing test before the implementation makes cheating
  harder, and good feedback loops set the ceiling on agent output.
- **Humans keep QA and architecture.** Automated review runs in a fresh context, but only a person
  catches taste and integration bugs.

## Detailed Notes

### Design the workflow around context limits and predictable resets · 00:48
- Pocock credits the **smart zone / dumb zone** idea to Dex Horthy: a model does its best work at
  the start of a conversation, when attention relationships are least strained.
- Every token added strains attention roughly quadratically — his analogy is adding a team to a
  football league, which adds many matches at once.
- Around 40% of a context, which he puts near **100K tokens**, quality starts dropping regardless of
  whether the window is 200K or one million. More window helps retrieval, less so coding.
- So size tasks to stay in the smart zone — old advice about not biting off more than you can chew
  (*Refactoring*, *The Pragmatic Programmer*) now applies to agents.
- Naive multi-phase plans are really just a loop; he reduces them to "phase N," echoing the **Ralph
  Wiggum** technique of describing the destination and then nudging repeatedly.
- Every session follows the same shape: a small system prompt, then exploration, implementation, and
  testing. Clearing context returns you to that base state.
- **Compaction** squeezes the conversation into a written history. He dislikes it because a reset is
  always the same, while a summary is not.
- He recommends a status line showing exact token usage, so you know how close you are to the dumb
  zone.
- **Subagents** run in isolated context windows and report a summary back; the demo's explore step
  spent ~93.7K Opus tokens while barely raising the main session's usage.

### Start with human alignment, then preserve the destination · 11:35
- The demo repo is **Cadence**, a course platform. A brief from client Sarah Chen says retention is
  weak — students do a few lessons and stop — so she wants gamification.
- Pocock argues against **specs to code**: iterating only on a spec while ignoring the code is vibe
  coding in disguise, and you need to keep a hand on the code.
- **Grill Me** is a deliberately tiny skill: interview me relentlessly until we reach a shared
  understanding, walk each branch of the design tree, ask one question at a time, and recommend an
  answer for each.
- The goal is a **design concept** — a phrase from Frederick P. Brooks's *The Design of Design* —
  meaning a shared idea between everyone involved, not a plan asset.
- Real questions the grilling surfaced: which actions earn points; skip video-watch events as noisy
  and gameable; should existing completions get points retroactively; the level curve; where the UI
  lives.
- Sessions can run long — he has seen 40, 80, even ~100 questions — and skills can be tuned if the
  interview feels too aggressive.
- The stage stays human-in-the-loop. If you can't answer, bring in a domain expert, another
  developer, or the team; meeting transcripts can be fed in to test assumptions you missed.
- The **PRD** is the destination document: problem statement, solution, user stories, implementation
  decisions, testing decisions, and out-of-scope items (which give a definition of done).
- He does not closely review the PRD — with a shared design concept, reading it mostly tests the
  model's summarization. He does check the proposed modules because they must fit the codebase (his
  workflow judgment, not a rule for every team).

### Turn requirements into vertical slices and dependency-aware work · 39:23
- A destination document is not an execution plan. He converts the PRD into a **Kanban board** of
  independently grabbable issues with explicit blocking relationships, rather than a numbered plan
  only one agent can work through.
- Issues are flagged **AFK** when they can run unattended, versus tasks that need a human in the loop.
- **Vertical slices**, also called **tracer bullets**, cross the layers needed to produce a small,
  testable behavior. The name comes from anti-aircraft rounds that glow so the gunner sees where
  shots land.
- Agents default to **horizontal** work — all the schema, then all the API, then the front end — and
  the wrongness shows only at the end, when you finally have an integrated system to test.
- In the demo the agent's first proposal created the schema and a gamification service with nothing
  user-facing. Pocock pushed back and got a slice that awards points for lesson completion and shows
  them on the dashboard.
- Those dependencies create a directed graph: some work starts immediately, other work waits on it,
  and independent branches become candidates for parallel agents.

### Automate implementation, not ownership of quality · 51:58
- The "day shift / night shift" split: humans plan and queue work; agents then work the backlog AFK.
  The human leaves the loop for implementation only.
- The simple **Ralph-style** script collects local issue files and the last five commits, then runs a
  coding agent with permission mode set to accept edits. The full AFK version loops and runs inside a
  **Docker** sandbox (an isolated container).
- He recommends running the single-iteration version repeatedly first, so you can watch behavior and
  tune the prompt before trusting a longer loop.
- The implementation prompt sets priorities — critical bug fixes, development infrastructure, tracer
  bullets, then polish, quick wins, and refactors — and tells the agent to explore the repo, use TDD,
  and run feedback loops.
- **TDD** is test-driven development, in the red-green-refactor cycle: write a failing test, make it
  pass, then clean up. Test-first makes it much harder for an agent to write the implementation and
  then wrap it in convenient tests.
- **Feedback loops** — tests and type checks — are the ceiling on agent output. Without them the
  agent is coding blind, and improving the loops improves the code.
- In the demo the agent created a gamification service, ran tests and type checks (fixing one type
  error), and produced a commit and summary; the repo ended with 284 tests.
- Automated review belongs in the pipeline but in a **fresh context**: reviewing at the end of a long
  session would put the reviewer in the dumb zone. He uses Sonnet to implement and Opus to review.
- For standards he distinguishes **push vs pull**: implementers pull relevant skills when needed;
  reviewers get the standards pushed directly so they can compare code against expectations.
- **QA is not automatable.** Tests and type checks passed, yet manually trying the feature revealed a
  missing `point events` database table. QA is where his taste enters the codebase.
- Front end is a special case: visual judgment needs human eyes, so he asks for a few throwaway
  prototypes rather than expecting the agent to nail a mature UI.
- Honest caveat: agents generate more code than people can comfortably review, and he has no full
  answer for keeping pull requests small while running loops.

### Shape the architecture agents inherit · 1:13:39
- **Bad codebases make bad agents.** Tangled dependencies and weak feedback loops make a system hard
  for an agent to understand, modify, and verify.
- **Shallow modules** are many small files exporting lots of things. Test boundaries are unclear, so
  AI tends to wrap every tiny function in its own brittle test and must trace dependency graphs by
  hand.
- **Deep modules**, a term from John Ousterhout's *A Philosophy of Software Design*, expose a small
  simple interface while containing substantial functionality. One test boundary around the module
  then tests meaningful behavior.
- Left unwatched, AI drifts toward shallow, horizontal codebases, which is why he keeps the module
  map in view while planning; the PRD lists the modules to modify and the ones to create.
- Interface design stays with humans; implementation can be delegated. He thinks of modules as gray
  boxes — he knows their shape and behavior, not every line.
- This matters because people with AI often work harder and understand their codebase less. Keeping
  the map is how you avoid handing your architecture to the agent.
- The **Improve Code Base Architecture** skill scans a repo and proposes clusters worth deepening. It
  flagged the quiz scoring service as having zero tests and helped wrap a browser-based video editor
  end-to-end in one testable module — "night and day" for the agent's ability to change it.

### Keep plans temporary: doc rot and stale PRDs · 1:23:16
- **Doc rot** is what happens when a finished PRD stays in the repo and later agents find it, even
  though names, file structure, and requirements have moved on. The stale document influences them
  badly.
- His preference is to remove it: in his GitHub-issues setup he closes the issue, so an agent can
  still fetch it but sees that it is done.
- Database migrations are a different case because they are a deterministic running record — though
  he admits the analogy is unresolved.
- He also warns against over-optimizing the PRD. The destination is a hint, the real investment is
  the grilling session, and the real work goes into QA.
- QA feeds the board: as implementation runs, reviewing it produces new issues that go back into the
  Kanban board.
- Team reality check: everything up to the PRD is shared work. Assets get argued over, prototypes and
  research may send the group back to the idea, and the loop keeps grinding until everyone knows
  where they are going.

### Scale only what remains coherent: Sandcastle · 1:29:50
- Parallel execution only pays off once boundaries, dependencies, and verification are clear.
- **Sandcastle** is his TypeScript library for AFK loops. It creates a git worktree per task (an
  isolated checkout on its own branch) and runs each agent inside a Docker container, so branches can
  be merged afterward.
- A **planner** looks at the backlog and picks a compatible set of issues; each issue gets an
  **implementer** on its own branch; **reviewers** inspect the commits; a **merger** agent merges the
  branches and resolves type and test fallout.
- The standards split repeats here: implementers pull, reviewers get standards pushed, Sonnet
  implements, and Opus reviews.
- His one big recommendation: read the pre-AI software engineering books. Decades of practice around
  decomposition, interfaces, testing, and review translate directly to working with agents.
