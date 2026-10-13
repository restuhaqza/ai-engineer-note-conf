# LLM codegen fails and how to stop them

| Field | Details |
| --- | --- |
| **Speaker** | Danilo Campos |
| **Affiliation** | PostHog |
| **Type** | Talk |
| **Date** | 2026-04-30 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=juoNbJiZUi0) |
| **Tags** | `agent-engineering` `rag-context-and-search` `coding-and-developer-tools` `reasoning-and-models` |

## Summary

Danilo Campos builds the PostHog Wizard, a command-line tool that has an agent add PostHog analytics to an existing app. It turns a task he describes as two hours of misery into roughly eight minutes of guided setup, and he says 15,000 people run it each month. The talk is about the many ways that "success" can still be a bad outcome: invented APIs, odd architecture, and unsafe access to user files.

The core problem is trust. A robot is editing code on someone else's machine, so an integration that technically works can still damage a project or leak secrets. Campos walks through each failure mode he has hit and the design change that pushed it down.

His fixes fall into a clear pattern. Give the agent fresh documentation instead of relying on stale training data. Show it a small working example of a good integration. Reveal the work in stages so it does not rush. Ask the agent what went wrong after each run. And keep secrets behind a narrow tool boundary. He argues that much of the Wizard's substance is now plain prose, and that better models will only make well-written instructions more valuable.

## Key Takeaways

- **Model rot is real.** A model is a snapshot of a world that keeps changing, so its PostHog knowledge can be months behind. Fresh context fixes it.
- **Prefer current docs over RAG when context allows.** With today's large context windows, Campos favors loading the right Markdown directly.
- **"Model Airplanes" show shape, not scale.** Small example apps with the integration already done give the agent a pattern without dragging in production complexity.
- **Sequence the work.** The Wizard discovers business-value files, plans events without editing code, then implements. Telling the agent the full destination up front makes it rush.
- **Ask the agent for feedback.** A short question at the end of every run ("what could we have done better?") surfaced broken tools and contradictory instructions.
- **Never read secrets to write them.** A tiny tool that only checks and sets an env key avoids sending existing values to a cloud.
- **Prose is now a core asset.** Campos describes the Wizard as mostly Markdown, and argues good instructions gain value as models improve.
- **Don't over-constrain the agent.** It is more like an octopus than a rigid pipeline: give it good information and useful boundaries, then let it move.

## Detailed Notes

### The integration problem · 00:18
- The **PostHog Wizard** is a CLI (command-line tool) that adds PostHog to an app for you. Campos says it replaces about two hours of work with an eight-minute guided run, and that 15,000 people use it monthly.
- His framing: he is not afraid of robots because they have "already bloodied his nose" many times over. The talk is about learning from those failures.
- The happy path is easy to see. He cites two unprompted posts in the previous six hours from users pleased with the terminal UI.
- The risk is that a good-looking integration can hide invented APIs, poor architecture, or unsafe file access. The Wizard's design is a response to those specific failures.
- These are the speaker's own operating figures; he does not define how usage or timing is measured.

### Give agents knowledge newer than training · 02:06
- **Model rot**: training is a big capital expense, so a model's knowledge is frozen while software keeps moving. Snapshots may be 6, 8, 12, or 18 months old.
- An audience member suggests RAG (retrieval-augmented generation). Campos agrees it helps, but with today's large context windows he prefers loading relevant Markdown directly.
- The Wizard offers tools that let the agent pick from a menu of current PostHog docs, based on what it is integrating and what it finds in the project.
- This started when users asked agents like Cursor to integrate PostHog and got made-up keys and non-existent APIs. PostHog did not produce those answers, but users blamed PostHog.
- His point: serving current context became part of supporting the product, because a codegen failure in someone else's agent became the team's problem.

