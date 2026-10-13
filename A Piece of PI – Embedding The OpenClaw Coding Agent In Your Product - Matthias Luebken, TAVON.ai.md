# A Piece of PI – Embedding The OpenClaw Coding Agent In Your Product

| Field | Details |
| --- | --- |
| **Speaker** | Matthias Luebken |
| **Affiliation** | TAVON.ai |
| **Type** | Talk |
| **Date** | 2026-05-11 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=vAIDdLKB6-w) |
| **Tags** | `agent-engineering` `enterprise` `coding-and-developer-tools` `architecture` |

## Summary

Matthias Luebken, founder of TAVON.ai, asks a simple question: what is actually
happening inside a coding agent when it appears to gain a new skill? His answer is
**Pi**, an open-source and deliberately minimal coding agent, which he uses to
build products where users never see a coding-agent chat at all.

He starts with the smallest building block. An agent is just a language model
calling tools in a loop. Pi's TypeScript agent class exposes that loop, an event
stream, and hooks where an application can step in before a tool runs. Add a
shell, and the core agent becomes a coding agent.

Luebken then pushes the idea toward a real product. For a client, he built an
email-driven sales assistant: a monitored inbox routes messages to a per-customer
agent, which uses command-line tools to read CRM and ERP systems and produce a
draft reply. The user stays in email and drafts while the agent machinery runs
underneath.

## Key Takeaways

- **Coding agents are reusable building blocks.** Luebken bets they will sit
  behind many products, not just developer tools.
- **Good products compose small tools.** Cowork's spreadsheet skill uses pandas,
  openpyxl, and LibreOffice instead of driving Excel directly.
- **Design for the agent.** The emerging pattern is to expose capabilities in
  forms an agent already handles well, such as command-line interfaces (CLIs).
- **An agent is an LLM calling tools in a loop.** Everything application-specific
  lives in the context, tools, and controls attached to that loop.
- **Intervene before a tool executes.** A before-tool-call hook can require
  confirmation, which supports approval and authorization checks.
- **Separate the layers.** Pi splits the coding agent, the agent core, a unified
  model interface, and the terminal UI; OpenClaw reuses those and adds plugins.
- **Keep the user where they work.** The sales assistant returns an editable
  draft in the inbox, not a new interface to learn.
- **A confirmation hook is not a sandbox.** Restricting filesystem, process,
  network, and credential access needs external isolation.

## Detailed Notes

### How small tools become product capabilities · 00:16
- **Matthias Luebken** builds agents for organizations at **TAVON.ai**.
- He got curious about **OpenClaw** because he wanted to understand the
  machinery, not just the flashy demos, and that led him to **Pi**.
- Pi is an open-source, minimal coding agent by Mario Zechner, who he notes was
  joining **Earendil**.
- He borrows Ken Thompson's Unix rule — programs should do one thing well — and
  applies it to agent tools.
