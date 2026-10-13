# Malleable Evals: Why Are We Still Evaluating Adaptive Systems with Static Tests?

| Field | Details |
| --- | --- |
| **Speaker** | Vincent Koc |
| **Affiliation** | Comet ML / OpenClaw |
| **Type** | Expo session |
| **Date** | 2026-05-12 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=4VhbYlfC7Gs) |
| **Tags** | `apis-mcp-and-protocols` `evals` `agent-engineering` |

## Summary

Vincent Koc works on evaluations at Comet and is a core contributor to OpenClaw. His argument is simple to state and hard to act on: we keep testing adaptive AI systems with static tests. A fixed benchmark, a hand-written set of questions, and an offline run can confirm behavior on the cases you already thought of, but they go stale the moment the agent, its tools, or its users change.

He borrows a familiar framing from software reliability. Normal testing covers known expectations; chaos engineering deliberately breaks a system to find its limits. In his view, AI evaluation mostly stops before that exploratory step, so teams only learn where an agent breaks when production surprises them.

His proposal is "malleable evals": evaluation that moves with the system. Traces of real usage feed an evaluation agent that notices when behavior has shifted, and the test suite is updated to match. He also wants operational signals — failures and cost — inside the loop so a harness can partly self-correct. The end goal is a shift in mindset: evaluation stops being a starting dataset and becomes a specification of the desired end state, something you maintain over time like living software.

## Key Takeaways

- **Static tests drift:** fixed benchmarks and offline tests describe a snapshot, while adaptive agents change continuously.
- **The missing layer is exploration:** teams rarely do the AI equivalent of chaos engineering — actively searching for where the agent breaks.
- **More datasets ≠ more insight:** a benchmark's relevance to your specific application still has to be shown.
- **Decompose the system:** because context engineering (RAG, tool calls) splits an agent into parts, individual pieces like an MCP tool can be tested on their own.
- **Intent engineering breaks the single expected answer:** if the system personalizes, your experience and mine may legitimately differ.
- **Traces should drive tests:** collect traces, let an eval agent detect usage changes, then revise the suite.
- **Telemetry in the loop:** give the harness visibility into failures and spending so it can attempt repair.
- **Eval as spec:** define the desired end state and optimize toward it, rather than only curating a fixed dataset up front.

## Detailed Notes

### The gap: static tests for changing systems · 00:51
- Koc opens with a 2013 VR story: a headset warned "five minutes," he used it for three hours and paid for it.
- The point is that intended use and actual use drift apart — and that gap is where testing gets interesting.
- He works on eval research at Comet, benchmarking and running evaluations for large organizations including Uber, Netflix, and UK banks.
- Against that background, the joke that "evals are dead" carries some truth: the question is what evaluation must become.

### What AI evaluation is missing · 01:54
- Normal software practice builds tests from an expectation: examples become unit tests, regressions get recorded, CI/CD keeps them from shipping again.
- Beyond known cases, **chaos engineering** (deliberately stressing and breaking a system) and observability reveal the limits.
- A typical AI eval stops earlier: handcraft questions, tune until they pass, run them offline, ship.
- That proves behavior on chosen cases but skips the exploratory layer — how do you find where the agent breaks, not just check what you already know?
- Adding more datasets does not fix this: a benchmark about, say, adding numbers can measure a capability without explaining its relevance to your app.

### When the harness changes, what does a passing test mean? · 04:21
- The application under test is itself moving: adaptive harnesses shift behavior and create new skills over time.
- A benchmark tied to an earlier version of the app becomes less representative as the app changes.
- He cites an **adaptive-testing** paper that asks whether benchmarks can change with applications, and describes it as smarter, more selective testing.
- But that is narrower than maintaining an evaluation system that follows real usage — choosing better questions is not the same as changing what the suite must cover.