### Show the shape of a good integration · 04:51
- Correct API knowledge is not enough. An agent can build something workable but structured in a strange place.
- **Model Airplanes** are thin example apps across frameworks and languages with PostHog already wired in. They keep the integration's shape but drop production machinery.
- Auth is the example: a Model Airplane login accepts any password. It is not real auth, but it is "auth-shaped", so it shows the agent where login events and identity tracking belong.
- Benefits: a recognizable app structure to attach to, an integrated pattern to follow, and far less context to spend (which also saves tokens).
- Campos reports this makes the pattern complete more consistently, but gives no measured success rate.

### Discover, plan, then implement · 06:48
- With 15,000 integrations a month, letting each agent improvise would create 15,000 different setups and an impossible support burden.
- To limit improvisation the Wizard **sequences what it reveals**. Giving the agent the whole destination up front makes it blast through the first tasks and over-polish the last.
- Stage 1 — find business-value files: look for login code, a Stripe interface, or signs of churn. Business logic casts a large footprint in code.
- Stage 2 — plan events without touching code: inspect those files, then save event names and descriptions to a file.
- Stage 3 — implement: now load the docs for the detected framework and language, and only then edit the app.

### Ask what made the run difficult · 09:31
- The biggest threat is the team itself: human memory is limited and patchy, so as tools and instructions change, contradictions creep in.
- Real example: one MCP (model context protocol) tool instruction contradicted another, and a required finishing tool did not exist. Hundreds of runs hit the missing tool.
- The fix is cheap: at the end of every run (the stop hook) ask the agent "what could we have done better to set you up for success?" Campos calls this user research where the user is a robot.
- This surfaced concrete defects: an unavailable tool (no permission granted), contradictory directives, and JavaScript instructions delivered to a Python project.
- These were fixable environment bugs that the team would not otherwise have noticed from a "completed" run.

### Let tools edit secrets without revealing them · 12:07
- Running an agent on someone else's machine demands trust. An early Wizard read `.env` files before editing them, which risked sending secret values to cloud logs.
- Reading was not strictly needed. The team locked down file reads around env files and built a dedicated tool with just two operations: check whether a key exists, and set a key to a new value.
- The agent can request the change without ever seeing existing values. Restricting normal file tools is essential, or the agent could just read the secret another way.
- The protected boundary is *existing env content entering inference*, not a claim that nothing leaves the machine.
- His broader rule: don't damage the user's project or betray their trust while technically doing the job.

### Put engineering effort into the information · 14:00
- Developers are trained to solve problems by writing more code. Campos pushes back: a new model does not make yesterday's code more valuable.
- He calls code a **depreciating asset** — it can ship with technical debt and lose value over time, even when shipping was right.
- He describes the Wizard as roughly 90% Markdown, 8% tools for delivering and processing that Markdown, and the rest the agent harness. (His description, not a measured repo count.)
- Investment thesis: better models extract more from the same good prose, so clear instructions, fresh docs, and useful examples keep paying off.
- His metaphor: an agent is an **octopus** — it can squeeze around obstacles. Over-scaffolding removes that flexibility. Keep the safety boundaries, but give enough information, in the right order.

### Shipping the Wizard: delivery and auth · 16:43
- Question: how are the Model Airplanes provided? A context service flattens them into a single Markdown file and references it inside a skill file. The agent can `grep` the full example.
- Why a Model Airplane instead of a plain skill? It is supplemental content inside the skill: docs give prose about the integration, the airplane gives code showing a successful shape.
- The runtime wraps the **Claude Agent SDK** in a CLI. One command, log into PostHog, and inference is free to the user via an LLM gateway that PostHog pays for.
- That packaging brought its own bug: Claude Code sometimes stored auth data in unexpected places and broke the experience. Campos says delivering an agent as a service is still early days.

## Resources

- **PostHog Wizard** (repo) — source and usage guide for PostHog's AI integration CLI.
- **PostHog AI wizard installation guide** (docs) — current install steps and supported integrations.
- **How PostHog's context mill works** (docs) — how docs, prompts, and example code become portable context.
- **PostHog's context-engineering follow-up** (article) — a later account of the Wizard's context infrastructure.
