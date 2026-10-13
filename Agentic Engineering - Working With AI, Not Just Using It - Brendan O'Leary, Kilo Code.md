# Agentic Engineering: Working With AI, Not Just Using It

| Field | Details |
| --- | --- |
| **Speaker** | Brendan O'Leary |
| **Affiliation** | Kilo Code |
| **Type** | Talk |
| **Date** | 2026-04-07 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=BEKc4P87XKo) |
| **Tags** | `agent-engineering` `apis-mcp-and-protocols` `developer-workflows-and-testing` `coding-and-developer-tools` |

## Summary

Most engineers now use AI coding tools, but few can explain their actual workflow: what they hand off, what they keep, and how they decide between the two. Brendan O'Leary argues that the shift from autocomplete to agents means we are now *working with* AI rather than simply *using* it. His mental model is a fast, endlessly enthusiastic junior developer who has read everything but has no judgment.

The core skill he teaches is **context engineering** — filling the model's working memory (the context window) with only what the next step needs. Bloated context costs money and can make the model less reliable; outdated or contradictory context can actively poison results.

His recommended loop is **research–plan–implement**. Research in a read-only mode, agree on a written plan, then implement in a fresh session and review the diff with Git. He rounds this out with agent modes, a lean `AGENTS.md`, task-specific skills, and careful use of MCP servers.

The talk ends with a simple prescription: pick a tool, get repetitions, and calibrate your trust through experience.

## Key Takeaways

- **Automation is not a workflow.** Being able to explain what you delegate and what you keep is what separates useful collaboration from auto-completing through the day.
- **Treat the agent as a fallible junior developer.** It is fast and broad, but it does not know your business context or why old decisions were made.
- **Technically correct code can be contextually wrong.** Supplying the "why" is part of directing the work, not an optional extra.
- **Manage context deliberately.** Persist info outside the window, select only what is needed, compress long investigations, and isolate unrelated tasks.
- **Start a new session when things go off track.** Ask the current agent to summarize, review that summary, then hand it to a fresh session.
- **Research first.** A small mistake in research can turn into a large amount of bad code, so front-load your human thinking.
- **Use Git as your first review.** Review the working-tree diff and stage intended hunks before committing or opening a PR.
- **Configure the agent and get reps.** Modes, `AGENTS.md`, skills, and MCP servers all shape behavior; trust comes from practice.

## Detailed Notes

### From autocomplete to agentic engineering · 00:00
- O'Leary opens with a question: can you really explain how you use AI at work, beyond "it makes me code faster?"
- He estimates that around 90% of engineers have used AI tools, with maybe half using them regularly. (His estimate, not a cited survey.)
- The unit of delegation has grown over time: from a line of code, to a whole function, to an entire task with file changes and tests.
- Citing Flask creator Armin Ronacher, he says we now *work with* machines rather than *use* them. A hammer is used and put down; an agent carries work forward between your interventions.
- The playful image: it is like an engineer who has read every Stack Overflow answer ever written — broad knowledge, but not your judgment.

### Treat the agent as a fast, well-read junior · 02:57
- The mental model to carry forward: an energetic, extremely well-read junior developer who can be confidently wrong.
- It is fast, rarely tires, and has no ego about rewriting code six times.
- What it lacks is judgment. It may not know a business constraint or why the team chose an architecture months ago.
- **Technically correct code can still be contextually wrong** — that gap is where your direction matters.
- O'Leary attributes to Ronacher a reported gain of more than 30% of his day, earned by knowing what to hand off and what to keep. No measurement method is given.
- The valuable habit is selective delegation plus direction and review, not accepting every suggestion.

### Context is a liability you must manage · 04:28
- Context engineering, borrowing Andrej Karpathy's framing, means filling the context window with just what the agent needs for its next step.
- Because chat history is resent as input tokens each turn, accumulating context adds recurring cost — not a one-time expense.
- More context is not always better. O'Leary uses roughly 50% window occupancy as a warning point for declining quality (his heuristic, not a proven threshold).
- Enabled MCP servers are a common culprit: each one can add tool descriptions even when irrelevant to the task, such as a Postgres connection during frontend-only work.
- Bad context is its own problem: mixing unrelated tasks, stale comments, or abandoned decisions can let rejected patterns creep back in.
- When repeated steering cannot fix an off-track session, start a new one so the failed path stops riding along in every request.

