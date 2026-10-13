# Building pi: Why Coding Agents Need Smaller Cores, Clearer Context, and Human Judgment

| Field | Details |
| --- | --- |
| **Speaker** | Mario Zechner |
| **Affiliation** | PI |
| **Type** | Talk |
| **Date** | 2026-04-16 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=RjfbvDXpFls) |
| **Tags** | `coding-and-developer-tools` `agent-engineering` `reasoning-and-models` `architecture` |

## Summary

Mario Zechner explains why he stopped using mainstream coding agents and built
pi, a deliberately small coding agent that both he and the model itself can
modify. His starting complaint is that the tool, not the developer, controls the
model's context: system prompts, tool definitions, and injected reminders change
between releases, and the developer cannot easily see or steer them. That loss of
control, he argues, makes an otherwise good tool unpredictable.

He contrasts this with pi's minimal core — a short system prompt, a simple
tool-calling loop, and four built-in tools (read, write, edit, bash) — plus an
extension system that lets the agent write new capabilities during a session.
Extensions are TypeScript modules (files written in a typed version of
JavaScript) that the agent can author, load, and hot-reload without restarting.

The second half of the talk is a warning. Zechner argues that agents can generate
code and bug reports faster than humans can review them, so quality and
understanding quietly degrade. He describes maintainer filters for a flood of
low-quality, agent-written open-source contributions, and closes with a practical
rule: give agents tightly scoped work, and keep critical decisions and critical
code in human hands.

## Key Takeaways

- **Own your context.** If the harness silently rewrites prompts, injects
  reminders, or prunes tool output, agent behavior becomes hard to inspect and
  control.
- **A small core can compete.** pi keeps just four built-in tools and still
  reached sixth place on an October Terminal-Bench leaderboard before it even had
  compaction.
- **Make the harness self-modifying.** pi ships its own extension docs and
  examples, so the agent can write a new extension from the user's spec.
- **Extensions reach everywhere.** They can add tools, slash commands, shortcuts,
  event reactions, custom compaction and providers, and even change the terminal
  UI.
- **Protect maintainer attention.** Agent-generated issues and pull requests can
  bury a project; a human-first gate (write a short issue in your own voice)
  filters out the bots.
- **Speed without understanding compounds debt.** More agents mean more code and
  more defects than a team can realistically review.
- **Agents fill gaps with what they learned.** Models absorb patterns from public
  code, so a vague spec gets filled in with those patterns.
- **Bound the work and keep judgment human.** Give agents tasks small enough to
  verify, read critical code line by line, and make important decisions yourself.

## Detailed Notes

### The real problem was losing control of context · 00:15
- Zechner liked Claude Code early on because it was simple, predictable, and fit
  his workflow.
- As features piled up, more things broke, and a deeper issue surfaced: the tool,
  not the developer, owned his context.
- System prompts and tool definitions changed across releases; tools were removed
  or modified.
- It inserted "system reminders" into the context that said the information "may
  or may not be relevant," which he says confused the model.
- He also wanted more observability, model choice, and deeper extensibility than
  the native harness allowed; its hooks only spawn separate processes.
- Looking at alternatives, he criticized OpenCode for pruning tool output once it
  passed a token limit, which he says strips information the model still needs.
- He also disliked auto-injecting language-server (LSP — tooling that reports code
  errors) diagnostics after every edit, since code is often invalid mid-change.
- A security note: he flagged a default setup where a local server was reachable
  by any website open in the browser.

### A minimal harness can be more capable than a crowded one · 04:18
- He highlights Terminal-Bench, where one top harness was extremely minimal: just
  a tool to send keystrokes to a tmux (terminal) session and read output.
- On a December 2025 leaderboard, that minimal setup scored above larger
  harnesses, including some native harnesses for the same model families.
- His two theses: coding agents are still experimental, and developers need better
  ways to experiment with them.
- pi has four packages: an AI provider abstraction (with context handoff between
  providers), an agent core built as a simple tool-calling loop, a custom TUI
  (terminal user interface) framework, and the coding agent itself.
- Its built-in tools are only read, write, edit, and bash.
- He argues models already know how to be a coding agent from post-training, so a
  huge system prompt is unnecessary. Skills (markdown instruction files) add a
  little text.
