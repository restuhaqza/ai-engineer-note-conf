# Building your own software factory

| Field | Details |
| --- | --- |
| **Speaker** | Eric Zakariasson |
| **Affiliation** | Cursor |
| **Type** | Workshop |
| **Date** | 2026-04-28 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=rnDm57Py54A) |
| **Tags** | `infrastructure-and-deployment` `architecture` `coding-and-developer-tools` `leadership` `agent-engineering` `apis-mcp-and-protocols` |

## Summary

Eric Zakariasson, an engineer at Cursor working on developer experience and product, asks what it would actually take to delegate software production to agents — to hand over a goal and let agents build, test, and ship. He borrows Dan Shapiro's six-stage autonomy ladder to describe how responsibility moves from the person to the system, from "spicy autocomplete" up to a dark factory where humans mostly supply intent.

His argument is that a software factory is not just a better prompt. It needs assembly lines: a repository agents can understand, guardrails and tests that let them check their own work, isolated environments so parallel agents do not collide, and feedback loops that turn repeated handoffs into automation. He demonstrates the ideas on a small Ableton-style web music tool that he deliberately refuses to code by hand, exposing the missing infrastructure — a way to start the server and a way to verify changes — as he goes.

The through-line is that the human role shifts toward management and judgment. Zakariasson keeps safety, security, payments, and authentication as human responsibilities, and warns that activity is not progress: a factory also needs to notice when an agent is busy but getting nowhere. He is candid that much of this is aspirational, and that architecture, team agreement, and stalled-work detection remain hard.

## Key Takeaways

- **Autonomy is a ladder, not a switch.** Most teams sit around pair-programming (levels 2–3); the factory (level 5) means the human supplies intent while agents build, test, and ship.
- **Make the repo agent-friendly.** Co-located code, existing patterns, startup scripts, and reusable boilerplate cut the choices an agent must invent.
- **Guardrails serve different jobs.** Hooks restrict sensitive areas, rules describe expected behavior, and tests give observable results. Rules are prompt context, not a security boundary.
- **Verifiable beats merely runnable.** Agents need a way to prove their work — unit, integration, and UI tests, or scripted browser checks.
- **Isolation is more than a branch.** Worktrees separate files but share databases and caches; full VMs avoid cross-agent side effects at higher cost.
- **Automate repeated handoffs.** If a human keeps copying logs, feedback, or specs around, a skill, MCP integration, or scheduled automation should do it.
- **Route review by risk, not location.** A variable rename and a change to trial-duration logic are not equally risky; agentic ownership can auto-approve low-risk changes.
- **Watch for no-progress agents.** Runtime, files touched, and repeated loops are signals that persistence has stopped being useful.

## Detailed Notes

### From autocomplete to a software factory · 01:26
- **Dan Shapiro's six levels** move responsibility gradually from human to system: 0 Spicy Autocomplete, 1 Coding Intern, 2 Junior Developer, 3 Developer, 4 Senior Developer, 5 Software Factory.
- Zakariasson places most adopters at levels 2–3 (back-and-forth pairing) and himself mostly at level 4, where he delegates work and reviews outputs before code.
- Level 5 is Shapiro's **dark factory**: the human gives intent and goals while production runs behind the interface.
- Why bother? Throughput (agents run 24/7), consistent output from assembly lines, and better leverage of human taste.
- If agents feel increasingly random as their freedom grows, read that as a signal the guardrails are thin — and better models follow instructions more reliably over time.

### Build a factory-friendly repo · 04:59
- Start with **primitives and patterns**: co-located, modular code lets an agent discover files with `ls` instead of searching all over.
- If a human can onboard quickly, an agent probably can too; existing auth services, startup scripts, and test conventions give it references to copy.
- **Hooks and checks** restrict changes to costly areas such as encryption and authentication.
- **Rules** should emerge from observed problems, not be installed wholesale. Using Next.js does not mean you need every Next.js rule; write rules when agents drift, and treat them as operating procedures rather than an enforced boundary.
- **Tests** let the agent discover breakage and confirm its change still runs — a rule asks for behavior, a test supplies a result.
- **Enablers** make independent work possible: skills and MCP integrations expose external context. A feature-flag skill, for example, lets an agent ship a change behind a flag, ready to enable, revert, or expand.
- Three practical checks for the environment: **runnable** (can the agent start it alone?), **accessible** (can it reach Linear, Notion, Datadog, Slack?), and **verifiable** (can it check its own work?).
- Backend verification is usually easier because contracts are clear; a UI needs the agent to click through the DOM and confirm, say, that a loading spinner actually appears.

