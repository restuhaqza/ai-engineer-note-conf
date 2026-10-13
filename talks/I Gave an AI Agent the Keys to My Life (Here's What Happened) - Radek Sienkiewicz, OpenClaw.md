# I Gave an AI Agent the Keys to My Life (Here's What Happened)

| Field | Details |
| --- | --- |
| **Speaker** | Radek Sienkiewicz |
| **Affiliation** | OpenClaw |
| **Type** | Talk |
| **Date** | 2026-05-02 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=sJ2jc7leKBk) |
| **Tags** | `rag-context-and-search` `reasoning-and-models` `safety-and-governance` `agent-engineering` |

## Summary

Radek Sienkiewicz, an OpenClaw maintainer, describes how he handed a personal agent
access to his email, notes, files, calendars, tools, and operating system — and why it
didn't go badly. The key was pacing. He started with one WhatsApp channel that could
only chat, moved to Telegram and then Discord, and added a single simple task at a
time. Because no change felt large, he kept thinking of his setup as simple; comparing
notes with other users showed how much had quietly accumulated. Trust grew through
changes he could undo: solve a need simply, extend only once it works, and when
something breaks, step back, diagnose it, fix it, and add a safeguard before moving on.

The system became genuinely useful once he connected his Obsidian knowledge base —
around 3,000 Markdown notes built over years. The unexpected effect is that saving a
bookmark surfaces related older notes, so **ingestion also becomes retrieval**. A
nightly routine (roughly 3–6 AM) indexes and backs up everything so each morning starts
prepared. His agent's work falls into a few areas: ambient operations, attention
filtering, execution support like drafted replies, and inbox synthesis. LLMs supply
judgment; deterministic scripts handle known conditions and run without the model.

The closing theme is maintenance and mindset. Growing the system is not just adding
capabilities — bad memory compounds, long automations get brittle, and noisy notes pile
up, so pruning matters. Instructions live in inspectable Markdown files, and the whole
point, for Radek, is to help his future self by getting tomorrow's work done today.

## Key Takeaways

- **Grant access one workflow at a time.** Trust grows through small, recoverable
  changes.
- **A personal knowledge base is the multiplier** — ~3,000 Obsidian notes giving the
  agent project context.
- **Ingestion doubles as retrieval:** adding a link resurfaces related older notes you
  may have forgotten.
- **Overnight jobs** run roughly 3–6 AM to index, back up, and update, so the day
  begins ready.
- **LLMs are for judgment; scripts are for known rules.** Simple if-this-then-that
  tasks can skip the model entirely.
- **Memory is editable Markdown**, not hidden state — you can read and fix it. A
  separate critical-rules file, referenced high in `AGENTS.md`, helped.
- **Growth needs pruning:** bad memory compounds, ten-step automations break, noisy
  nodes accumulate, and weak boundaries need correction.
- **Build for your future self** — let the agent finish the delegable work so tomorrow's
  you starts with less waiting.

## Detailed Notes

### Access grows one workflow at a time · 00:20
- "Giving the keys" is nearly literal: the agent can reach email, notes, files,
  calendars, tools, and the OS, building on memory of his computer activity.
- Installing it and expecting it to run everything immediately would be reckless; that
  isn't how it happened.
- He began with one WhatsApp channel that could only chat, later moved to Telegram, then
  Discord.
- The pattern: see a need, solve it in a simple way, then add the next step. Each change
  was small enough that he still thought of the setup as simple.
- The same progression took him from user to OpenClaw maintainer: a workflow error led
  to a pull request, then another, then Discord involvement. Being a maintainer isn't
  required — the useful habit is making each problem small enough to understand.
- His operating loop: if something breaks, step back, diagnose, fix it, add a measure so
  it won't recur, then continue. He credits this for avoiding the "it bricked my
  computer" failures others report.

### The knowledge base makes access useful · 04:48
- The system became far more helpful when he connected his existing Obsidian vault:
  roughly 3,000 Markdown notes accumulated over years.
- It holds work and personal material, tasks, projects, research, articles, and an inbox
  of saved links — background that connects a new request with work already underway.
- Retrieval isn't one undifferentiated memory store. He combines ordinary search,
  **QMD** search for Obsidian, and separate memory for his workspace.
- These parts are interlinked, and that interlinking is where "the magic" happens.

### Bookmarks become connected knowledge · 06:57
- The vault graph he shows is his real graph, with dense clusters (likely projects) and
  more isolated nodes (likely one-off bookmarks).
- When a link arrives (tweet, thread, article, or YouTube video), the workflow:
  1. Inspects and analyzes the linked content.
  2. Adds tags and contextual information.
  3. Finds existing vault material on the same subject.
  4. Considers where else the new material could be useful.
  5. Adds connections to those existing notes.
- The result is more than a saved URL — it's material situated in a personal knowledge
  base.
- He often finds he'd forgotten exactly the older note that would help him think through
  the new link, so saving something surfaces what he already knew.

