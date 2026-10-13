# The End of Apps

| Field | Details |
| --- | --- |
| **Speaker** | Kitze |
| **Affiliation** | Sizzy |
| **Type** | Talk |
| **Date** | 2026-04-23 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=4fntwuOoedA) |
| **Tags** | `apis-mcp-and-protocols` `agent-engineering` `rag-context-and-search` |

## Summary

Kitze has been trying to fix his own productivity since he was a child, when his to-do list was a
checkbox next to "eat the string cheese." This talk follows that quest through text files and phone
automations, a personal life OS called Benji, and finally self-hosted personal agents — with the
candid admission that none of it has fixed his life yet.

The problem he keeps returning to: capturing an intention is easy, but getting it back at the right
moment, and trusting software to act on it, is hard. Each generation of tools removed some friction
and exposed another. Merging apps into one product did not remove the effort of maintaining it.

After a viral voice-to-app demo that never shipped, Kitze found the right "shell" for an assistant in
OpenClaw (agents you reach through WhatsApp or Telegram). Then five Discord servers of specialized
bots became a new coordination burden. His response is Wolfer, a deliberately small personal
experiment where context comes from a hierarchy of topics instead of a memory system.

His forecast: people will describe tasks to a computer instead of opening apps, the AI will start
prompting us for decisions, and most consumer apps will fade except specialist tools. He also guesses
that Apple could win this future if local models get good enough.

## Key Takeaways

- **Capture is the easy half.** The recurring gap is surfacing an intention at the right moment and
  then actually acting on it.
- **Consolidation is not freedom.** Merging tools into one app still needs forms and upkeep; Kitze
  cycled between logging everything and ignoring it entirely.
- **A visible capability is not a shipped product.** His voice-to-app demo went viral but never
  shipped, while a single-feature competitor made millions.
- **Capability and shell are separate problems.** Claude Code could already do personal work; a chat
  app like OpenClaw made it feel usable day to day.
- **Ask "does this need the package?"** In his Tinkerer Club group, the use cases that specifically
  needed OpenClaw shrank by around 90% once Claude Code or Codex were considered.
- **More agents did not mean more throughput.** Five Discords of specialized bots left rent,
  mortgage, and customer emails later than ever.
- **A promise is not an action.** An assistant that says it will do something, then doesn't, can make
  supervising harder than doing the task yourself.
- **Context can be assembled, not retrieved.** Wolfer injects a topic's and its parents'
  descriptions into the first prompt, which he finds more predictable than a memory system.

## Detailed Notes

### From notebook checkboxes to a life OS · 00:45
- Kitze's first to-do list was a childhood notebook reminder to eat his string cheese; the tools
  changed, his frustration with to-do apps didn't.
- About fifteen years ago he moved to plain text files, with the Android app **Tasker** (a phone
  automation tool) firing contextual reminders: ping me when I connect to Wi-Fi or arrive somewhere.
- **IFTTT** (a service that links apps with simple "if this, then that" rules) plus Google Assistant
  let him capture tasks by voice around the house. Not AI — but it made capture frictionless.
- He wanted a **life OS**, not another to-do app: tasks, habits, and calendar held together across
  web and mobile.
- Earlier attempts (Tododo, which scored tasks by tags to push them up a priority list, and Better,
  which was impossible to find in search) led to **Benji** in 2022, named after his dog.
- Benji grew into one app for routines, health, travel, finances, journaling, groceries, and more;
  a podcast host once pointed out that his grand Benji-phone ambition came with no Benji landing page.
- More features didn't fix the core issue: input still needed forms, and he oscillated monthly
  between logging everything and abandoning the system.

### Speech turns into API calls · 04:13
- When ChatGPT plugins arrived, Kitze phoned his wife to announce that apps and SaaS (software as a
  service) were finished — a call so repeated over three years that she stopped listening.
- One clarification worth keeping straight: plugins and **MCP** (Anthropic's later open standard for
  connecting tools to models) are related ideas, but MCP was a separate protocol, not renamed plugins.
- In 2023, getting usable JSON (structured data) out of a model still took coaxing: asking it not to
  write Markdown could still yield a friendly preamble before the JSON, forcing cleanup.
- He added a Benji feature where a keyboard shortcut started dictation; the tool cut up his speech,
  called Benji APIs, and the calendar and to-dos changed live on screen.
- The demo got excited reactions on Twitter, but the Benji assistant still had not shipped. Meanwhile
  a focused photo-based calorie tracker made millions — the talk's uncomfortable product lesson.

### Wrong shell, right capability · 05:38
- By the previous October, Kitze noticed Claude Code's tool calls were useful beyond programming:
  email, tasks, maybe even taxes.
- Mixing personal skills into his coding setup got confusing; what he wanted was an assistant with
  tools and context aimed at personal work.
- The terminal was the wrong interface. When Peter's Clawdbot — later named **OpenClaw** — made the
  assistant reachable through WhatsApp or Telegram, the capabilities suddenly felt usable daily.
- He joined its Discord early, when he recalls fewer than a hundred people having working setups and
  onboarding barely existing.
- He never learned the internals. Codex or Claude Code configured, repaired, and adjusted memory for
  him; asked what was in his JSON config, his honest answer was essentially "ask my bots."

### Owning the stack, and multiplying agents · 07:14
- The reaction went into "full lobster mode": a lobster suit at the first Vienna meetup, merchandise,
  tutorials, and (by his account) making the OpenClaw logo at 2 AM.
