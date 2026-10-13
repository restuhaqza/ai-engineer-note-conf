# Fast Models Need Slow Developers

| Field | Details |
| --- | --- |
| **Speaker** | Sarah Chieng |
| **Affiliation** | Cerebras |
| **Type** | Talk |
| **Date** | 2026-05-22 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=TeGsFFNqRLA) |
| **Tags** | `rag-context-and-search` `infrastructure-and-deployment` `coding-and-developer-tools` |

## Summary

When AI code generation was slow, developers built workarounds: huge prompts, one-shot features, giant commits, and many agents running at once. Sarah Chieng argues that these habits become dangerous now that coding models are much faster. If a workflow produced bad code slowly, speed just produces bad code faster.

She points to **Codex Spark**, a coding model reported at around 1,200 tokens per second, versus roughly 40–60 for the Sonnet and Opus families. That speed comes from optimizations across the whole inference stack — hardware, model design, and serving software — and it changes how developers should work.

Her playbook leans on deliberate pacing: plan with a larger model, execute bounded steps with a fast one, validate and refactor constantly, and steer the agent in real time instead of leaving it unattended. She also treats rapid generation as a way to generate options and inject human taste.

The bigger message is that speed should buy better feedback loops and more human judgment, not simply more unreviewed code.

## Key Takeaways

- **Speed amplifies habits.** Faster models scale bad practices just as easily as good ones.
- **Don't optimize for agent count.** Six terminals or a 500-agent swarm is not the same as understood, verified work.
- **Speed is a third selection axis.** Choose models on intelligence, cost, and now latency.
- **Plan slow, execute fast.** Use a larger model for the plan and a fast one for bounded steps.
- **Validation becomes cheap.** Run tests, linting, diff reviews, and browser QA at every step, not at the end.
- **Generate options, not slop.** Many variants let you select with taste instead of accepting the first result.
- **Context fills faster too.** Break work into small goals before 80–100% context occupancy triggers compaction.
- **Persist memory in files.** `AGENTS.md`, `plan.md`, `progress.md`, and `verify.md` carry continuity across sessions.

## Detailed Notes

### Slow-model habits amplify with fast models · 00:17
- Familiar slow-generation workarounds: massive prompts, one-shotting a feature, huge commits, ten agents thinking at once.
- **Codex Spark** is reported at about 1,200 tokens per second; Sonnet and Opus families around 40–60.
- Chieng frames that as roughly a 20× difference, based on her historical comparisons rather than a controlled benchmark.
- The core warning: speed does not fix the feedback loop around generation.
- Over the past two years, popular model families sat around 50–150 tokens per second even as models got smarter and contexts grew.
- Her day job is introducing fast inference to developers for the first time, which surfaces the question of how to work when code outruns comprehension.

### Why inference got faster: memory, prefill/decode, and pruning · 03:15
- The speedup comes from optimizing hardware, model architecture, and inference software together.
- The **memory wall** is central: moving weights and KV-cache values can dominate the time per token.
- Chieng attributes 50–80% of inference latency to hardware and memory movement (no workload specified).
- NVIDIA GPUs read from off-chip HBM (high-bandwidth memory); designs like Cerebras put memory near compute, with SRAM spread across the wafer.
- **Disaggregated inference** splits a request into two phases so each can run on the right hardware.
- **Prefill** processes input tokens and fills the KV cache — parallel and compute-bound. **Decode** generates output token by token — sequential and memory-bound.
- A **mixture-of-experts** model activates only a subset of experts per token, separating total capacity from per-token compute.
- **REAP** (Router-Weighted Expert Activation Pruning) removes experts entirely, ranking importance by router gate values and activation norms.
- At the serving layer, **KV-cache reuse** avoids recomputing representations for the whole preceding sequence.

### Speed is now a third model-selection axis · 07:05
- Online coding culture shows six Claude Code terminals, swarms of 500+ agents, and eight agents across five screens.
- Chieng's concern is code that nobody verifies: faster production also raises the rate of technical debt.
- With a 20× speed gap, latency joins intelligence and cost as a dimension for choosing models.
- **Planning and long-horizon work:** use a larger model such as GPT-5.4 or GPT-5.3 to produce the plan.
- **Bounded execution:** hand Codex Spark specific, clearly scoped steps in a defined order.
- The point is to give a fast executor something specific, not to make every agent an independent project owner.
- A successful hard session can become a reusable **skill**: capture the working trajectory, then let a smaller fast agent repeat it in the background.