### Overnight maintenance and guarded updates · 08:54
- The overnight routine prepares the system for another day. He says the jobs actually
  run roughly between 3 and 6 AM (the slide's 4 AM is shorthand).
- They index content and make backups; his stated expectation is that a loss would cost
  a couple of hours of work or content — an expectation, not a demonstrated restore.
- It refreshes indexes for QMD, memory, and the Obsidian vault, and morning output
  can include email and calendar summaries.
- Updating OpenClaw is part of this and took its own incremental work: scripts encode
  what an update should and shouldn't do, known causes of breakage, and checks to run
  before updating or restarting the gateway.
- The operational requirement is that the gateway comes back online; by morning, the
  updated system is ready.

### From maintenance to prepared replies, organized by Discord · 10:25
- He won't share his exact setup because it's tuned to *his* needs, anticipated needs,
  and the errors he encountered.
- He groups the agent's work into areas:
  - **Ambient operations:** updates and plumbing that should happen without his
    attention.
  - **Attention filtering:** deciding what deserves interruption. Using project
    background from Obsidian, it judged importance and urgency.
  - **Execution support:** preparing replies.
  - **Synthesis:** interpreting new inbox sources and joining them to existing knowledge.
- Concrete examples he cites: a Netflix payment failure fixed within five minutes, and a
  domain renewal caught from an email he'd likely have missed — flagged on Discord and
  renewed.
- Drafting replies with project context is **execution support with a visible handoff**:
  the agent reads an email, considers what the project has already done, and leaves a
  response in the drafts folder for him to accept, edit, or delete.
- Discord organizes recurring work. Conversations start in General, and a dedicated
  channel appears once a type of conversation recurs:
  | Channel | Recurring work |
  | --- | --- |
  | Inbox | Receive links; build the knowledge base |
  | Consulting | Client context, projects, quotes, deadlines |
  | Video research | Research for upcoming episodes |
  | Briefing | Morning briefings |
  | Instagram | Social posting |
  | YouTube | Video creation |
  | OpenClaw | Maintainer work |
  | Playground | Test models, workspaces, memory arrangements |
- The **Playground** gives experiments a home next to working flows; if something works
  he promotes it, otherwise he discards it.

### Judgment, scripts, and editable memory · 14:51
- The system works through a division of responsibilities: **LLMs supply judgment**
  (understanding an email, interpreting context, making connections), while files,
  tools, and scripts supply the machinery.
- When a task is a known condition followed by a known action, a script can run it and
  skip the LLM entirely.
- Memory needs deliberate upkeep. He works on `SOUL.md` and `AGENTS.md`, and after
  finding instructions could still be missed, added a separate **critical-rules**
  Markdown document referenced near the top of `AGENTS.md`, which he says helped.
- A compact rule can make an intended handoff explicit, e.g. an "Email replies" rule:
  use relevant project notes, save the reply as a draft, and leave acceptance, editing,
  or deletion to the user.
- Written rules guide behavior but **do not enforce permissions** — real controls live in
  permissions, tool restrictions, and sandboxing.
- His memory arrangement grew from a single file to a whole folder, and he describes
  "dreaming," which promotes memories.
- Everything stays inspectable: these are Markdown files he can open, read, understand,
  and edit — so he can improve what the system remembers instead of trusting hidden
  state.

### What becomes harder as the system grows · 16:28
- More memory and automation create their own maintenance work:
  - **Bad memory compounds.** A poorly organized memory system gets more troublesome as
    notes and memories grow into the thousands; it needs active attention.
  - **Long automations become brittle.** He cites a ten-step workflow as the kind of
    thing likely to break eventually — split it up or add more effective guardrails.
  - **Noisy nodes accumulate.** He regularly cleans the knowledge base and removes
    entries that add noise.
  - **Weak boundaries need correction.** He revisits `SOUL.md` and related files to make
    the intended behavior fit his needs.
- The lesson: growth includes pruning and simplification, not just another capability.

### Build for the future self · 17:19
- Start with one recurring pain. Solve it, grow trust incrementally, build the knowledge
  base, and move as much material as you want into Markdown so connections can form.
- Keep the system inspectable — these habits make it possible to build something useful
  without reproducing someone else's entire environment.
- He closes with a comparison of past, present, and future selves: the past self seems
  lazy and left today's work undone; the present self carries it; the future self is
  imagined as an all-powerful being who will handle anything postponed. That asymmetry
  makes deferring feel easy.
- His alternative is to treat the future self as a friend worth helping. The agent's job
  is to do the delegable work that makes tomorrow easier, so that when he wakes up, as
  much as possible is already prepared.
- That, for Radek, is the purpose of giving the system more access: less unfinished work
  waiting for the person he'll be tomorrow.

## Resources

- [QMD](https://github.com/tobi/qmd) — local document search with keyword and semantic
  retrieval, reranking, and agent interfaces.
- [OpenClaw Memory Masterclass](https://velvetshark.com/openclaw-memory-masterclass)
  — Radek's guide to instruction files, retrieval, and compaction.
- 50 days with OpenClaw — his account of daily personal-agent use and operational
  failures.
- LLM Wiki — Karpathy's pattern for an agent-maintained Markdown knowledge base.
- OpenClaw memory documentation — memory files, retrieval, and background maintenance.
- [OpenClaw security boundaries](https://docs.openclaw.ai/gateway/security) — trust
  boundaries, tool restrictions, and sandboxing.
