# How Google DeepMind Runs Agents at Scale

| Field | Details |
| --- | --- |
| **Speaker** | KP Sawhney, Ian Ballantyne |
| **Affiliation** | Google DeepMind |
| **Type** | Talk |
| **Date** | 2026-05-24 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=7gujZrJ9L5I) |
| **Tags** | `observability-and-reliability` `apis-mcp-and-protocols` `developer-workflows-and-testing` |

## Summary

Ian Ballantyne and KP Sawhney run an audience-interactive panel about how Google DeepMind builds and operates agents internally. Ballantyne (developer relations) opens with a live demo of Google Antigravity, a coding environment that looks like an IDE but has an agent manager behind it. Sawhney (who works on DeepMind's AI platform team) then fields questions about running these systems at Google's scale.

The demo sets the theme. The agent is given a written spec, plans its work, opens a browser it can control, inspects the page structure, and writes a report of what changed. The interesting part is not that it can build a game, but where a person can step in — editing the plan before work proceeds, or reading the agent's notes mid-run.

The scaling discussion is mostly about cost and control. A single request can trigger a lot of downstream work, so DeepMind uses per-user and per-team quotas, mixes cheaper models with frontier ones, and relies on custom tracing tools. The panel also covers how to evaluate skills, MCP versus command-line tools, quota messages that look like task completion, and why code review becomes the next bottleneck.

## Key Takeaways

- **Antigravity is more than an IDE.** Behind the editor is an agent manager that can spawn agents across projects.
- **The plan is editable.** A person can change a line or leave a comment before the agent proceeds, correcting intent early.
- **Token demand drives operations.** Per-user and per-team quotas are the current blunt control against runaway usage.
- **Mix models on purpose.** Cheaper/open models such as Gemma 4 run on local GPUs or TPUs, while frontier models handle only the components that need them.
- **Observability is custom-built.** Hierarchical tracing follows a query down to the raw model request; an agent trajectory store finds where a run started looping.
- **Evaluating skills is hard.** It needs sandbox environments, new datasets, and tests written by the skill's author; agents designing those tests is still experimental.
- **Skills over MCP, for now.** Sawhney prefers skills plus guardrailed CLI (command-line interface — commands typed in a terminal) use, and questions MCP's (a standard way for agents to call tools) long-term role, though he values its authentication.
- **Review is the next bottleneck.** Auto-review models and tools like Jules help, but producing more code means there is more to check.

## Detailed Notes

### The live demo: spec in, browser out · 00:16
- Ballantyne opens with a show of hands: only a handful of people have used Antigravity.
- He attaches a spec file and asks Flash (a Gemini model) to implement it; the first attempt errors, so he restarts.
- The agent discovers existing generated files and uses built-in planning and to-do features to decide what still needs changing.
- It launches a browser that Antigravity controls, reads the DOM (the page's structure), and feeds that information back into its work.
- A completion report lists what changed, with screenshots or a recording as review artifacts.
- Before more changes run, the plan appears as an editable list: inspect the proposed diff, edit or comment on a line, then press proceed.
- Later in the demo a blue border shows the agent has control of the browser. It rebuilt the game from the spec — the earlier files came from a different model, so it is not even the same game.
- The agent's "scratchpad" holds notes on the behavior it is working through, giving a trace a person can inspect or interrupt.

### Extend the harness beyond coding · 04:32
- Sawhney worked on the Deep Research agent, available through the Interactions API (a way for developers to call the agent).
- His focus has shifted to improving the Antigravity harness internally, including its use against Google's huge monorepo (one large shared code repository).
- One idea for research: instead of passing huge blobs of search text between stages, let stages collaborate through a shared filesystem.
- Internally there is a large, growing library of skills (instructions that give agents specialized know-how); the risk is that it sprawls out of control.
- The stated goal is letting only the best skills survive, but how to evaluate them is an open question.

### Scale starts with controlling demand · 07:49
- At Google scale, the first concern is how many tokens these workflows consume.
- Per-user and per-team quotas are the central operational controls.
- Lowering cost also means deciding which components actually need the most capable model.
- Sawhney suggests mixing open models such as Gemma 4, running on available GPUs or TPUs, with advanced models for selected parts. He calls local inference "effectively free" in the sense of hosted-model quota — not hardware or operating cost.
- Evaluation adds another load; the team is looking at mock TPUs to exercise the harness and agent flow without spending real TPU hours.
- Audience examples of scale: a radiology project combining EHR (electronic health record) and radiologist data, and an open-source memory layer built on an ontology-driven graph.
- The memory project's participant reports about 15,000 stars and roughly 3 million graph generations per month — self-reported figures, not verified counts, and not a user count.
- Ballantyne's thought experiment is a power user running a team of 100 agents. Sawhney's answer is blunt quota enforcement; some power users are eventually told to stop.
- He cites Anthropic blocking OpenClaw as a sign that subscription economics struggle with token-hungry agents.
- Ballantyne recalls asking how you know a job uses too much, and being told "they'll tell you" — SRE (site reliability engineering) teams watch for spikes and ask engineers to stop a job.

### Find the step where the run went wrong · 13:06
- Asked about observability, Sawhney describes a custom web app connected to an agent backend used for many Google workloads.
- When a user submits a query, it appears automatically in the UI, and engineers can drill down the hierarchy all the way to the raw prediction request sent to the model.
- A separate agent trajectory store focuses on coding, where a single run may contain a huge number of steps.
- Its purpose is to pinpoint the exact step where the model started looping or diverged from the task.
- Both are custom internal tools for now; an audience member says they simply export everything to a log file.

### Make research stages collaborators · 14:36
- Asked whether Deep Research will come to Antigravity, Sawhney says it is being actively explored, with no release commitment.
- The hoped-for benefits are faster execution, lower cost, and better results from orchestrating research through the same harness.
- The motivation is context volume: Deep Research passes a huge amount of material through its pipeline, which is expensive and consumes context capacity.
- A shared workspace could let each stage behave more like a collaborator in a human research team.
- It could also yield outputs beyond a final report, such as infographics and supporting documents.
- On nested sub-agents, Ballantyne does not know the depth limit. Tracks can run in parallel, but which agent does what is fairly opaque.
- Sawhney's long-term view: make agent-to-agent communication efficient, and give the human the ability to supervise, like overseeing a digital assembly line.

### Skills, MCP, and how to test them · 17:22
- The question is where skills, MCP, and CLIs are heading as community preferences keep changing.
- Sawhney favors skills plus guardrailed CLI interactions. He is skeptical MCP will last, while valuing its authentication.
- His example is a skill for debugging raw logs that does most of its work through the CLI.
- The organizational win is who writes the skill: at Google, a specialist can encode their knowledge so both colleagues and their agents get it "for free."
- Ballantyne notes Antigravity supports both skills and MCP, and that community usage will guide continued support — this is not a plan to drop MCP.
- On evaluating skills, Sawhney lists obstacles: setting up sandboxed environments per problem set, creating new datasets for specialized skills, and the author's responsibility to provide a test.
- People are experimenting with agents designing evaluations, but that is still work in progress.

### A quota message is not task completion · 20:06
- An AI Ultra subscriber says browser testing is great when it works, but retries and apparent network failures make it hard to tell whether quota ran out.
- Ballantyne says internal users have worse limits because customers get priority. He expects tier-specific limits to persist.
- He describes a proposed pattern, driven by user preference: if capacity runs out on Pro, move to Flash; if the whole subscription is exhausted, continue on a local model.
- The key requirement: hitting a quota limit must look visibly different from finishing the task.
- He imagines sending a job away and returning to find it did nothing for an hour because it hit a limit; clearer signaling is the goal.

### Close the loop with review · 22:24
- The final question asks how review can keep up with generated code, and whether models could learn from past Google review comments and commits.
- Sawhney describes an existing two-layer system: per-language auto-review models fine-tuned on internal style guides and good code examples, then product-area-specific instructions and prompts.
- The first layer captures language conventions; the second tunes the signal reviewers on a given product receive.
- He gives a recent example: a PR he submitted the day before got a useful comment from an agent someone had deployed, without him triggering auto-review.
- Ballantyne connects this to Jules, a public web interface for working on users' own GitHub projects, and closes on the pressure of ballooning agent-generated code: producing more software expands the work of checking it.

## Resources

- **Introducing Google Antigravity** — Google's introduction to Antigravity's agent manager. *(mentioned)* — https://antigravity.google/blog/introducing-google-antigravity
- **Build with Gemini Deep Research** — Introduces the Deep Research agent through the Interactions API. *(mentioned)* — https://blog.google/innovation-and-ai/technology/developers-tools/deep-research-agent-gemini-api/
- **Gemma 4 launch and deployment options** — Gemma 4's license, model sizes, and deployment. *(related)* — https://blog.google/innovation-and-ai/technology/developers-tools/gemma-4/
- **Getting started with Jules** — Connect a GitHub repository, select a branch, and review a change. *(mentioned)* — https://jules.google/docs/
