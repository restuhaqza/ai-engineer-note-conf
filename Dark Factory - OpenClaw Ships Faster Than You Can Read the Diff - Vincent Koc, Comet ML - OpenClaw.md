# Dark Factory: OpenClaw Ships Faster Than You Can Read the Diff

| Field | Details |
| --- | --- |
| **Speaker** | Vincent Koc |
| **Affiliation** | Comet ML / OpenClaw |
| **Type** | Talk |
| **Date** | 2026-06-05 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=pmoDeA3RBZY) |
| **Tags** | `infrastructure-and-deployment` `leadership` `reasoning-and-models` `coding-and-developer-tools` |

## Summary

OpenClaw maintainer Vincent Koc describes what he calls a "dark factory": a way of working where developers supervise many parallel coding agents instead of authoring every change by hand. The pace is real but unruly — he gets rate-limited by GitHub hourly and has seen the project peak at around 800 commits a day with roughly 10–15 core maintainers, all keeping day jobs.

His central argument is that raw output is not the goal. Burning tokens or maximizing commits says nothing about quality, so the real engineering work moves to process: allocating tasks across agents, noticing when a session needs intervention, and keeping reusable skills sharp. Returning to the industrial metaphor, he says the bottleneck shifts from the weaver's hands to human taste — judgment about what should be built.

The talk walks through a concrete example, the "Great Refactor," where the team cut the core down and moved responsibilities into a plugin architecture. It also shares unglamorous lessons, like why dozens of live Git worktrees overwhelmed his machine. His closing line: 2025 was about maximizing tokens; 2026 is about not wasting them.

## Key Takeaways

- **Process is the product.** The hard part is directing, questioning, and checking agent work, not the model choice.
- **More output is not the goal.** Commits and token burn are poor objectives on their own.
- **Supervise with swim lanes.** Split work (tests, features, urgent bugs) into lanes that need different levels of attention.
- **Read the explanations, not the internals.** An agent that rambles or cannot account for its work is a warning sign.
- **Keep the core small.** In a world of cheap code, the scarce decision is whom to refuse; plugins separate responsibilities.
- **Reuse working instructions.** Maintain skills like dotfiles and improve them from real session logs.
- **Isolation has a cost.** Dozens of simultaneous Git worktrees can crush a heavy test harness; separate clones may be simpler.
- **Engineer the intake.** Deduplicate incoming work with embeddings and a semantic graph to decide what deserves attention.

## Detailed Notes

### Waking up to another advancement · 00:16
- The opening meme captures the maintainers' reality: wake up, find a new advancement, repeat.
- Koc is a core OpenClaw maintainer working alongside Peter, and he has a day job too.
- The practical question he sets out to answer: what actually makes this shipping velocity possible?
- His answer starts with engineering the process, not luck.

### The cost of working at the frontier · 01:05
- In 2013 he tried an early VR headset. Its box said to use it for five minutes; he played for three hours and then was sick for three hours.
- The point: anything at the edge is janky and brings unfamiliar failure modes.
- OpenClaw's pace has rough edges too — he hits GitHub rate limits almost hourly.
- At the edge you deal with these constraints while the tools and practices are still forming.

### From writing code to managing production · 02:18
- Britain moved from hand production to mills. Software teams now face a similar shift in how work is organized once much more can be produced.
- His day job is careful and measured: evals, telemetry, structured behavior. On OpenClaw he places real trust in the harness (the machinery that runs and checks agent work).
- Those two worlds are converging.
- Where the weaver's hands once limited output, engineers now manage the production process — and Koc names taste as the new bottleneck.

### What the throughput figures mean · 03:39
- Public acknowledgment of autonomous agents lags private experimentation, much like early ChatGPT use.
- He cites Anthropic's C compiler project as a visible example: a bounded Opus 4.6 experiment with human-designed tests, with remaining GCC dependencies and correctness limits.
- He repeats, with qualification, a claim that Spotify developers no longer write code by hand; it should not be read as established company-wide practice.
- He invokes Steve Yegge's "Vibe Maintainer," noting the PR volume there concerns incoming contributions, not solo authoring.
- Koc reports OpenClaw peaked at 800 commits per day with 10–15 core maintainers, and close to 3,000 personal commits on March 15.
- These are differently scoped activity counts, not a quality benchmark. His commits stop when he sleeps.
- The operational question survives: if this volume becomes normal, how much of reading every PR remains workable?

### Beyond maximizing commits and tokens · 05:28
- One response is simply to maximize commits, which he links to Ralph looping: hand an agent a task, let it burn tokens for hours, and hope for a result.
- The waiting is part of the problem. A long run spends resources without guaranteeing anything useful.
- He jokes about "Bart looping" as a more opinionated alternative, then asks the real question: what should the reward mechanism be?
- No new algorithm is given here. The requirement is a better objective than continued execution.

