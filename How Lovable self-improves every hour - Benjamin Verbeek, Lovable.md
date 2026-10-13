# How Lovable self-improves every hour

| Field | Details |
| --- | --- |
| **Speaker** | Benjamin Verbeek |
| **Affiliation** | Lovable |
| **Type** | Expo session |
| **Date** | 2026-06-02 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=KA5kPbdkK2E) |
| **Tags** | `infrastructure-and-deployment` `coding-and-developer-tools` `developer-workflows-and-testing` `reasoning-and-models` `evals` |

## Summary

Benjamin Verbeek, a member of technical staff at Lovable, explains how the platform learns from its own users. Lovable lets people describe software in a chat and watch it appear in a sandbox, and it deliberately targets non-technical users. When one of those users hits a hard blocker, they tend to abandon the project before ever reaching a successful result. So the goal is to make blockers disappear.

Two feedback loops do the work. The first turns stuck sessions into reusable knowledge. An LLM (large language model) judge flags likely stuck sessions; when the user and the agent eventually find a fix, Lovable asks what could have been injected at the start to skip the failed detours, clusters similar cases, and evaluates candidate entries. A lightweight model then injects that guidance into later sessions, and a holdout group that receives nothing measures whether it genuinely improves project completion. Entries that stop helping are retired.

The second loop lets the working agent complain. A "vent" tool sends material frustrations to Slack: missing tools, unclear schemas, conflicting docs and platform failures. Roughly twenty complaints in the first hour exposed a copy bug involving spaces in filenames; the first fix missed non-breaking spaces, and further reports drove it to completion. A monitoring agent now deduplicates, investigates and drafts pull requests, which developers still review before merging. Verbeek frames the unfinished work as closing the loop into fully automated, evaluated continuous improvement.

## Key Takeaways

- **Lovable's aim is that a mistake is explained once and never again.**
- **Non-technical users quit at hard blockers,** so "can a user finish the project?" is the real reliability metric — not "does the agent usually make progress?"
- **An LLM judge can spot stuck sessions** from repeated requests, complaints about a bad fix, explicit failures and abandonment.
- **Successful recoveries become knowledge entries,** clustered to avoid overfitting, then validated by holding the guidance back from some eligible projects.
- **Knowledge is retired when models or features change.** Stale context can actively hurt the agent.
- **A vent tool gives the working agent an outlet** for missing tools, unclear contracts, conflicting docs and platform failures.
- **Repeated complaints drive a real fix.** A subtle copy bug with non-breaking spaces only got solved once agents kept reporting it.
- **Vent spikes double as incident detection,** and automated patches are still human-reviewed before production.

## Detailed Notes

### Why explain the same mistake twice? · 00:23
- Verbeek, a member of technical staff at Lovable with a physics background (satellites, particle physics, fusion reactors), frames the talk around **continuous learning at scale**: a mistake should happen once and never again.
- The Lovable interface pairs a chat with a sandbox preview: describe software, see it, test it, ship it. He loosely credits Lovable with the term "vibe coding" — building without reading the code.
- Users tend to stay in one long project, which gives the agent unusually rich context about both the artifact and the person, unlike short, unrelated chat sessions.
- The target audience is the "99 percent who can't code" (his framing of the ambition, not a measured figure); like startups adopting new technology, they push until it works.

### A blocker can end a non-technical user's project · 03:24
- When he joined about a year earlier, Lovable had a few thousand users; by the talk it was creating more than 200,000 projects per day — project creation, not published sites. Scale brought both learning examples and operational pain (GitHub banned Lovable on day one for creating too many repositories).
- With AI, a technical user alternates between big speed-ups and friction. Yellow friction yields to a more precise prompt; a red blocker may need a config edit, a setting change, an environment variable or an API key — recoverable for a developer.
- A non-technical user may prompt past the yellow, but quits at the first red one and never reaches a successful result.
- So the reliability target is whether the user can finish the project, not merely whether the agent usually makes progress.

### Detect a stuck session, then choose the right fix · 05:22
- Signals of being stuck: asking for the same thing again, complaining about an implementation, explicitly reporting failure, or abandoning a session they would otherwise continue. An **LLM judge** can scan sessions and flag likely cases. A single request to improve something is not, by itself, a stuck signal.
- He splits stuck into three classes: solvable with the current product (fix the guidance), unsupported but straightforward (fix the product), and unsupported and hard (real engineering work).
- He argues the first two deserve fast action: if a task is solvable it should work for everyone, and if a fix is simple, ship it.