- **Cowork** (Claude's desktop experience) packages a coding agent for work that
  is not software development.
- Its spreadsheet skill does not drive Excel directly; it composes tools such as
  **pandas**, **openpyxl**, and **LibreOffice** behind a task-oriented interface.
- The emerging pattern is to make the system easy for the agent to use. He says
  there is no settled playbook yet.

### Start with the core agent loop · 04:03
- His first advice: open Pi and ask it to build the functionality you want.
- One layer below a coding agent, an **agent** is a large language model (LLM)
  calling tools in a loop.
- The loop: give the model a goal and context (often an `AGENTS.md` file), let it
  request a tool call, run the tool, return the result, and repeat until it can
  answer.
- Everything application-specific lives in the context, tools, and controls
  attached to that loop.
- Pi exposes this as a TypeScript agent class that takes prompts and emits events.
- His first demo is a terminal **CRM** (customer relationship management) lead
  qualifier: a three-file TypeScript app.
- The request is simply "show all leads and score them," and the trace shows tools
  being called and results returned.
- The system prompt lists the available tools and when to use them. He calls it a
  small learning prototype.

### Control the boundary before a tool executes · 07:35
- The tool boundary is where an application can intervene.
- A **before-tool-call hook** can decide whether an operation should run — useful
  for authorization and role-based access.
- The key distinction: an agent requesting a change is not the same as the
  application permitting it.
- The source shows a TypeScript wrapper where a contact update stays pending
  until a confirmation callback approves it; only then does the write run.
- A confirmation is one application control; full authorization still means
  deciding who may make the request.
- **Events** handle the other side: telling the interface what happened.
- The terminal stream subscribes to activity, and a check mark shows a tool call
  completed and returned.
- The hook controls whether execution proceeds; events make its progress and
  outcome visible.

### Add a shell and runtime · 08:34
- A coding agent keeps the same loop but adds a **runtime and shell**, commonly
  Bash, so it can execute programs while solving a task.
- He retells an anecdote from Peter about OpenClaw and a voice message.
- OpenClaw had no voice capability then, so it assembled one from available
  tools, including **FFmpeg** on the local machine.
- He does not establish whether a dedicated voice plugin existed by the time of
  the talk.
- **The mechanism is composition:** behavior that looks like learning from
  outside can be another tool call inside the loop.

### Turn the CRM into an interactive extension · 10:16
- The next CRM example uses Pi's **extension API** for the coding agent, focusing
  on session events and UI interaction.
- A TypeScript extension registers a `/pipeline` command, loads the CRM context,
  and presents choices through the extension's UI.
- The workflow can now ask the user to decide instead of relying only on
  free-form prompts and backend tool calls.
- Running in the downloaded Pi coding agent, the command shows selection
  controls, including dropdowns, so a session can combine with explicit user
  decisions.
- He suggests asking the coding agent to generate an extension, but notes Pi's
  extension framework is oriented to coding-agent use cases.
- The next experiment is a **web UI**, built by asking Pi, reusing the same
  command and selection mechanism.
- He describes limits in the support and refactoring underway; this is an
  exploration, not a finished feature.

### Separate the execution engine from orchestration · 12:49
- **OpenClaw** moves from one agent in one session to many channels, threads, and
  agents.
- In the architecture he describes, OpenClaw uses Pi's core packages and calls an
  embedding function that creates an agent session and streams information back.
- So sessions are useful outside the terminal: an enclosing application can own
  the interaction while reusing the execution machinery.
- The packages split responsibilities: the coding agent (coding behavior), the
  agent core (the underlying loop), `pi-ai` (a unified LLM abstraction), and the
  terminal UI.
- OpenClaw adds its own plugin mechanism for needs beyond a coding session:
  multi-channel routing, provider orchestration, sub-agents, and gateway support.
- This is the historical integration shown in the talk. He warns that OpenClaw's
  current documentation describes its own runtime, so those internals should not
  be mapped onto this walkthrough.

### Build around the sales process · 14:55
- The client application starts from sales: customers email requests for
  proposals.
- Instead of a coding-agent interface, the team starts with the incoming email. A
  monitored inbox feeds a gateway that routes each message to an agent assigned
  to that customer.
- The customer agent combines three scopes: a **general harness** (`AGENTS.md`
  defining its role and how to respond), **customer context** (a Markdown file
  with access, perks, and discounts), and a **case session** (create or reuse a
  session per case for continuity).
- The customer determines which context applies; the case determines which
  conversation continues, so the user never manages sessions.
- Tools connect the agent to the CRM and **ERP** (enterprise resource planning)
  systems for business information such as contact details.
- The team exposes those tools as **CLIs** (command-line interfaces) because
  agents use them well — the opening "make it easy for the agent" principle in
  practice.
- They use their own sandbox and secure data before drafting, while acknowledging
  the sandboxing work is still developing.
- A gate can stop one tool operation, but it does not restrict filesystem,
  process, network, or credential access. Pi's documentation places those
  restrictions outside the agent, requiring external isolation.
- They are investigating **NVIDIA OpenShell** as a possible way to secure
  execution — under consideration, not deployed in the client system.

### Keep the user in email · 18:19
- The final demo shows the product from the inbox outward.
- The dashboard lists incoming email; an LLM relevance decision ignores most
  messages and selects one for processing.
- The selected email is associated with a case, and the case is backed by an
  agent session, which then produces a draft response.
- The dashboard is mainly for administrators; the user can stay in the familiar
  inbox-and-drafts workflow.
- Behind the draft, the session trace shows the assistant looking up items,
  calling tools, and continuing the loop (the example is in German).
- For the user it resolves into something simple: an email belongs to a case, and
  an editable draft is ready.
- Automatic sending is not part of the described outcome.
- His bet: **coding agents as reusable software building blocks**, embedded
  behind an interface that fits the work.
- His close invites people to experiment with Pi, because its minimalism makes
  the pieces easy to take apart and recombine.

## Resources

- *A Piece of PI: CRM examples* — the speaker's companion repository.
- *Pi Agent Harness* — Pi's source repository, including its coding agent and
  agent runtime.
- *Pi extension API* — TypeScript extensions, custom commands, and lifecycle
  events.
- *OpenClaw's current runtime architecture* — current documentation, which
  differs from the historical integration shown in the talk.
- *NVIDIA OpenShell* — NVIDIA's policy-controlled runtime for running agents
  more safely.