- The real attraction was access to a long backlog: Google Drive and iCloud files, presentations,
  high-school photos, unfinished ideas.
- Control over files, memory, and session deletion pulled him toward self-hosting — a NAS (a home
  network storage box), Nextcloud, Immich, and local Markdown. The setup felt local and owned; that
  does not prove model inference was local.
- He even switched back to Android, because he wanted an agent that could read and clear
  notifications and install or remove apps.
- Specialized agents package four choices: **inference** (provider, model, thinking level), **role**
  (a system prompt or "soul"), **capabilities** (tools and MCP connections), and **authority**
  (permissions).
- The result was bots for different parts of work and life and five Discords with channels, threads,
  and forum posts — and, he admits, a far messier life.

### Promises are not completed actions · 10:46
- Community excitement faded too: weekly meetups dropped to about five people and sounded, in his
  joke, like "OpenClaw anonymous," everyone comparing what their setups failed to do.
- His own failures clustered where a persistent assistant needs the most: cron jobs (scheduled
  tasks), agents coordinating and talking to each other, and remembering context from the last
  message.
- Discord and Telegram were built for messaging, not for running a life. He does not blame the agent
  projects — they were pressed into service until something better exists.
- Part of the charm faded with a model-access change and newer models; his colorful verdict on one is
  that talking to it feels like talking to a box of oats.
- The serious complaint is behavioral: ask whether something was done, hear "no," repeat the
  instruction, and find it still wasn't done. A commitment in chat is not evidence of execution.
- Delegation only pays off when supervising is cheaper than doing. For finding the venue address,
  opening the website beat waiting on an agent that might ask "are you ready for it?"

### Two futures, and the Wolfer experiment · 12:17
- Kitze sees two camps: custom agents (OpenClaw, Hermes) for tinkerers and cloud agents (Anthropic's
  Cowork, an expected OpenAI offering, Perplexity) for everyone else.
- In his view neither serves both audiences. Even pinball-machine-building tinkerers are tired of
  maintaining agents, while after about five minutes with Cowork he judged it capable of under 5% of
  what OpenClaw can do (his impression, not a benchmark).
- His daily rotation: OpenClaw, Hermes, Paperclip (a Kanban-like interface for agents that burns
  credits), and plain `tmux` with Codex — the terminal is the fallback when frameworks frustrate him.
- **Wolfer** is his own small answer: a thin layer over a coding agent, using Codex only when he
  presented, with no plans for broad provider support.
- Its limits are deliberate: its own chat UI only, no Telegram or iMessage, no plugin ecosystem, no
  memory system, limited modularity, and no promise it will ever be released.
- What he wants in return is predictability — conversation behavior he can rely on, in a UI built for
  many agents, topics, and conversations from the start.

### Context from the topic tree, not memory · 15:17
- **Explicit context assembly** is Wolfer's distinctive mechanism: topics form a hierarchy such as
  `work → projects → Benji → Benji customer support`.
- When a conversation starts, the first prompt receives the description of the current topic plus
  every parent, so the support conversation already "knows" about his work, projects, and Benji.
- His claim: this works better for him than hoping a memory system retrieves the right background,
  and the assembled prompt is inspectable.
- Execution is made visible: switchable workspaces, expandable tool calls, loading spinners, and a
  stop button instead of remembering a slash command.
- Scheduled cron messages read the entire conversation and carry an explicit cron label, so their
  origin is clear and the agent doesn't feel lost.
- A side panel shows the active agent, its model, and its capabilities; removing a capability removes
  it from that agent's configuration.
- A knowledge base plus dynamic `@` mentions (a document, a password, a skill) supplies the exact
  material for the task, while the topic tree supplies standing background.

### The computer that asks the questions · 17:32
- His starting observation: open a laptop after a few hours and it greets you with seventeen app
  updates and yesterday's tabs — the state of its applications, not the work that matters now.
- He imagines an OS that ingests notifications, email, and to-dos, then greets you with the next
  task and the one after that, and suggests a break when appropriate.
- The inversion: today we prompt the AI; in his forecast the most productive people delegate nearly
  all their work and the AI prompts them — for a passport photo, a form, a decision.
- Ordinary users won't need to program. They describe a task and the interface is generated on
  demand, so the habit of "finding an app" recedes.
- Specialist software survives: color grading, filmmaking, and music creation — places where the
  tool is part of the craft.
- His pick for a possible winner: Apple. If local models get good enough, a Siri-like agent taking
  tools from installed apps could act without per-request credit spend or sending data away.
- Nearer-term evidence he points to: Pixel phones doing background app tasks, including ordering
  coffee — a beta capability with monitoring and interruption controls, where the human still
  completes parts of the flow.

## Resources

- Benji — Kitze's personal operating system for habits, projects, and finances.
- OpenClaw — the personal AI assistant discussed throughout the talk.
- Hermes Agent — Nous Research's agent project.
- Paperclip — open-source tooling for managing agents at work.
- Claude Cowork — Anthropic's cloud assistant product.
- March 2026 Pixel Drop — Google's rollout including beta background app tasks.
- Gemini multi-step tasks on Android — device, region, and app limitations.
- OpenAI's November 2023 developer announcements — JSON mode for GPT-4 Turbo and GPT-3.5 Turbo.
- Introducing the Model Context Protocol — Anthropic's original MCP announcement.
- Historical Google Assistant + IFTTT automations.
