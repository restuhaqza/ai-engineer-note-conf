# Scaling Agents on Kubernetes with acpx and ACP

| Field | Details |
| --- | --- |
| **Speaker** | Onur Solmaz |
| **Affiliation** | OpenClaw; TextCortex at the time of recording |
| **Type** | Talk |
| **Date** | 2026-05-21 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=VaS2h-dY1-4) |
| **Tags** | `apis-mcp-and-protocols` `enterprise` `leadership` `developer-workflows-and-testing` |

## Summary

Onur Solmaz, an OpenClaw maintainer, explains how he turned chat channels into development workspaces and then into a way to scale agents. It begins with a problem: he wanted to use his preferred coding agent, Codex, from Discord, but had to ask another agent (Opus) to relay instructions. The relay paraphrased his wording — an uncontrolled transformation at the boundary where prompting matters. Binding a Discord channel directly to Codex through the **Agent Client Protocol** (ACP) removed the middleman and made Discord feel more like an IDE.

The talk separates three protocol boundaries: MCP (tools given to a model), ACP (interaction between a client and an agent), and agent-to-agent protocols. Solmaz built **acpx**, a CLI that lets any agent with command execution call another agent over ACP. He then uses it to attack a maintainer's flood of pull requests — more than 60,000 total, roughly 300–500 opened per day — by turning repeated review conversations into executable procedures.

The second half moves to enterprise scale. Solmaz argues for on-demand, disposable agents instead of one persistent personal assistant, and describes the infrastructure needed: Kubernetes for orchestration, an agent harness for execution, and synchronization for working files. He demos TextCortex Spritz, a self-hosted orchestrator that dispatches debugging agents from Slack into per-task environments. He is explicit that the last part is his TextCortex work, separate from his OpenClaw contributions.

## Key Takeaways

- **Don't relay through another agent.** Paraphrasing at the client boundary changes wording, which matters when prompting.
- **ACP standardizes agent-to-client interaction.** MCP gives tools to a model; ACP connects a client to an agent; agent protocols connect agents to each other.
- **acpx exposes ACP through a CLI.** An agent with command execution can invoke another agent across a common boundary.
- **Reaching an agent and delivering its artifacts are separate problems.** Codex could make a PDF but could not send it into Discord — that needed another channel and the surrounding harness.
- **A bad patch is still evidence.** A PR may be unmergeable yet reveal broken behavior worth recording.
- **Turn review into an executable procedure.** Intent, implementation judgment, conflict resolution, and CI repair can run as structured steps with JSON output driving the branch.
- **Loops have a bounded role.** Run an unattended review/refactor loop for shallow bugs — not architectural design; escalate fundamental refactors to a human.
- **Agent-per-task needs infra.** Chat platforms give one app identity, so disposable agents require Kubernetes provisioning and state synchronization.

## Detailed Notes

### Getting instructions to the right agent · 00:41
- Solmaz has been building agent harnesses since before ChatGPT, starting with a JupyterLab extension over an early Codex model. It was rebuilt into his startup's current harness over time.
- After following Peter Steinberger's "Claude Code is My Computer" post, he installed OpenClaw in his company cluster, where coworkers interacted with it through Discord. Enterprise needs led him to add Microsoft Teams support and become a maintainer.
- Early chat-driven development was a telephone game: he preferred Codex for complex work but, from Discord, was asking **Opus** to tell Codex what to do. Inspecting the Codex session showed his instructions had been paraphrased. (His assessment concerns the Opus version he used then; he notes the agents have since improved.)
- Direct channel binding made Discord work more like an IDE. He reports running one to five agents across separate channels, plus a channel for testing OpenClaw's ACP support. This made side projects practical while away from his desk.
- **acpx** was built through Discord itself: bind a channel to Codex through ACP, or via the Codex App Server integration credited to fellow maintainer Harold.
- The PDF example shows what channel binding does not solve. Before flying to London, he asked Codex to convert the ACP docs into a PDF. Codex could create the file but did not understand the chat harness or how to deliver an attachment into Discord. He saved it to a temp location, then moved to another channel to send it.
- **Reaching an agent and delivering its artifacts are separate integration responsibilities.**

### One client interface, multiple agent harnesses · 05:16
- ACP addresses the boundary between an agent and its client. The motivation is visible in editors: if every coding agent builds a separate plugin for every editor, much of the work is duplicated.
- Zed's idea is a common interface, so clients do not rebuild the same machinery for each agent.
- The three boundaries:
  - **MCP** — tools made available to a model.
  - **ACP** — interaction between a client and an agent.
  - **Agent-to-agent protocols** — communication between agents.
- These categories do not stop an agent from acting as an ACP client. An interface built for a human-facing client can also let one agent invoke another.
- Solmaz chose ACP because, when he needed it most, only Zed had the Codex and Claude Code adapters. His comparison concerns those adapters at the time, not Google's participation in ACP.
- OpenClaw's habit of adding features through a CLI suggested the next step: expose ACP through a CLI. That became acpx — starting as a way to invoke agents, then growing into a toolkit for ACP.