### Concurrent agents on a moving codebase · 06:12
- At NVIDIA, working on NemoClaw, he and Peter ran multiple Codex sessions. Peter also dialed into a Mac Studio at home over VPN.
- He estimates about 15 sessions for Peter and another 10–15 for himself, maybe 60–70 agents with subagents, and roughly 15 foreground swim lanes.
- While they worked, another maintainer moved channel integrations (like Microsoft Teams and Slack) to a new location in the codebase.
- Work already in flight had to accommodate that change. The challenge was not just task distribution: other maintainers could move the ground underneath those tasks.

### The Great Refactor · 07:33
- Contributors wanted features, but accepting every request would turn cheap code generation into expensive bloat.
- The scarce decision was whom to refuse, so the team chose to shrink the core and split responsibilities with a plugin architecture.
- A provider piece, for example, could be owned by OpenAI, Mistral, or Anthropic. That was a possibility, not a claim those vendors had taken it on.
- They decided to refactor late at night. Koc reports about 2,700 commits, nearly a million lines changed, and 82% of the core touched before plugins launched.
- Near the end he tried to sleep while tests still failed.
- The saving grace was AI-generated unit tests that were tightly coupled to the old implementation. He usually treats that overfitting as a weakness, but during the rebuild those tests gave a concrete regression signal: going green suggested the new system approached the old behavior.
- Passing tests gave limited confidence, not proof of correctness.

### Assigning attention across swim lanes · 09:22
- His factory is many Codex sessions organized into swim lanes, whose number grows or shrinks with the work.
- The count matters less than the division of responsibility. Different lanes get different levels of attention.
- Lanes 1–2: refactor tests in a stable codebase. Just pass, commit, and push; little supervision.
- Lanes 3–4: Docker or messaging-channel features. These are conversations: agents investigate, report, and continue with his input.
- Lane 5: new P0/P1 issues. Monitor and surface urgent work. Agents in Discord report what happened in the last two hours.
- At this scale, raw compute and his own capacity to track sessions are more limiting than tokens. Adding a lane still costs attention even when usage is affordable.

### When workspace isolation becomes overhead · 11:04
- Trusting the harness does not require a complicated workflow, but isolation can get expensive.
- He created a Git worktree for every PR he touched. A heavy test harness spread across many workspaces overwhelmed his machine.
- He reports roughly 70–80 active worktrees in a day. He had to build custom tooling so his Codex session knows about worktrees and can self-recover.
- In hindsight he would have preferred the simpler setup others use: several independent repository clones, one per session.
- This is a lesson from his workload, not a universal rule against worktrees. His ordinary workflow is conversational; he does not use plan or spec mode.
- To spot trouble, he compares watching agent streams to reading the Matrix: after enough exposure, patterns appear.
- His warning signal is usually how an agent explains its work. It rambles, becomes incoherent, or cannot account for what it is doing.
- The response is like managing a colleague whose account no longer makes sense: ask what is going on, then reconsider. He may end the session, leave that area to another maintainer, or return days later.

### Maintaining the agent development environment · 13:21
- The repeatable part of his practice is what he calls an agent development environment.
- He manages `.skills` much like dotfiles: a maintained collection of reusable instructions, some public, some private.
- Skill improvement can use a skills gym or GEPA. His more direct loop is grounded in real work:
- Ask Codex to inspect the last two weeks of session logs for a skill.
- Use those sessions to find and make improvements.
- Deploy the revised skill into OpenClaw or his personal environment.
- He mentions skills.sh as part of that loop, and adds that distribution is not proof of improvement — testing is separate.

### Selecting work and checking behavior · 14:28
- Incoming work gets the same treatment. Each new maintainer jokes about tackling the PR backlog by clustering it.
- During the talk an initial figure of 6,000 is corrected in the room to 60,000; the exchange does not settle whether that means open PRs or a cumulative count.
- The clearer problem: many contributors submit their own version of the same issue.
- He uses vector embeddings and a semantic graph over GitHub material to inspect related submissions and expose duplication.
- Repeated submissions around one problem can also signal demand — a reason to give it a maintainer's time.
- After the refactor the team built a fake Slack-like environment with synthetic and real models to run evaluations that check providers and channels work together.
- Asked how he manages 10+ agents, he asks back how you would manage 10+ staff. Leading large teams made parts of this familiar.
- The soft skills matter: asking for status, noticing shaky explanations, and coordinating sessions with different assignments. The process, not the model, is now central. His close: 2025 was token maxing; 2026 is token efficiency, with the agent in the loop.

## Resources

- [Building a C compiler with a team of parallel Claudes](https://www.anthropic.com/engineering/building-c-compiler) — Anthropic's account of coordinating parallel agent work.
- [Vibe Maintainer](https://steve-yegge.medium.com/vibe-maintainer-a2273a841040) — Steve Yegge on maintaining open source amid AI-generated contributions.
- [NVIDIA's NemoClaw announcement](https://nvidianews.nvidia.com/news/nvidia-announces-nemoclaw) — the platform referenced in the talk.
- [Vincent Koc's dotfiles](https://github.com/vincentkoc/dotfiles) — his public development-environment configuration.
- [GEPA](https://github.com/gepa-ai/gepa) — reflective optimization tooling for prompts, code, and agent behavior.
- [Agent Skills Directory](https://www.skills.sh/) — directory for discovering and installing reusable agent skills.