### Four moves: persist, select, compress, isolate · 07:11
- **Persist:** keep durable information outside the conversation — scratchpads, memory files, and `AGENTS.md`.
- **Select:** pull in only what the current step needs, using deliberate file references and disabling unneeded MCP servers.
- **Compress:** after a long debugging investigation identifies the problem and solution, summarize the findings instead of carrying every turn.
- **Isolate:** split unrelated tasks into separate sessions or agents so their contexts do not merge. He links this to the rise of parallel-agent workflows.
- These four operations keep useful knowledge while controlling what participates in the next request.
- An earlier management story makes the point: at a healthcare software company, interns built a wireframe prototype with Comic Sans and placeholder graphics intact. O'Leary blames himself for not explaining which details were placeholders. An agent handoff needs the same distinction.

### Research–plan–implement · 11:19
- The recurring mistake is asking an agent to implement a feature before anyone has established how it fits the existing system.
- O'Leary rejects the "prompt and watch code stream past" demo because speed can hide incorrect assumptions.
- Citing Dex Horthy, he warns that "a bad line of research can potentially be hundreds of lines of bad code." Review belongs upstream.
- **Research:** use a mode that cannot edit (Kilo's `Ask` mode). Identify current behavior, relevant files, existing patterns, data flow, and edge cases, then produce a research document you agree with. That document is the first review gate.
- **Plan:** name the files to change, describe the steps and effects, state what is in and out of scope, list tests and verification commands, and record it all in a plan file (often in a `Plans` directory) plus a rollback strategy.
- A clear plan lets a smaller or cheaper model do the implementation, because the reasoning is concentrated where a human can inspect it.

### Review with Git, configure roles and rules · 16:11
- Implement in a **fresh session** seeded with the reviewed plan, keeping the coding context small.
- Treat local Git review as your first PR review with the agent. A concrete sequence: `git status --short`, `git diff --stat`, `git diff`, then `git add -p` to stage only intended hunks, then `git diff --cached`. Run the plan's verification commands before committing.
- **The highest-leverage human work is research and planning.** By implementation time the hard decisions should already be explicit.
- Configuration maps to roles: `Ask` for research, `Architect` for planning, `Code` for implementation.
- Three layers serve different needs: **modes** define the role; **`AGENTS.md`** supplies always-on project conventions, build/test commands, and pre-commit checks; **skills** are on-demand playbooks for recurring procedures.
- Decide autonomy deliberately: can the agent read files, read outside the workspace, or run tests? Start with permissions you understand, then revisit them. Mention Dex Horthy's point that AI "can't replace thinking" — it only amplifies what you have, or have not, thought through.

### Tools, MCP servers, and getting repetitions · 22:47
- Use `@` mentions for files, commits, and terminal output; slash commands to start tasks or condense context; selected code added to the agent; and autocomplete for prompts as well as code.
- These agents are spreading beyond the IDE to the CLI, mobile, cloud, and Slack, so they can participate where work is discussed.
- **MCP (Model Context Protocol, a standard way for agents to call tools)** exposes extra capabilities. GitHub's MCP server adds pull-request, comment, and issue access; Context7 supplies current framework docs past the model's training cutoff.
- Watch the context cost: exposed tool descriptions can become recurring prompt input. Disable integrations a task does not use, such as a Postgres MCP during frontend work.
- For internal platform APIs, choose by need: use an existing OpenAPI/Swagger spec; store Markdown in the repo; fetch a reference URL for fast-changing info; or build a custom MCP server when a workflow spans multiple steps and systems.
- Isolate your changes from the agent's work and review its contribution like a junior engineer's PR. Then pick a tool, accumulate reps, and retry the research–plan–implement loop. Senior engineers, he notes, report having more fun again.

## Resources

- [Agentic Engineering for Humans](https://path.kilo.ai/) — workflow guides and patterns
- [AGENTS.md](https://agents.md/) — project instruction-file convention
- [Agent Skills specification](https://agentskills.io/specification) — defines `SKILL.md` and on-demand loading
- [GitHub MCP Server](https://github.com/github/github-mcp-server) — repo, issue, and PR tools
- [Context7](https://github.com/upstash/context7) — up-to-date library documentation
- [Kilo Code](https://kilo.ai/) — the agent tool used in the examples