### From prompt wordsmithing to modular component tests · 05:36
- The road here starts with **prompt engineering**: change the words, watch the output, hope it improves.
- He compares that to developing a liver medicine and finding it also relieves pain — a useful effect with no clear explanation.
- His quip that prompt engineering "died in 2023" targets unguided wordsmithing, not the people still doing it.
- **Context engineering** made the moving parts explicit: incoming data, search, retrieval-augmented generation (**RAG** — fetching context to ground answers), and tool calls.
- Because **MCP** (a standard way for agents to call tools) turns capabilities into separate components, a sales tool can be tested for its own behavior apart from the whole agent.
- Passing component tests still does not settle the behavior of the complete agent.

### Intent engineering makes one answer obsolete · 07:25
- More available tokens make more generation possible; his phrase "tokens become fast food" captures the resulting velocity.
- Model capability expands what that generated software can do. He mentions models solving ARC-AGI-2 puzzles and trying the interactive ARC-AGI-3 environments (no model names, harnesses, or scores are given).
- This leads to **intent engineering**: systems that adapt toward what a user wants, as he sees in OpenClaw and harnesses around Claude and Codex.
- Personalization creates a new testing problem: if the system adapts differently to each person, one expected interaction no longer describes everyone.
- That raises, rather than lowers, the need for evaluation and observability — vague claims like "agents are insecure" become useful only when made specific.

### Malleable evals: let traces reshape the tests · 10:00
- An intent-based outcome needs a different target. A **rubric** (written criteria) makes it explicit, much like grading artwork without demanding identical pictures.
- The suite can also change as usage changes: imagine 80% of traced activity is familiar, then the customer base shifts and questions change.
- Those traces are evidence that the suite may no longer match the workload.
- His proposed trace-driven loop:
  1. Collect traces of the app's behavior.
  2. Feed them to an evaluation agent that looks for usage changes.
  3. Notify owners when something has changed.
  4. Revise the suite to cover the new behavior.
- The agent helps curate tests instead of leaving all of it to people.

### Always-on evaluation and telemetry in the loop · 11:19
- Once agents read traces and run evaluations, evaluation can become an **always-on evaluation and optimization service** rather than an occasional offline run.
- He describes **telemetry in the loop**: expose operational information to the harness running the software, its MCP tools, or the agentic system.
- Two signals matter: failures (what broke, so the harness can attempt a repair) and cost (resource spend, with conditions that guide acceptable behavior).
- This changes what the harness can respond to — evidence from the current run rather than only defects anticipated during development.
- He frames it as enabling self-correction "to some degree," not guaranteed recovery.

### Make the desired end state the target · 12:12
- He tentatively calls the opposing failure mode **eval calcification**: evaluation grows rigid while the system keeps changing.
- He points to Karpathy's **autoresearch** as a pattern: set a target, evaluate a candidate change, use the signal to iterate.
- A deliberately mundane example — barbecue mix — makes intent concrete: "tastiest" and "cheapest" are different objectives, so they need different reward signals even over the same candidates.
- An adaptive loop goes past picking from fixed candidates: it proposes changes, assesses them, and corrects its next attempt.
- The reframing: evaluation becomes a **specification of the desired end state**, not just the dataset you started from.
- Closing thought experiment: ~80% of behavior is stable and defined by intent, while ~20% keeps changing. The changing slice holds the odd question that can hurt the business, and it should be monitored and maintained by agents.
- He treats evaluation as code, software, or a living agent — a growing, self-optimizing system, not a point-in-time dataset. The deeper Comet demonstration was not shown because it was still unfinished.

## Resources

- [OpenClaw](https://github.com/openclaw/openclaw) — the personal-assistant project connecting models, tools, and skills.
- [Karpathy's autoresearch](https://github.com/karpathy/autoresearch) — an optimization loop that tunes toward a goal.
- [ARC-AGI-2 paper](https://arxiv.org/abs/2505.11831) — the reasoning benchmark's original paper.
- [Introducing ARC-AGI-3](https://arcprize.org/blog/arc-agi-3-launch) — interactive environments with unstated rules and goals.
- **Mind the Metrics** (paper) — Koc and collaborators on bringing telemetry, traces, and evaluation into the loop.
- **ATLAS: Adaptive Testing for LLM Evaluation** (paper) — selecting informative benchmark questions with psychometric models.
