# Ralph Loops: Build Dumb AI Loops That Ship

| Field | Details |
| --- | --- |
| **Speaker** | Chris Parsons |
| **Affiliation** | Cherrypick |
| **Type** | Workshop |
| **Date** | 2026-05-04 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=2TLXsxkz0zI) |
| **Tags** | `apis-mcp-and-protocols` `agent-engineering` `coding-and-developer-tools` `rag-context-and-search` `developer-workflows-and-testing` |

## Summary

Chris Parsons opens with a failed automation: an n8n workflow that built his newsletter and broke almost every Monday. Maintaining the machine that wrote the newsletter cost more than writing it by hand. So he pasted the workflow's exported JSON into a coding agent and asked for a skill instead — instructions the agent reads and then acts on, one step at a time.

That leads to the core idea. A **Ralph Loop** — named after the Simpsons character who keeps retrying the same thing — is an outer loop wrapped around a coding agent: ask for a feature, let the agent finish, then ask again. Parsons credits Geoffrey Huntley for the pattern, and Matt Pocock for the extension that matters most: point the loop at a backlog and tell it to pick the next most important ticket. The agent reads the repo and the backlog and works through them.

The through-line is that useful autonomy depends on feedback, durable context, permissions, and knowing when to stop. Parsons argues you should start with one continuously running agent, not a swarm, because the bottleneck is rarely generation speed. It is usually specifying work and reviewing it — a "theory of constraints" problem. He also ends on a personal note: deciding which work he wants to keep doing himself, rather than reviewing an endless stream of AI drafts.

## Key Takeaways

- **A Ralph Loop is just "do the thing, then ask again."** The agent often catches a missed detail on the second pass, or notices that the ticket it implemented was never marked done.
- **Point the loop at a backlog.** Ask for "the next most important ticket" and let the agent infer dependencies from current state instead of building a full execution graph upfront.
- **Start with one agent.** More parallelism just queues work behind the human who specifies and reviews; Parsons has not needed orchestration tools.
- **The loop only checks what its instructions ask it to check.** A one-line prompt leaves testing, completion, and shipping ambiguous — write a real delivery contract.
- **Use fresh-context subagents for review.** Inline self-review tends to approve the implementer's own choices.
- **Give unattended agents boundaries.** A separate machine, read-only tools, dedicated credentials, and selected permissions — not skipping every permission.
- **Passing tests is not enough.** When the agent writes both code and tests, both can encode the same misunderstanding of the feature.
- **The constraint is review, not generation.** If release is the limiting step, faster coding just piles up work in the queue.

## Detailed Notes

### From brittle orchestration to a simple loop · 03:47
- Parsons's n8n newsletter workflow failed most Mondays; each failure meant reopening the graph, finding the broken step, and fixing it.
- n8n itself is fine and convenient for API keys, but at this complexity the workflow was brittle, so fixing it cost more than just writing the newsletter.
- He replaced it in one move: paste the exported workflow JSON into a coding agent and ask for a skill based on the flow.
- The agent preserved the instructions but now owns the sequencing: read the relevant instruction, call a tool, inspect the result, continue.
- Claude Code is itself a loop — read, decide, act, repeat — so wrapping it is natural.
- He attributes the shift to more capable recent models (he names the GPT-5.x and Claude Opus/Sonnet 4.6 families). His remarks about Mythos are speculation, not experience.
- The second feedback cycle: after a writing session he asks the agent to update the skill with anything it should have done differently.

### The minimal Ralph loop: repeat the request · 08:26
- The original idea, credited to Geoffrey Huntley: wrap the agent's own tool loop in an outer loop.
- Mechanism: build a feature, let the agent announce it is done, then send the same prompt again — a second look often finds an unfinished detail.
- The demo uses a deliberately incomplete Python Pomodoro timer with one test and a Markdown ticket backlog.
- Ticket 001 asks for a status command showing time remaining; the first run adds the command plus a new test.
- Repeating seems pointless — Opus notices the feature already exists — but a later pass marks the ticket itself done. Code and tracker had drifted.
- The dumbest loop is a shell one-liner that reruns the agent in noninteractive print mode, e.g. `while true; do claude -p 'Implement ticket 001 from doc/tickets.'; done`.
- That bare loop has no stop condition, iteration budget, or permission setup. It is a starting point, not something to run in production.
- Clearing context and re-asking separates two styles: resubmitting within a session, or starting the coding agent fresh from an external loop.

### Let the loop choose the next ticket · 22:07
- Parsons credits a Matt Pocock video for the shift: stop re-checking one feature and instead let the loop advance through the backlog.
- His earlier attempt went the wrong way: decompose a project into tickets, sub-tickets, and an explicit dependency graph, then run parallel agents.
- Two agents reached the same shared prerequisite, both implemented it, and duplicated the work. Tracking what was actually done became a coordination problem of its own.
- He saw the resemblance to heavyweight waterfall planning: an elaborate spec handed over before implementation had taught anyone what needed to change.
- The fix is simple: "implement the next most important ticket using TDD principles, commit when done." The agent reads the repo and backlog and infers satisfied dependencies on the fly.
- Start with one continuously running agent; adding orchestration (for example Gas Town) just queues work behind the same human bottleneck.
- Session isolation and repository isolation are separate concerns: a fresh session still sees the dirty working tree, so the commit was not atomic. Atomic commits need an explicit policy.