### A bad patch can still be useful feedback · 07:26
- OpenClaw sees more than 60,000 total pull requests, with roughly 300–500 opened per day on average at the time of the talk — his estimates of incoming work, not automated throughput.
- With so many stakeholders, the hard part is absorbing their needs without letting every local fix set the architecture.
- Review starts by asking what a PR actually does, then whether it is the best available fix. An AI-generated description may answer neither.
- Typical origin: a user hits a problem, asks their agent to fix it, and submits the result through GitHub — without doing the reasoning a maintainer needs before merging.
- **Separate the evidence of a problem from the proposed implementation.** A patch may be unmergeable yet still identify broken behavior that matters to a user. Categorizing that signal preserves it for future work.
- Solmaz shows Codex sessions making these judgments. After the same questions repeat, the procedure starts to look programmable — the next automation target.

### Turn review conversations into executable procedures · 09:39
- The PR workflow breaks into explicit stages:
  1. Identify the intent behind the incoming PR.
  2. Judge whether the implementation fits that intent.
  3. Resolve conflicts and address issues raised in review.
  4. Fix failing CI before presenting the result to a maintainer.
- The goal is to clear mechanical work from the maintainer's queue — not to make every patch mergeable.
- Repeated review/refactor loops have a bounded role. Solmaz argues an unattended loop can find shallow bugs and make superficial repairs, but should not be trusted with architectural design. When the required change is a fundamental refactor, relay it to a human.
- In acpx this is an n8n-like workflow engine driving a Codex session. The demo replays one PR: the agent reproduces the bug, assesses the refactor, and enters a review loop that yields no further findings.
- Programmatic steps surround the agent's work. The key connection is **structured JSON output** — a judgment becomes data the workflow inspects to choose its next step. The agent supplies a judgment; ordinary code handles the branch.
- Presentation described it as "standard operating procedures for agents," with the workflow engine usable for other agent-turn-plus-code workflows.

### Provision an agent for each task · 12:18
- Solmaz expects enterprises to consume far more inference than individuals, and sees commercial potential in that gap.
- His proposed model is on-demand, disposable agents: start one per task, let it work, and avoid forcing every task through one persistent personal assistant.
- **Chat identity is an obstacle.** Connecting an OpenClaw app to Slack, Teams, or Discord gives one app identity. Giving another agent its own name, avatar, and addressable presence can require creating another app and manifest — manual work that does not suit an agent-per-task workflow.
- A slide of multiple named agents illustrates the desired experience, not a shipping chat-platform feature: he generated the concept with ChatGPT.
- Because the desired provisioning is not supported, his implementation moves task-agent conversations into a separate UI. Those agents create and edit files, so the system must also keep working state synchronized.
- **The interface problem leads directly to an infrastructure problem.**

### A shared concierge, separate task environments · 14:46
- The proposed infrastructure has distinct responsibilities:
  - **Orchestration** — Kubernetes as the substrate for running task agents.
  - **Agent execution** — OpenClaw, Codex, or Claude Code as the harness, with ACP as a common interaction boundary.
  - **Working data** — repository read/write access plus synchronization to keep files and state where needed.
- Solmaz mentions GitHub, `rsync`, and Dropbox-like synchronization as possible ingredients; he does not specify one implemented algorithm for this part.
- The orchestrator comes from his TextCortex work, outside OpenClaw: a Go operator that handles the provisioning complexity.
- His example: a shared Slack concierge serving 100 employees becomes a bottleneck, so a new task should get its own agent. The concierge creates that agent and returns a website link to continue.
- He credits Cognition and Devin as precedents for this category of developer-agent experience.

### Dispatch debugging work from Slack into Spritz · 15:58
- The project is **TextCortex Spritz**, and the demonstrated use case is error reporting.
- A user asks in Slack about new bugs after a production release, then asks for an agent to investigate. Slack is the starting point; the new agent's conversation moves into a separate web interface because the desired Slack agent presence cannot be provisioned directly.
- The web UI is an open-source React app, hosted in the same cluster as the Helm charts. Opening the task starts a conversation there, and the agent begins working. The shown frame contains an error report and a debugging response covering root cause, fix, and verification.
- The experience resembles Codex Web or Devin, but the task runs in a full Kubernetes pod.
- Solmaz acknowledges this can be wasteful. He chose it for the flexibility of giving an agent a full, computer-like environment — a capability he credits OpenClaw with demonstrating. He tentatively mentions OpenHands and Firecracker while noting limited familiarity; his concrete claim is narrower: he has a working product running on Kubernetes.
- Intended deployments include internal web access to Codex and separate agents per task. He offers setup help to open-source projects handling hundreds of issues per day — an adoption target, not a demonstrated rate.
- **The product handles the Slack integration, agent lifecycle, and user interaction, while ACP abstracts the agent so the harness can be switched.** He reiterates that this orchestration and deployment work is his TextCortex work, separate from his OpenClaw contributions.

## Resources

- [acpx](https://github.com/openclaw/acpx)
- [TextCortex Spritz](https://github.com/textcortex/spritz)
- [Agent Client Protocol — Introduction](https://agentclientprotocol.com/get-started/introduction)
- [Codex is Live in Zed](https://zed.dev/blog/codex-is-live-in-zed)
- [Claude Code is My Computer](https://steipete.me/posts/2025/claude-code-is-my-computer)
- OpenHands V0 runtime options (legacy runtime documentation: Docker, local execution)