### Demo: start, run, and verify a music app · 10:42
- The demo runs in **Cursor 3**, an agent-first workspace that Zakariasson describes as a rewrite without VS Code (the traditional IDE remains available).
- The first request is just to start a local dev server; the agent reads `package.json`, finds the start script, and serves on `localhost:3000`.
- This is what making a repo familiar to the model means: standard JavaScript conventions already hint where to look. An `AGENTS.md` file carries project instructions across harnesses.
- To expose missing infrastructure, he forces himself not to write code and to inspect little of it.
- The agent creates **Playwright** end-to-end tests that open browsers, visit routes, and use `getByTestId` to exercise playback and note creation; Vitest is also present.
- The useful contract is observable behavior: playback toggles state, and adding a note increases the note count. Selectors and exact assertions are illustrative.

### Give humans reviewable evidence · 14:35
- Implementation is one stage of the assembly line; planning, producing, and reviewing stay recognizable even as the factory codifies them.
- An agent can review its own changes, or **Bugbot** can review a GitHub PR and return findings.
- **Computer-enabled cloud agents** get a separate, reproducible VM and tools to interact with the app, producing evidence a diff cannot.
- In the Glass demo, the task is keyboard accessibility (`Control+Tab`); the first recording showed only the sidebar, so he asked for the full editor, and the returned video shows the highlighted row and keyboard navigation.
- The recording gives the reviewer context about what the agent actually exercised and what the interface did.
- Automated review, tests, and rules cover much of the workflow; his talk of effectively unlimited scaling describes architectural ambition, not measured capacity.

### Isolate work, then multiply agents · 20:33
- As agents run in the background, following every trace gets harder; you need aggregated updates and eventually agents that manage agents — like a human org adding managers.
- **Parallelism starts with scope.** Two tasks touching the same code can conflict, so the human must know data flows and critical systems well enough to divide work.
- Longer asynchronous tasks need context up front: a plan or specification that states the intended result before the agent leaves the conversation.
- Practice reveals each model's strengths and weaknesses; better models may need shorter prompts, but clear intent still matters. Learn these limits without granting production-push access immediately.
- **Worktrees** give separate working directories on a shared repo but do *not* isolate databases, caches, or user state — branches sharing services can affect each other.
- Independent VMs separate full app and service environments, at the cost of provisioning and setup. Zakariasson prefers a VM per task and guesses (without certainty) that Cursor runs multiple thousands of agents daily.

### Automate handoffs and review by risk · 22:20
- Hunt for places where a person acts as a context-transfer mechanism: pasting Datadog logs into a session, moving Twitter feedback into a task, exporting a Notion spec.
- Mistakes point to improvements: wrong database naming suggests a missing rule; consistently poor UI suggests an absent or unreachable design system.
- The agent can scan chat history for repetitive tasks, surfacing repeated plan-execution loops, product direction, UI iteration, and housekeeping — durable rules are candidates.
- **Scheduled/event automations** turn this into routine: a daily job reads Slack and GitHub and summarizes yesterday; another collects human review comments on merged PRs so corrections are reused.
- **Static code ownership** is a bottleneck: Cursor's code owners were right about 80% of the time, and the remaining 20% caused waits, sometimes across time zones.
- **Agentic code ownership** starts from what a change means. Low-risk changes can be auto-approved; higher-risk ones bring in previous contributors to check safety and refresh context.
- The **Continual Learning plugin** stores recurring preferences (a component to use, a preferred explanation depth) as rules, because he does not reliably write them himself.

