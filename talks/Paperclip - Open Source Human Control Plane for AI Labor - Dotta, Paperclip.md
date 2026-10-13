# Paperclip: Open Source Human Control Plane for AI Labor

| Field | Details |
| --- | --- |
| **Speaker** | Dotta |
| **Affiliation** | Paperclip |
| **Type** | Talk |
| **Date** | 2026-04-15 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=h403btjldDQ) |
| **Tags** | `rag-context-and-search` `apis-mcp-and-protocols` `enterprise` `reasoning-and-models` `leadership` `coding-and-developer-tools` |

## Summary

Paperclip's creator, Dotta, walks through an open-source orchestrator for AI agents. The product's headline is "zero human company," but he offers a plainer definition: a **human control plane for AI labor**. You configure an org chart of agents, and the human keeps supplying the taste and direction that make the output useful.

The demo starts with a local install (`npx paperclipai onboard`) and a tour of the real company Dotta uses to run Paperclip itself. It has a CEO above a CTO and several coders, plus marketing roles. Paperclip is bring-your-own-agent: Codex and Claude are his most-used coders, but Gemini, Pi, Hermes, and OpenClaw also fit, and all agents share memory and can communicate and negotiate.

To test the system, Dotta files an issue asking the CEO to hire a video writer and produce a Remotion video celebrating 40,000 GitHub stars. Paperclip exposes its own app operations to agents, so the CEO hires the writer and installs a Remotion skill from skills.sh. The human then gives editorial feedback — two-second cuts, animate the star count — and gets an on-brand video with real stats in about five minutes. The advantage over a standalone coding session is that the dashboard, brand guide, skill, and access already lived in the system.

The rest of the talk covers operating discipline: QA and approval roles so work is actually checked, recurring routines (pull requests, Discord summaries, changelogs), built-in code review via Greptile, and starting a new company one agent at a time. Dotta also covers budgets, model choice, and the ongoing work of tuning agent instructions. Many organizational-learning features are still manual and on the roadmap.

## Key Takeaways

- **Paperclip is a control plane, not a coding tool.** It is meant for creating and running businesses, and non-coders can use it.
- **Bring your own agent.** Codex, Claude, Gemini, Pi, Hermes, and OpenClaw can all be employees in the same organization.
- **The human directs the CEO**, which breaks work down through executives to individual contributors.
- **Reuse organizational context.** Dashboards, brand guides, skills, and access are stored once instead of rebuilt for every prompt.
- **Completion is not approval.** Separate assignee, reviewer (QA), and approver roles; finishing a task should trigger review.
- **Routines package recurring work** (PR handling, daily summaries, changelogs) and can take inputs like a branch name.
- **Match model cost to the role.** Reserve frontier models for the agents that need them and use cheaper options where they suffice.
- **Agent instructions are living documents.** Dotta keeps revising them whenever an agent behaves poorly.

## Detailed Notes

### From "zero-human company" to a human control plane · 00:01
- Install locally with `npx paperclipai onboard` (add `--yes` to accept the default options).
- Paperclip's showcase promise: hire employees, set goals, and automate jobs so the business can nearly run itself.
- Dotta's practical definition: **a human control plane for AI labor**, where your taste shapes how agents work.
- The premise is that you remain accountable for the work, so your preferences belong in the process from high-level design through execution.

### A request enters the org chart · 02:18
- In Dotta's own company, a CEO sits above a CTO and many coders; he mostly uses a **Codex** coder and a **Claude** coder.
- Paperclip is bring-your-own-agent: Gemini, Pi, Hermes, and OpenClaw are other examples he names.
- The orchestrator gives agents a shared setting to communicate, negotiate, and store memory.
- Beyond engineering, a CMO oversees marketing specialists such as a content strategist and a video writer.
- To test the system, Dotta creates an issue and assigns it to the CEO agent; the human directs the CEO, which delegates down the hierarchy.
- He reports Paperclip crossed 40,000 GitHub stars that week and expects it to pass 50,000.

### From a hiring request to a reviewed video · 04:12
- The task: hire a video writer and produce a Remotion video celebrating the milestone.
- Every part of the app has an agent surface, so the CEO can hire agents and install skills through the built-in skills manager.
- Here the CEO installs the **Remotion best-practices skill** from skills.sh; Remotion builds videos with React and needs no extra signup.
- The workflow: reference existing work (an `@` mention to the stats dashboard), request a plan, then correct the creative choices.
- Dotta asks for roughly two-second cuts instead of six and for the star count itself to animate.
- About five minutes after feedback, the result was an on-brand animation with real statistics and charts.
- The contrast: a standalone Claude Code session would first need the stats, brand guide, Remotion skill, and dashboard access gathered by hand.
- The real mechanism is **reusing organizational context across tasks**, not rebuilding it for each prompt.

