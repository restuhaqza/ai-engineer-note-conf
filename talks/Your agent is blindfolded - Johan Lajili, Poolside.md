# Your agent is blindfolded

| Field | Details |
| --- | --- |
| **Speaker** | Johan Lajili |
| **Affiliation** | Poolside |
| **Type** | Talk |
| **Date** | 2026-07-08 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=iRcX54EO5g8) |
| **Tags** | `apis-mcp-and-protocols` `agent-engineering` `coding-and-developer-tools` |

## Summary

Johan Lajili, an engineer at Poolside, opens with a split everyone recognises: one developer says AI writes everything for them, while another says the output is unusable in a real production app. He argues the real dividing line is not new code versus legacy code. It is whether the agent can observe the application and check its own work.

In a fresh project the agent's guesses usually match the environment it is building. In an existing app, dead code, hidden dependencies, and behaviour outside what the agent inspected mean a change can look correct but fail. So the feedback loop — not the age of the codebase — decides whether a result can be trusted.

At Poolside he built an internal CLI (a command-line tool) called Spoolside to give agents screenshots, compact page snapshots, logs, service control, navigation, and even conversation steps for the team's VS Code extension. His message is not the tool itself but the pattern: build application-specific access so agents can reproduce bugs and verify fixes. He frames this as a new engineering role focused on making AI-built work observable.

## Key Takeaways

- **Feedback beats codebase age.** The decisive difference between AI success and failure is whether the agent can observe outcomes.
- **"Done" is a claim, not proof.** An agent's confidence is bounded by what it could actually check.
- **Build application-specific access.** Screenshots, compact snapshots, logs, service restarts, navigation, and conversation flows.
- **Reproduce the bug before editing.** The agent should reach and recognise the failing state first.
- **Prefer exploratory checks.** Lajili treats his checks as ephemeral and likes mimicking a human tester (his personal preference).
- **Turn repeated actions into commands.** Frequent `sleep` calls reveal a missing "wait for ready" operation.
- **Design the surface for your product.** A game may need a text view of its world; a permission-heavy app may need quick logins.
- **The new role makes the product easy for AI to build.** He calls it the AIX engineer.

## Detailed Notes

### Two developers, two experiences · 00:16
- Lajili works at Poolside, one of the few companies training its own foundation models and coding agents.
- He opens with the familiar split: some say they never touch code now; others see "garbage" in production.
- He rejects the easy explanations — the enthusiast as a shill, the skeptic as a fake who never really tried.
- Greenfield versus brownfield does not fully explain it either: he has used AI successfully in legacy applications.
- Codebase age alone is therefore not the reason for the different results.

### Plausible code is not verified code · 01:54
- In a new project the agent's intuition is usually right: place a component here, a service there, and it behaves as expected.
- Brownfield apps hide surprises — code that is no longer used, and behaviour in parts of the repo the agent never looked at.
- That makes the **feedback loop** decisive: intuition suggests an implementation, observation proves it works in this app.
- When an agent says a workflow "works perfectly", it means only that it looks correct given what it could see.
- One user supplies logs and retries; another gives up. The opportunity is to let the agent obtain that corrective evidence itself.

### Give the agent access to the application · 03:44
- Lajili built an internal CLI, **Spoolside**, so agents can test the application.
- It can take screenshots and **snapshots** — compact, token-light representations of a page the agent can consume instead of the whole page.
- Poolside's app is a **VS Code extension**, not a webpage, so it needs extra machinery to be driven like one.
- Extended access includes pulling logs from frontend and backend services and restarting services.
- Higher-level actions include opening a specific menu or page.
- Conversation workflows can send a message, wait for the reply, follow up, and upload an image.
- These operations stack into a workflow, giving the agent both a way to act and a way to observe the result.

### Reproduce before you fix · 04:57
- Agents are eager: they often propose a fix immediately — add a margin here, insert code there.
- Lajili wants the agent to **reproduce the bug first**, proving it can reach and recognise the failing state.
- An intuitive fix may still be correct, but without inspection a human has to verify it anyway.
- That manual burden caps how much work can be delegated and makes unattended overnight runs hard to trust.
- Reproduction is a first step toward trust, not evidence that overnight autonomy already works.

### Make verification part of the environment · 05:48
- The work shifts from only building the product to making the product easier for an agent to work on.
- The levers are better tools, a more navigable codebase, and better knowledge bases.
- The interface can be a CLI, a skill, or an MCP integration (MCP is a standard way for agents to call tools) — he chose a CLI for simplicity.
- **Producing more is worthwhile only when errors don't pile up along with it.**
- He contrasts the 2025 "product engineer" with a proposed role he calls the **AIX engineer**, who makes AI-built work observable and reviewable.
- His oxygen-mask analogy: equip the AI to help itself before asking it to build more features, even if that slows delivery now.

### Ephemeral checks versus committed tests · 07:39
- An audience member asks where mechanical primitives like screenshots and logs end and checked-in unit or integration tests begin.
- Lajili describes his checks as fairly ephemeral and calls that a personal preference.
- He sometimes finds automated tests too rigid and hard to maintain over time.
- He prefers tooling that lets the agent explore the app the way a human tester would.
- His worked example: a button is visibly too far left. Instead of telling the agent, ask what capability would let it notice the problem itself.

### Find missing commands in the agent's logs · 08:53
- After several rounds of work, ask an AI to review the interaction history for recurring issues and repeated actions.
- Repeated `sleep(50)` calls are a signal: the agent may be waiting arbitrarily for a **missing "wait for condition" command**.
- Repeatedly starting Storybook can similarly reveal a workflow worth packaging into a reusable operation.
- The useful abstraction is a named readiness condition with a bounded wait.
- His TypeScript helper (`waitFor`) polls an application-specific probe every 100 ms until a timeout, then throws.
- The poll interval only controls how often the probe runs; it does not decide whether the app is ready.

### Choose an interface that fits the product · 09:16
- The right observation surface depends on what you are building.
- For a **Unity** game, he suggests considering an ASCII representation of the 3D world the agent can inspect while reasoning.
- For an app with extensive permissions, give the agent easy access to different logins so it can experience each role.
- A screenshot, a world representation, and a role-specific session each expose a different kind of evidence.
- Build the access that lets the agent discover the failures your product can actually have.

## Resources

- [gstack browser skill](https://github.com/garrytan/gstack/blob/main/browse/SKILL.md.tmpl) — example browser commands for screenshots and accessibility snapshots (mentioned).
- Inside Poolside's Model Factory — Poolside's account of its training and evaluation infrastructure (related).
- Playwright auto-waiting — how browser actions wait for visibility and stability (related).
- [Testing UIs with Storybook](https://storybook.js.org/docs/writing-tests) — using component stories for interactive development and tests (mentioned).