### Durable rules, and function vs. visual acceptance · 40:58
- The plugin extracts rules from transcripts; updating model weights to a team's codebase would be another form of continual learning, discussed but not demonstrated.
- The immediate problem is familiar: stateless agents forget instructions, and that lost continuity undermines trust. Rules bridge default model behavior and human intent.
- Concrete example: Cursor omits foreign keys in certain migrations for performance, while a model defaults to including them. Bugbot flags the divergence against a rule — useful because it captures a known decision, not general advice.
- For new projects, preferences may not be stable yet; temporary specs and plans are better until patterns settle.
- **Separate functional proof from visual acceptance:** a working login does not prove pages look right. For a docs change, he asked an agent to visit every affected instance and return screenshots, then supplied the visual judgment himself.
- Computer use explores from task instructions, while Playwright or Puppeteer scripts give reviewable, reusable steps. Computer use takes time and many tokens.
- In the component-comparison demo, a failing View Code button was fixed by a cloud agent that started the dev server, interacted with the page, and returned a recording.

### Run and scale the factory · 50:42
- Hide startup complexity behind a dependable interface: an internal tool exposes backend-start and frontend-start, masking OrbStack, ClickHouse, Postgres, Redis, Electron, and Glass. An authenticated snapshot can preserve login state.
- Cost is real but variable — he guessed about $1 for the first demonstrated turn and had not checked; the chosen model also affects cost.
- **Hand off intended interaction, not an accidental architecture.** PM prototypes in Claude Code can encode backend choices delivery teams cannot keep; a prototype can instead serve as a specification of behavior to rewrite in the production stack.
- **Plan synchronously, execute asynchronously.** Task scope sets how long an agent stays busy; parallelism sets how many run at once. He typically works across roughly four repos with about five to ten agents, five async in the cloud.
- Keep the factory understandable: check readable practices, rules, and process descriptions into a repo folder, with a forum to decide what belongs there and what capabilities are missing.
- **Let system events initiate maintenance:** an internal Linear integration starts a cloud agent for each new ticket; a feature flag at 100% for two weeks becomes eligible for removal via an auto-created issue and agent change (a human still inspects and merges).
- **Self-hosted Cursor Workers** put the cloud-agent orchestration layer on a machine you control (`agent worker start`), running on a dev machine, Mac mini, or another VM while still using hosted frontier models. Local tool execution and local inference are separate choices.
- Ownership: monthly shipping goals and broad domain ownership beat a rigid process, though he expects scaling limits; agentic code ownership is one response.
- **Detect no-progress activity:** an agent in the wrong repo kept trying to gain access for an hour. Useful signals include elapsed runtime, whether any files were touched, and repeated loops — Cursor's own repo is heavily tuned, which can hide problems that appear in fresh projects.
- Roles remain distinct: PMs set direction, designers move between Figma and production code (roughly 50/50 by his estimate), engineers collaborate on implementation, and data scientists instrument usage and flag bottlenecks.

## Resources

- [Meet the new Cursor](https://cursor.com/blog/cursor-3) — Cursor 3's agent workspace and local-to-cloud handoffs.
- [Cursor rules and AGENTS.md](https://cursor.com/docs/rules) — reusable and scoped project instructions.
- [Playwright locators](https://playwright.dev/docs/locators) — how tests find and drive browser elements.
- [Git worktrees](https://git-scm.com/docs/git-worktree) — separate working directories attached to a shared repo.
- [Build agents that run automatically](https://cursor.com/blog/automations) — scheduled and event-triggered workflows.
- [Continual Learning plugin](https://cursor.com/marketplace/cursor/continual-learning) — extract durable preferences from transcripts.
- [Run cloud agents in your own infrastructure](https://cursor.com/blog/self-hosted-cloud-agents) — self-hosted workers.
- [Cursor agents can control their own computers](https://cursor.com/blog/agent-computer-use) — computer-enabled cloud agents and their artifacts.
- "Securing our codebase with autonomous agents" — Cursor's security team on PR review, dependency review, and related workflows.
