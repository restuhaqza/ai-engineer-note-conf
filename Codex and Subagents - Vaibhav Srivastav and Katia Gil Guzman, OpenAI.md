# Codex and Subagents

| Field | Details |
| --- | --- |
| **Speaker** | Vaibhav Srivastav, Katia Gil Guzman |
| **Affiliation** | OpenAI |
| **Type** | Workshop |
| **Date** | 2026-04-29 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=MhHEGMFCEB0) |
| **Tags** | `infrastructure-and-deployment` `coding-and-developer-tools` `developer-workflows-and-testing` `reasoning-and-models` `apis-mcp-and-protocols` `agent-engineering` |

## Summary

Vaibhav ("VB") Srivastav and Katia Gil Guzman, from OpenAI's Developer
Experience team in London, present Codex as an agentic software engineering
system rather than a code generator. Writing code is only part of the job: an
engineer also explores the repository, runs commands, runs tests and interprets
what happened. The workshop is demo-first, walking through plugins, scheduled
automations, code review and custom subagents.

The through-line is delegation. As agents produce more work, a person cannot
read every change, so **review becomes a first pass** and the skill shifts to
splitting work into slices that agents can own. Codex offers review on GitHub,
in the app or CLI, and even inside Claude Code, with findings labelled by
severity.

The session is candid about limits. Some live demos do not finish, creating a
subagent hits a configured concurrency cap, and the presenters state plainly
that cloud tasks do not support packaged skills. They end on a trust point:
reading a repository resource and authorizing a bundled executable are different
operations.

## Key Takeaways

- **Codex = models + a unified agent harness.** The harness handles tool execution, environment setup, behavior and safety; the app, IDE, CLI, Slack and GitHub are just surfaces.
- **Models span sizes and speeds.** GPT-5.3-Codex, a fast Spark variant built with Cerebras, GPT-5.4, plus mini and nano models for short tasks and subagents.
- **Plugins bundle three things:** skills (reusable instructions, scripts and resources), apps (connections like Notion or Linear) and MCP servers (external tools).
- **Automations are cron for agents.** Scheduled background instructions handle recurring work such as daily Slack or Gmail triage.
- **Code review is the first pass.** It runs on PRs (with P0/P1/P2 findings), via `/review`, or through a Codex plugin inside Claude Code.
- **Subagents split independent work.** Use review slices (a few files each) or role-based personas; give reviewers read-only sandboxes.
- **Hooks tie scripts to lifecycle events** (after tool use, session start, stop); a stop hook can force one more bounded, validating pass.
- **Mind the trust boundary.** Cloud tasks can read checked-in repo resources but do not support packaged skills, because a skill may bundle an executable the sandbox cannot trust.

## Detailed Notes

### Beyond code generation: the harness · 00:16
- Coding is one part of software engineering; exploring a repo, running commands and tests is the rest.
- **The model supplies capability; the harness makes it usable in an environment.**
- Models named: GPT-5.3-Codex, its fast Spark variant, GPT-5.4, and a recently released mini model.
- The unified agent harness handles tool execution, environment setup, agent behavior and safety.
- Surfaces include the app, IDE extension and CLI, plus Slack and GitHub for delegating work. Integrations cover Figma, Linear and Notion.

### Models and serving speed · 04:28
- The progression: GPT-5.2 → GPT-5.2-Codex (longer tasks) → GPT-5.3-Codex → GPT-5.3-Codex-Spark (built with Cerebras) → GPT-5.4.
- GPT-5.4 mini and nano target short tasks and subagents.
- The real choice is about task difficulty and how much time and compute it deserves.
- VB reports roughly 1.75× faster tokens from WebSockets at no extra user cost, then another 2× from fast mode; he gives no baseline, so these are workshop figures.
- Later documentation describes 1.5× speed for fast mode and 2× Standard credit use for GPT-5.4 — useful context, not a spec of the demonstrated build.

### The app, worktrees and automations · 07:01
- The app groups projects (Codex, ChatGPT, Sora) and uses **Git worktrees** so features, bug fixes and investigations get separate working directories with less interference.
- Native Git support lets you push with the appropriate identity.
- Automations run background instructions on a schedule — every morning or evening, or a calendar briefing.
- Windows support arrives with a native Windows sandbox; VB calls it a first among competing harnesses.
- Adoption snapshot per the speakers: Codex crossed 3 million weekly active users the night before, more than tripling since January.