### Make corrections reusable, then enforce review · 07:04
- Repeated feedback becomes material for improving agent instructions; a Paperclip-specific skill can add brand identity, style, and pacing preferences.
- In this version you create an agent to learn from conversations; deeper integration is planned for later versions.
- Dotta notes Paperclip released March 4 and this recording is April 8 — roughly a month of open-source development.
- The basic interface is familiar task management: configure agents in an org chart, make projects, and assign tasks.
- **QA** is how the organization makes expectations operational. A QA agent can use **agent-browser** to open websites, fill forms, and click buttons.
- The familiar failure: dozens of coding tabs, a request to test in the browser, and no test performed; hooks differ between Claude and Codex.
- Paperclip sits above individual agent harnesses, so agents from different labs can join the same workflow.
- Distinguish three roles: **assignee** (produces the work), **reviewer** (QA, iterates with the coder), and **approver** (judges organizational and brand fit). Finishing a task must trigger QA.

### Package recurring work as routines · 11:04
- **Routines** handle recurring work: pull-request handling, a Discord summary of everything merged to `master`, and release changelogs.
- They can be grouped by project or agent and run on a schedule or manually.
- Routines can accept inputs via placeholders like `{{branch}}`, so the procedure is reusable while the branch changes per run.
- Routines and skills overlap and can be combined; an organization that already has a PR skill can invoke it inside a routine.
- For the open-source project, Paperclip uses **Greptile** for first-pass reviews; a routine tells the agent to use the **Greploop** skill after submitting a PR.
- Dotta emphasizes this is a coding example — Paperclip is not itself a coding or code-review tool.

### Create a company, then earn each additional agent · 14:33
- One instance can hold multiple companies; Dotta builds a sample "agent tools directory."
- Initial setup makes three decisions: choose the CEO agent (he recommends Claude Code or Codex), set the first assignment (default: hire the first engineer and create a hiring plan), and add business constraints.
- The human must state what they value, because the agents cannot infer it.
- Templates exist for large organizations, but Dotta advises against starting with something like a 130-agent template before establishing how agents should behave.
- In the live company, the CEO asks permission to hire a CTO, and Dotta explicitly approves; the organization grows through need and approval.
- The advice: add agents individually, confirm quality, then fan out when the work warrants it.
- Reports can lead to more work later: a bookmark strategy report is currently just a report, with action buttons proposed for future versions.

### Budgets, instructions, and what comes next · 17:34
- Not every agent needs frontier-model pricing; Paperclip supports **OpenRouter** through the OpenCode agent for cheaper models.
- Example at recording time: **Qwen 3.6 Plus** was free up to usage limits; he warns cheaper models may not handle the most demanding work.
- Paperclip tracks monthly spend with budgets per agent and per project; the displayed zero monthly spend reflects Claude and Codex subscriptions, not zero cost.
- Default agent concurrency is one parallel run, with an option to increase it.
- Instructions are ongoing work: for the Codex coder he adds rules such as give a best-guess fix for blockers, avoid writing a whole test suite when tests were requested, and stop to fix instructions when behavior is poor.
- A **Skill Consultant** agent helps other agents use their skills better; much of this learning is still manual in the demonstrated version.
- Roadmap items include maximizer mode, multiple human users, cloud and sandbox execution (E2B, exe.dev, Claude agent deployments), artifacts, deployments, CEO chat, and a free desktop app.
- The walkthrough ends with agents working on deployment infrastructure, a CI pipeline, and partner integrations — and the human deciding where attention should go next.

## Resources

- [Paperclip](https://paperclip.ing/) — product site with local installation information.
- [Paperclip source and setup guide](https://github.com/paperclipai/paperclip) — code, installation, and architecture overview.
- [Remotion best-practices skill](https://www.skills.sh/remotion-dev/remotion/remotion-best-practices) — installable guidance for agent-made Remotion videos.
- [agent-browser](https://github.com/vercel-labs/agent-browser) — browser automation CLI and skills for agents.
- [Greploop](https://github.com/greptileai/skills/blob/main/greploop/SKILL.md) — cycles through Greptile review feedback, fixes, and renewed reviews.
- [Qwen 3.6 Plus pricing](https://openrouter.ai/qwen/qwen3.6-plus:free/pricing) — one example of a low-cost model option.