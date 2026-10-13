# Why More Context Makes Your Agent Dumber—and What to Do About It

| Field | Details |
| --- | --- |
| **Speaker** | Nupur Sharma |
| **Affiliation** | Qodo |
| **Type** | Talk |
| **Date** | 2026-06-08 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=EcqMYoIV57A) |
| **Tags** | `architecture` `rag-context-and-search` `agent-engineering` `infrastructure-and-deployment` `reasoning-and-models` |

## Summary

Nupur Sharma, who works on agentic code review at Qodo, argues that a bigger context window does not guarantee better answers. Drawing on her DevSecOps background, she explains that models tend to weight the beginning and end of a long input while paying less attention to the middle — so material you feed in can be overlooked even though it technically fits.

She then lays out practical ways to control context: a "context engine" that ranks and selects evidence, hierarchical summaries, knowledge graphs, lightweight iterative retrieval, and a critic node that checks output against the original goal. Each option trades up-front setup effort against ongoing cost and latency.

The second half covers two failure patterns. The "orchestration paradox" is a capable model that spends its effort choosing a method instead of doing the work; her team answers with an 80/20 hybrid that explores broadly but finalizes under constraints. Then she walks through Qodo's pull-request review architecture, where specialist agents produce candidate findings and a judge agent decides which are relevant — and where calibration decides what "relevant" even means.

## Key Takeaways

- **Capacity is not usable context.** A large window that accepts input does not use every part equally well; the middle is often neglected.
- **Selecting evidence is still engineering.** Rather than dumping everything in, build a process that chooses what matters for the task.
- **Pick a context strategy you can maintain.** Context engines, hierarchical summaries, and knowledge graphs carry different setup and upkeep costs.
- **Iterative retrieval is a low-setup start.** Give the agent a topic index and let it pull deeper code on demand.
- **Watch the orchestration paradox.** A strong model can burn tokens researching how to solve the problem instead of solving it.
- **Split exploration from finalization.** Let the research phase roam, then constrain validation and summarization under hard gates.
- **Specialists need a judge.** Independent agents can return plausible but incompatible results; a judge reconciles them.
- **Separate rules from preferences.** Feedback can tune bug findings, but explicit rules should stay highlighted regardless of what developers accept.

## Detailed Notes

### Failures when inputs multiply · 00:15
- Sharma comes from **DevSecOps**, where pipelines are deterministic: they run or they crash, and you fix the crash. Agentic systems behave differently.
- Early prompting used **static prompts** with a **4K context window**, forcing developers to choose what mattered before asking the model. Wrong inputs meant wrong answers.
- Bigger windows plus tool-equipped agents added a new failure: without a **stopping rule**, the agent keeps searching for more input instead of finishing.
- **Multi-agent** setups add expertise and coordination cost. A security agent, a review agent, and a coding agent can interpret the same code differently.
- Her framing: more context, tools, and agents raise capability **without automatically producing a coherent result**.

### Capacity isn't the same as usable context · 02:34
- A model may accept a large input without using every part equally well. Sharma describes a **U-shaped pattern**: the start and end stay prominent, the middle gets less reliable attention.
- The issue is neglected evidence, not literal deletion of tokens from the window.
- Asked how she knows, Sharma cites internal code-review benchmarking: task-specific agents get context, including experiments with a whole codebase, and the team checks whether that context changes the result.
- She reports that the initial goal and final input stay salient while intermediate material — Jira data, context available through MCPs — can be overlooked.
- The talk gives **no named models, evaluation protocol, or numbers** for that comparison, so treat it as a working observation.
- Takeaway: **context selection remains the engineer's job** even when capacity is no longer the binding limit.

### Context strategies you can maintain · 04:55
- A **context engine** acts as a gatekeeper (her word: a "bouncer"). For a large, messy codebase it builds search and ranking logic so a task pulls relevant material before the model runs.
- Indexing is moderate effort, but she reports mapping and indexing slowdowns around **600–700 repositories**. No repository sizes or latency figures are given — it is an operational warning.
- Maintaining that infrastructure may not be worth it if your product is not the context engine itself.
- **Hierarchical summarization:** summarize files and folders, then let an agent read summaries to decide where to inspect. Costs are ongoing, because new or changed files need fresh LLM processing.
- **Knowledge graphs:** represent dependency chains, such as one file affecting another. Strong for complex, multi-repo logic, but they demand heavy developer setup and hosting.
- **Iterative retrieval:** give the agent a topic index — a "library card" — so it checks relevance, then retrieves the associated code. Lower up-front effort, still a retrieval cost. Sharma favors this for teams building agents for their own processes.
- **Critic node:** compares the output against the original goal and can request a retry. Little initial developer input, but repeated runs add latency.
- The choice comes down to what the agent needs to navigate: descriptions of content, or explicit relationships between pieces.