### Spend the saved waiting time on validation · 09:57
- Chieng calls validation "basically free" — that means less generation waiting, not zero cost. Tests and human review still take time.
- The useful change is that checks can run inside each development step instead of a final phase.
- **Test suites:** check behavior against expected results.
- **Linting and pre-commit hooks:** catch recurring mechanical problems before a commit.
- **Diff reviews:** look at what the agent actually changed.
- **Browser-based QA:** inspect behavior where users encounter it.
- Smaller changes make each check easier to interpret and each correction easier to contain.

### Use variety to inject human taste · 10:47
- Example: ask for a midnight-blue nav bar with four icons and get one acceptable, not ideal, result.
- Instead, request 15 variants in the time an older model produced one (an illustrative comparison, not a benchmark).
- Push further: five agents × 15 variants yields 75 candidates to choose from.
- These are alternatives to compare, not changes that all need to ship — unlike a pile of unreviewed implementation.
- This helps with research directions, architecture options, and graphic design.
- Models lack reliable taste; generating options shifts effort from writing detailed specs to making judgment calls on concrete candidates.

### Stay present: steer, constrain, and clean up · 12:07
- The old pattern — start a session, walk away, return when done — can give way to real-time collaboration.
- Ask the agent to gather context across the repo and explain how the system works, then make the implementation decisions yourself.
- Chieng's rule of thumb: the AI helps you decide; you remain responsible for the decision.
- Steer two or three sessions actively rather than leaving ten unattended.
- Useful boundaries: ban file deletion, set a maximum diff size, limit operations to read/write, and give steering directions mid-task.
- Narrow instructions as you go: change only the relevant area, postpone type changes, or stop and redo as soon as an approach looks wrong.
- Apply the same short cycle to refactoring: after each checklist task, run a cleanup pass for unused imports and consistent structure.

### Context management and external progress files · 14:30
- Faster generation also fills context faster: Chieng's illustration assumes 20× accumulation, so a 10-minute window fills in about 30 seconds.
- Break large tasks into smaller, bounded goals before the session gets crowded.
- She advises avoiding 80–100% context occupancy because compaction can lose information (her working heuristic, not a fixed threshold).
- Put durable memory where a fresh session can read it, using four files:
- `AGENTS.md` — project guidance and agent/subagent responsibilities.
- `plan.md` — the overall plan and ordered checklist.
- `progress.md` — completed work, remaining work, and where to resume.
- `verify.md` — the checks required before advancing.
- In her setup, a planner model (GPT-5.3/5.4 Codex) writes the plan and a fast executor (Codex Spark) works through the checklist one item at a time.

### Make careful development sustainable · 16:54
- Closing slide gathers concrete Codex controls: `/permissions`, `/experimental`, `/skills`, `/review`, `/rename`.
- It also lists `/new`, `/resume`, and `/fork` for managing conversations.
- These were the controls shown in the recording, not a guarantee of availability in every client.
- They tie back to permissions, reusable procedures, review, and deliberate session boundaries.
- The developer-experience payoff: quality work without turning every question into a long wait.
- Faster feedback makes it practical to stay with the code, inspect a small change, request a revision, and verify while the problem is fresh.

## Resources

- [Introducing GPT-5.3-Codex-Spark](https://openai.com/index/introducing-gpt-5-3-codex-spark/) — launch account of the fast coding model.
- Stop Shipping AI Slop (related) — Sarah Chieng and Sherif Cherfa's practical guide to a fast coding workflow.
- [Cerebras is coming to AWS](https://www.cerebras.ai/blog/cerebras-is-coming-to-aws) — partnership announcement covering splitting prefill and decode.
- [REAP the Experts](https://arxiv.org/abs/2510.13999) — research on compressing mixture-of-experts models by pruning experts.
- [REAP implementation](https://github.com/CerebrasResearch/reap) — official code and experiment instructions.
- [Custom instructions with AGENTS.md](https://learn.chatgpt.com/docs/agent-configuration/agents-md) — giving agents persistent repository instructions.
- How KV caching works (related) — an explanation of how autoregressive transformers reuse cached token representations.