- pi runs "YOLO" by default — no approval prompts — because he does not consider
  repeated bash confirmations a real security model, and expects users to build
  controls suited to their own needs.

### Extensibility makes the agent itself part of the development loop · 06:21
- pi ships handcrafted documentation plus example extensions, and simply tells the
  agent to read them and write its own extension.
- Extensions are TypeScript modules — in the simplest case, a single file on disk
  that you point pi at.
- The extension API can define tools, slash commands, and shortcuts; listen to
  events; save session state; implement custom compaction or providers; and
  modify the UI.
- Saved session state can be shared with the agent or kept for tools that analyze
  sessions.
- **Hot reloading** lets a developer build an extension inside a live session and
  see the effect immediately, echoing fast iteration from game development.
- Examples shown: a side chat with the agent during its main task, a chat room
  connecting multiple pi agents, and interfaces that run NES games or Doom.
- Optional features such as sub-agents, plan mode, and MCP (a standard way for
  agents to call tools) support are added as extensions rather than built in.
- Extensions are shared through npm or GitHub, not a new proprietary marketplace.

### Agent-generated contributions can overwhelm open-source maintainers · 10:10
- After pi became the agentic core inside OpenClaw, Zechner's project was hit by
  unwanted activity from OpenClaw instances whose users did not know.
- The problem is not only wrong contributions but the sheer volume of unsolicited
  issues and pull requests, which consumes a maintainer's time.
- His gate: initial pull requests are auto-closed with a comment asking the
  contributor to first write a short issue in their own voice.
- If he approves that issue, the account is added to a file in the repository, and
  later pull requests are allowed through.
- Automated agents rarely return to read the comment, so this small human step
  filters exactly the behavior he wants to discourage.
- Other controls: deprioritizing issues tied to OpenClaw interactions, embedding
  issue and PR text into 3D space to spot clusters, and temporarily closing the
  tracker to take a break.
- He is candid that these measures work "sort of," not completely — the tension
  between automated volume and finite human capacity remains.

### Speed without understanding compounds technical debt · 12:22
- More agents produce more code and more defects than a team can inspect; a
  separate review agent catches some problems but does not fix the imbalance.
- The real limit is how much resulting behavior humans can understand, evaluate,
  and maintain.
- His explanation is **local optimization**: models learn from public internet
  code, which he describes as largely mediocre or poorly designed.
- When a codebase is too big for the context window, an agent may add abstractions,
  duplication, compatibility layers, and defensive complexity without seeing how
  they interact across the system.
- A detailed spec helps only where it actually resolves decisions; leftover blanks
  are still filled by learned patterns. A sufficiently detailed spec, he jokes, is
  basically a program.
- Humans bring two things agents lack: a natural ceiling on how fast mistakes pile
  up, and the pain that motivates fixing them.
- The danger peaks when a team stops reading its own code. If something breaks,
  neither the humans nor an overloaded agent may have enough system-wide context to
  diagnose it.
- He is skeptical that bigger context windows or agentic search are guaranteed
  fixes, and warns that locally plausible patches can cause global regressions.
- Trust erodes further when the same agents that wrote the implementation also
  wrote its tests.

### Use agents for bounded work, and keep critical decisions human · 15:24
- His alternative is not to abandon agents but to give them a **well-defined
  scope**: a task small enough that the agent can find everything it needs to
  finish it.
- Modularizing the codebase makes those boundaries easier to draw, and an objective
  evaluation function makes success measurable.
- Good fits: optimization or research loops with measurable feedback, non-critical
  work, repetitive chores, and building reproduction cases from partial bug
  reports.
- Agents can also serve as a stand-in conversation partner when no human
  collaborator is around.
- The workflow still ends with human evaluation: keep what is reasonable and
  finalize the result deliberately.
- Closing advice: **slow down**, decide what is worth building, and resist adding
  features just because an agent can now generate them.
- Cap the amount of generated code a team can actually review, and use agents to
  polish the features that matter.
- For critical code, read every line; for important decisions, write the code by
  hand with the agent assisting rather than deciding. The friction is where
  understanding and learning happen.