### The orchestration paradox · 08:31
- Selection does not fix the **orchestration paradox**: a more capable model spends its effort deciding how to solve a problem instead of solving it.
- It picks a tool, doubts the choice, researches another method, and loops. Tokens go into method selection while the task stays unfinished.
- She uses the then-current **Opus** (no version given) as an example of this repeated self-challenging.
- The fix is an **80/20 hybrid**: let the research phase explore broadly, then constrain final validation and summarization. The split is a design heuristic, not a measured optimum.
- The final stage is prescribed: given a result, produce the required output instead of reopening the whole method search.
- Boundaries for exploration: an **iteration limit** (after four or five cycles, proceed with the latest result) or a **time limit** (after five minutes, proceed with the latest choice and revisit if the output is poor). These force a move to execution; they do not make the latest answer correct.

| Stage | Work | Model role |
| --- | --- | --- |
| Exploration | Discovery, planning, tool choice | High-reasoning model |
| Finalization | Validation, goal checks, summarization | More constrained model task |

- Deciding what to investigate next belongs to exploration; assembling the requested output belongs to the constrained side. That second stage may not need the strongest reasoning model because its instructions are narrower.

### Specialists need a judge · 12:22
- A big window tempts teams to give one agent every job. Testing and reviewing overlap in material but are distinct tasks.
- Failure example: an agent given four tasks does well on two and loses the other two — "lost in the middle" again.
- A **mixture of agents** replaces one overloaded agent with small issue experts, each owning a specific job.
- Specialization creates a reconciliation problem. Vacation example: one agent finds a hotel in Greece, another a flight from Amsterdam to Portugal — each plausible alone, useless together.
- A **judge agent** receives the specialist outputs and checks whether they make sense together, turning local suggestions into one coherent result.
- Specialists supply depth; the judge supplies the shared decision about what belongs in the final answer.

### Qodo's pull-request review pipeline · 14:41
- The pipeline separates gathering evidence from producing findings:
- **Collect context** — a context collector gathers material from the PR, the context engine, and available tools.
- **Partition the evidence** — instead of generating a review immediately, it distributes selected material to specialist agents.
- **Produce candidate findings** — specialists investigate concerns such as security flaws, code differences, and Jira issues.
- **Judge relevance** — the judge reviews the findings and can revisit the context engine and the PR before deciding what belongs in the review.
- In her example, **ten candidate findings** are inputs to refinement, not ten comments that must reach the developer.
- On communication between agents: Qodo uses **LangChain** as underlying infrastructure. The handoff is plain — collect one agent's results and use them to build the next agent's prompt. When several outputs need combining, another agent can collect and refine them first.
- The talk does not name a specific LangChain API or version. The prompt-building step forms the next task; it does not decide which findings are valid.

### Calibration and the system-wide picture · 17:06
- **Calibration** asks how an agent learns what counts as a good finding. The same Java framework can serve healthcare, retail, and finance teams with different priorities.
- Qodo indexes **PR history** and retrieves earlier changes similar to the current submission.
- That history enters **twice**: once when specialists receive their context, and again when the judge weighs their recommendations. With 15 recommendations, the judge can consult past reviewer and developer comments to decide what to surface.
- This applies to every agent, but not every agent receives the same full history — the context engine selects the relevant subset. Shared organizational knowledge is not the same as identical prompts.
- An audience member pushes on the missing system-wide view: local checks like linting or tests are easy, but architecture and security tradeoffs need broader judgment.
- Sharma compares this to human review: a senior engineer knows the codebase, a security expert watches for hardcoded keys and SQL injection, an auditor checks ISO or SOC 2 logging. They contribute specialized knowledge rather than duplicating one another.
- The agent equivalent needs explicit inputs: a **web portal** where architects and compliance staff supply guidelines, with the context collector holding the broader picture and routing relevant parts to specialists.
- Consequence: an agent cannot validate a requirement that never reaches its context. Teams using the system out of the box should not expect findings specific to their practices unless relevant PR history exists.
- Sharma notes PR history is **one source among several**, not a complete specification of how a team should work.

### Preferences versus rules · 23:46
- Weighting signals include the importance set in the compliance portal, whether an item is an error or a recommendation, and whether developers accept the suggestions. Accepted suggestions gain weight; rejected ones lose it. No numerical formula is given.
- Two evidence sources feed this: acceptance of current recommendations, and whether similar past findings were actually implemented.
- That evidence can encode bad habits. Sharma recalls arguing with a developer used to hardcoding API keys — an established practice is not a safe one.
- The audience makes the objection explicit: historical behavior does not prove correctness.
- Her closing distinction is between **explicit rules** and **feedback-sensitive bug findings**. A supplied rule stays highlighted no matter what developers prefer. A bug finding can lose weight after repeated disagreement and non-implementation; her example of ten rejections illustrates the behavior, not a product threshold.
- That boundary settles what adaptation may change: feedback tunes which bug findings get attention, while rules remain obligations. A review system needs both — evidence of what a team accepts, and requirements that acceptance cannot erase.

## Resources

- Lost in the Middle: How Language Models Use Long Contexts — research on how evidence position affects recall (related).
- Qodo 2.0's specialist-and-judge review architecture — the release explanation behind the pipeline (related).
- [LangChain subagents](https://docs.langchain.com/oss/python/langchain/multi-agent/subagents) — guidance for coordinating subagents and controlling handoffs.
- Benchmarking Qodo's multi-agent review system — a May 2026 account of controlled PR defects (update).
- Qodo's Context Engine — an August 2026 explanation of bounded context packages and separation of concerns (update).