### Turn a recovery into reusable knowledge · 06:56
- The "Stack Overflow for Lovable" idea: collect problems and solutions the agent can reuse. Example — a user's site is laggy while scrolling; the agent quantizes animations, which leaves it jumpy as well as laggy; the repeated request plus complaint makes it a clear stuck case. The real cause turns out to be individual gradients on overlay text.
- The useful moment is the transition from stuck back to resolved (carefully distinguished from the user simply giving up), which yields a concrete question: what could have been injected at the start to reach the solution directly?
- Lovable clusters similar issues so entries capture the common problem rather than one exact prompt; an external reviewer (usually an agent, sometimes a human when there is uncertainty) generates and runs a quick eval against the cluster's examples.
- Accepted entries feed a continually updated problem-and-solution bank.

### Knowledge has to keep earning its place · 09:46
- A lightweight model watches the main agent's work and injects a relevant knowledge entry when it finds one.
- For a small sample of eligible projects the system deliberately injects nothing, creating a comparison group. The team then compares outcomes: projects that received guidance should succeed more if it helps, and guidance linked to worse outcomes should be used less.
- The metric is overall project success, not whether the injected text sounds helpful. The talk does not give holdout sizes or allocation details.
- **Knowledge retirement is part of learning.** New models, new features and obsolete workarounds make entries stale quickly, and stale context can hurt the agent (sometimes called context rot). Entries are rebalanced and discarded as the surrounding system changes.
- He reports fewer fix-intent or stuck messages and more deployments in early data (without effect sizes), and says all models at the top of Lovable's internal ranking use the knowledge bank.

### Give the working agent a vent tool · 12:00
- Knowledge cannot fix a missing capability, so a second loop lets the agent report problems to its creators — analogous to an employee complaining to a manager.
- He argues against an external reviewer that must find something to say on every iteration (low signal-to-noise, overfitting to noise); instead the working agent is prompted to report only real frustration, and that threshold can be tuned.
- The **vent/send feedback tool** covers several categories: missing or unsuitable tools; unclear tool names, parameters or schemas; confusing or conflicting docs and instructions; and broken platform behavior or repeated environment failures.
- The working agent has context the user lacks — it already attempted the task and observed the tools — so asking it to report preserves that detail at the moment the friction happens.

### From a Slack complaint to a reviewed fix · 14:34
- One complaint concerned Framer Motion's TypeScript types: the agent wanted to pass four numbers for what Verbeek thinks was a cubic Bézier curve and objected to the required casting. He questions whether the complaint was relevant, while noting the use case could have been simplified and that the conversational form is easy for engineers to read.
- The copy-tool bug: the agent reported that copying certain uploaded files failed when the filename contained a raw space. Engineers' own checks said the tool worked. Roughly twenty complaints arrived in the first hour after the feedback tool launched.
- The first fix replaced spaces with underscores, but reports continued. The team then found **non-breaking spaces** (from WhatsApp or macOS screenshot filenames) that the regular expression had missed. Further special characters followed until the issue stopped.
- Tool telemetry could show a failure, but the complaint described the circumstances closely enough to act on — that is the added value here.

### The loop that isn't closed yet · 18:04
- The first version was direct: the agent hits an issue, calls the vent tool, and the report lands in a restricted Slack channel. He worried about spam; the head of product read every message.
- A monitoring agent now deduplicates, investigates and drafts pull requests, but developers still review and often merge them, so automation has not become autonomous production approval.
- Vent spikes turned out to be an **incident signal**: outages and broken sandboxes make many agents complain at once, and the reports hint at which functionality is affected.
- Collecting feedback inline is cheap: the working agent has already processed the context with a strong model, whereas an equally capable external reviewer would reread a large context.
- A last piece of feedback came from the agent about the vent tool itself — it was too easy to send and impossible to retract. Verbeek closes by inviting others to help close the loop of detect, fix, and continuously review and evaluate.

## Resources

- [Motion easing functions](https://motion.dev/docs/easing-functions)
- [Lovable's introduction to vibe coding](https://lovable.dev/blog/what-is-vibe-coding)