### Make unattended work observable and safe · 31:35
- The headless demo runs blind because streaming output is not configured — progress and control are poor, and Parsons says he would not use that wrapper as-is.
- An unattended loop needs permissions that let its intended operations proceed; he enables selected actions rather than skipping all checks.
- He introduces **Lockbox**, his attempt to cut filesystem access once untrusted content enters an agent's context — an experiment, not a demonstrated guarantee.
- Outside the toy repo he combines boundaries: most work on a VPS (virtual private server) away from his main machine, read-only developer tools, dedicated credentials for an audit trail, and selected Claude permissions.
- **Docker Sandboxes** offer local isolation, but filesystem isolation alone does not stop data leaking between services the agent can reach.
- The **lethal trifecta** (Simon Willison's phrase): private data plus untrusted content plus the ability to communicate externally creates a data-theft path. Reduce how often those three coexist.
- He is critical of OpenClaw's default security while acknowledging ongoing improvements.

### A skill is a delivery contract · 47:51
- "The loop checks what its instructions ask it to check." A one-line request leaves testing, completion, and shipping ambiguous.
- His full Ralph skill casts the agent as one engineer in a relay team: make exactly one change, stop, and let the next session start with fresh context.
- Because context resets, the files are the handoff between workers: read `CLAUDE.md`, understand ticket format and status values, and inspect Git state before starting.
- Interrupted work needs a recovery policy: a dirty tree with passing tests may be finished but unrecorded (inspect and verify); a dirty tree with failing tests is likely partial work (diagnose first); passing tests alone still means checking actual behavior.
- The rules grew through use and add project-specific test commands, parallel checks, completion updates, handoff to testers, and a task-selection policy.
- He recommends starting from a template and adapting it — "fix this for my project" — rather than copying his skill verbatim.

### Fresh reviewers and just-in-time specs · 55:19
- An audience member found that defects started appearing once validation moved into subagents; validating in the same context mostly approved the existing work.
- The reason: a subagent starts with only a small slice of context, so it does not inherit the implementer's account of why its choices were correct.
- The bundled `simplify` skill runs three subagents over recent changes; in the demo it found issues to fix.
- On spec-driven development, Parsons supports thinking a change through and planning it, but prefers planning just in time, when implementation can still inform the next decision.
- His concern is that rigid tools freeze a workflow tuned to today's model limits and preserve it after those limits change.

### Run your work as loops · 1:02:39
- Claude Code's `/loop` command schedules repeated work inside the session — for example, "build the next ticket every minute" — using a cron-style schedule.
- The pattern generalizes: an hourly loop can check a ticket tracker (such as Linear) for new bugs, and a team-specific skill can encode what happens next (claim, implement, open a PR, respond to review, merge, release).
- He runs personal operations the same way: a 15-minute VPS heartbeat that checks calendar and state and sends Telegram updates, a 6 a.m. daily briefing, and a personal Kanban app where a worker performs the next step on a project file.
- His permission rule is whether an action is "reversible without embarrassment": preparing slides and drafting replies are fine; sending emails or posting publicly are not.
- He deliberately keeps strategic thinking for himself, asking the system to gather information rather than produce endless drafts for him to review.
- Different jobs get different loop files with their own stopping rules: stop when context runs low or the next action is irreversible, then update the project file with progress and the decision awaiting review.
- Durable knowledge lives outside the session. His vault is an Obsidian-based collection of thousands of Markdown notes in a Zettelkasten "one thought per note" style, alongside a code directory and a command-line embeddings tool for retrieval.

### The constraint is review, not generation · 1:32:09
- Team coordination starts with ticket ownership: connect the agent to the tracker, check whether someone already owns the work, and claim the ticket before implementing.
- Faster implementation does not help if review or release is the limiting step (the **theory of constraints**). A monthly release growing from 20 PRs to 200 without more release capacity is the failure mode; he points to Goldratt's *The Goal*.
- Repository layout (one monorepo vs many) matters less than ownership and release coordination; cross-service release dependencies are the real question.
- Passing tests is necessary but not sufficient: when the agent writes both the implementation and the tests, both can encode the same misunderstanding of the requested feature.
- He limits unattended shipping to pre-release projects and still reads consequential diffs, especially where customer data and security are involved, and checks production database migrations before running them.
- Agent-driven UI click-through checks work about 50% of the time in his experience (he gives no denominator); maintained end-to-end tests are more concrete evidence.
- More agents can mean different perspectives, not just speed: his "simulate audience" skill has independent personas review a draft in parallel, and different models can add a contrarian view.
- The review queue does not disappear. Better automated critique helps, but humans remain the final gate — and he questions whether hiring people just to absorb the tedious leftovers is the right answer.