### Plugins: skills, apps, MCP · 12:28
- A **skill** is reusable instructions, scripts and resources; Codex can help write one.
- An **app** is a connection to a service like Notion or Linear; an **MCP server** exposes tools from an external system.
- Plugins bundle these so you don't install each piece separately, though external services may still need their own authorization.
- Automations reuse these capabilities on a schedule, much like a cron job.
- For visual work: **Playwright Interactive** opens an app, clicks controls, takes screenshots and inspects the result; **ImageGen** creates assets and sprites. The Game Studio plugin combined them to build a platformer.

### Scheduling triage and checking results · 17:28
- With the Google Drive plugin, Codex populated a meetups spreadsheet from YAML files in the repo: about two minutes to analyse, then 57 event rows.
- Recurring automations already handled Slack triage (daily at 9 a.m.) and Gmail triage (reply-worthiness, urgency, legitimacy).
- Katia says the email triage saves her hours a day — her own experience with her inbox.
- Creating an automation by conversation did not reliably show the expected popup; the manual path makes it explicit (task, plugin, frequency, project).
- **Check the destination, not just the "done" message:** she opens the spreadsheet to confirm the change actually landed.

### Code review as a first pass · 26:59
- Delegating more work creates a review problem: many agents can outproduce one person's reading speed.
- Review entry points: GitHub pull requests (inline findings with `P0`, `P1`, `P2`), `/review` in the app or CLI, and a Codex plugin inside Claude Code.
- VB chooses to review uncommitted changes; Codex opens a dedicated review thread with a review system prompt and reads the diff in the context of the monorepo.
- He reports that 100% of employee PRs across OpenAI repositories get Codex review by default, and calls it the first pass (while acknowledging his bias).
- The demo returned P1 localization and P2 translation findings; fixing them or opening a follow-up PR is a separate task.

### Subagents: review slices and personas · 32:35
- **Subagents** let a coordinating agent delegate independent parts of a task and collect the results; API or subscription capacity limits concurrency.
- VB's example: roughly 40–50 persona files (documentation reviewers, test creators, test runners, an accessibility reviewer, an architect) need review after a spec change.
- He asks for 20 subagents; Codex finds 45 curated persona TOML files and plans 20 review slices, displaying a five-task checklist (plan mode).
- A configured limit of six concurrent threads means six agents launch first — that limit is his setup, not a universal rule.
- Each assignment carries a role, the exact files, and pointers to repo guidance. Completed agents are torn down and feed findings back to the main thread.
- Roles differ: read-only for reviewers and vulnerability investigators, write access for documentation and bug-report authors. The audit even flagged an over-privileged persona.

### Hooks, guardian approvals, and the cloud trust boundary · 49:29
- **Guardian approvals** (experimental, via `/experimental`) add a reviewing agent for privileged actions such as deleting a directory, starting a server, or exposing a file to the internet — aiming to reduce approval fatigue. The live attempt did not produce a clear result.
- **Hooks** attach scripts to lifecycle events (after tool use, session start, stop); a stop hook can ask a long-running task to make one more pass, run one validating command, then stop.
- Personalization sets communication style and persistent preferences; Codex Security extends review to application vulnerabilities with Codex-assisted patches.
- Q&A: point Codex at previous sessions to suggest automations or subagents. Cloud execution supports best-of-N — running one task several times and choosing an output (four runs in her example).
- **Trust boundary:** cloud tasks can read checked-in repo instructions and resources but do not support packaged skills, because a skill may bundle a Python script or executable the sandbox has no basis to trust. The practical fix is to ask the cloud task to consult the relevant repo resources explicitly.

## Resources

- [Introducing GPT-5.3-Codex-Spark](https://openai.com/index/introducing-gpt-5-3-codex-spark/)
- [Configure Codex subagents](https://learn.chatgpt.com/docs/agent-configuration/subagents)
- [Install and use plugins](https://learn.chatgpt.com/docs/plugins)
- [Configure lifecycle hooks](https://learn.chatgpt.com/docs/hooks)
- [Codex fast mode](https://learn.chatgpt.com/docs/agent-configuration/speed)
- [Codex plugin for Claude Code](https://github.com/openai/codex-plugin-cc)
- [Codex Security research preview](https://openai.com/index/codex-security-now-in-research-preview/)
