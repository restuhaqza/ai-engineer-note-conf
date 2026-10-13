# The maturity phases of running evals

| Field | Details |
| --- | --- |
| **Speaker** | Phil Hetzel |
| **Affiliation** | Braintrust |
| **Type** | Expo session |
| **Date** | 2026-05-27 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=FB-MLPhL9Ms) |
| **Tags** | `evals` `infrastructure-and-deployment` |

## Summary

Phil Hetzel leads solutions engineering at Braintrust, an agent-quality company. His talk maps how teams' evaluation practices mature as their agents get more complex — from loose human checks to production-scale, trace-level automated evaluation.

The starting point is human judgment. Run a handful of inputs through an agent, have someone (ideally a subject-matter expert) mark each answer good or bad, and — most importantly — write down why. That written rationale is the reusable part; it later becomes the basis for automated scorers.

As patterns emerge, teams turn those justifications into scoring functions: **LLM-as-a-judge** evaluators (a model grading another model's output) for subjective criteria, and plain code for objective checks like tool-call or token budgets. But a judge is not trustworthy just because it is a model — it needs its own evaluation.

Once agents touch external systems, evaluating only the final answer is not enough: teams must assess the whole trace and deal with the hard problem of replaying external state without corrupting production data.

## Key Takeaways

- **Evals and observability are one confidence loop.** Evals build confidence before launch; observability keeps it afterward.
- **Start from failure modes, not exhaustive coverage.** Enumerating every way an agent can fail is unbounded and leaves no time to ship.
- **Always record why** a reviewer rated an answer good or bad — that justification is what you scale later.
- **LLM-as-a-judge scales subjective assessment, but validate the judge.** A model in a judge's robe is not automatically trustworthy.
- **Use deterministic code for objective checks** such as tool-call or token budgets, and judges for subjective ones.
- **Results don't need to be perfect.** Directional signals that a change is helping are still useful.
- **Include production traces in your dataset.** Think of evals as rerunning production.
- **As agents call external systems, inspect the entire trace** and replay carefully so you do not overwrite production data.

## Detailed Notes

### Why prototypes stall before production · 01:13
- Hetzel leads solutions engineering at Braintrust; before that he spent 12 years consulting (four at KPMG, eight at Slalom, where he led the global Databricks unit).
- He noticed customers were prolific at building generative-AI proofs of concept but poor at getting them to production; he started using Braintrust to close that gap, then joined the company.
- Braintrust treats two things as the same systems problem: **evaluation** (gaining confidence before launch) and **observability** (staying confident after launch).
- Quality matters for risk and brand. An unkind or unhelpful agent damages reputation, a wasteful one costs money, and one that goes off the rails can create legal or compliance exposure.
- Evals also play offense: they tell you, for each change, whether the application improved and by how much.

### Start from failure modes, not exhaustive tests · 04:26
- Evals are not unit tests. Unit tests aim for exhaustive coverage, but trying to enumerate every possible agent failure is infinite — you would write tests forever and never ship.
- Instead, start high level: you or a subject-matter expert name the agent's failure modes, and you build evals around those cases.
- Results do not have to be perfect. **LLM-as-a-judge** may be nondeterministic yet still gives directional evidence that things are improving.
- Three eval primitives:
  - **Task** — the agent or prompt under test.
  - **Dataset** — examples that invoke the task.
  - **Scoring functions** — how you judge the task's quality.
- Keeping those separate lets you change the agent while keeping a fair basis for comparison.
- He lays out four overlapping maturity areas (a continuum, not strict gates): getting started, measuring to manage, accounting for complexity, and advanced techniques. More capable agents have more ways to fail, so evals expand with them.

### Getting started: write down why an answer is good · 06:57
- A fine first step is to run roughly ten inputs through the agent and inspect the outputs (an illustrative batch, not a proven sample size).
- Have a person score each response — ideally a subject-matter expert who can recognize a good answer.
- Capture two things per response: a thumbs up/down, and a **written justification**.
- The justification is the reusable asset. It records what the reviewer noticed and why, moving the quality standard out of one person's head.
- One audience member described running an agent, eyeballing results, tweaking a prompt, and repeating. Hetzel's advice: preserve what that loop teaches you.
- The review interface matters. A generic trace dump may not fit how an expert thinks, so present traces in a form reviewers recognize (his example is Braintrust's customizable annotation views).

### Measuring to manage: turn explanations into scorers · 09:43
- Once justifications accumulate, analyze the negative ones (with tools like Cursor, Claude Code, or Codex) to name recurring failure modes.
- Then automate the assessments that previously depended on a few experts.
- Use **LLM-as-a-judge** for criteria that need interpretation — but giving a model the role of judge does not make its verdicts trustworthy. The judge's own outputs need evaluation.
- For objective failure modes, plain code is often better. His example is a small TypeScript scorer that checks an agent's tool-call budget and token budget.
- Each check returns 1 when the budget is respected and 0 when it is exceeded. Keeping them separate preserves which failure occurred; neither score tries to decide whether the answer was helpful.
- The limits are inputs, because the acceptable budget depends on the application.

### Use production traces to guide improvements · 11:27
- By this stage the dataset should include production traces, or at least user-acceptance-testing traces.
- The mental shift: do not think of evals as running tests — think of them as **rerunning production**, so offline experiments reflect real usage.
- The improvement flywheel:
  1. **Observe** — capture agent traces in production.
  2. **Analyze** — use humans or tooling to understand the failures.
  3. **Evaluate** — bring real examples into an offline experiment and rerun them.
  4. **Improve** — use the results to guide the next change.
- The same production behavior that exposes a weakness becomes evidence for deciding what to fix, which connects observability back to evaluation.
- LLM judge scoring extends this loop beyond what human-only review can handle.

### Evaluate the whole trace, not just the answer · 12:54
- Level two: agents start doing work with external systems, often through tool calls.
- He splits tools into two kinds:
  - **Context gathering** — retrieves data and injects it into the LLM.
  - **CRUD** — creates, reads, updates, or deletes records in an external system.
- Both add usefulness and failure surface. The distinction points to what a call adds to the model's context versus what it changes in the world.
- The unit of inspection grows from the final answer to the **entire trace**: capture the sequence of steps and target evals at individual tool or MCP calls where needed. (MCP is a standard way for agents to call tools.)
- A single final answer cannot reveal every problem in the path that produced it.

### Replay external systems safely · 14:14
- Offline evaluation of external calls faces two hard problems: representing the state those systems were in when the input was captured, and avoiding overwriting production data during create, update, or delete.
- He calls this not completely solved; today's approaches approximate production rather than perfectly reconstruct it.
- **Mock APIs** give you a place to exercise interactions against a stand-in for external state.
- He suggests treating the trace as a container for far more context than conventional application tracing carries — store the relevant system state alongside the run and inject it into the task during evaluation.
- Another option: query the external system as it existed at the original time. A vector database, for example, might support timestamp or version queries to recover the retrieval state — but only if it is configured and retains history that way.

### Automate discovery, then evaluate the judges · 16:47
- Two emerging patterns: **topic modeling** across production traces to surface recurring failure modes automatically, and using coding agents (like Claude Code) with an eval provider's CLI to automate more of the workflow.
- These are directions for further work in the talk, not demonstrated implementations.
- An audience question asked whether platforms should favor deterministic graders over LLM judges.
- His answer starts with the nature of the criterion: some aspects of agent behavior are subjective, so an LLM judge can fit.
- The requirement is to evaluate the judge extensively against what a human would decide in the same situation.
- A concrete method: constrain the judge to discrete decisions and build a ground-truth dataset of human judgments, so its choices are easy to compare. Deterministic checks still handle objective conditions.

## Resources

- [Build custom annotation views](https://www.braintrust.dev/docs/annotate/custom-views) — tailoring the human review interface to how experts think.
- [Write code and LLM scorers](https://www.braintrust.dev/docs) — choosing scoring methods and evaluating spans or whole traces.
- [Build evaluation datasets from production](https://www.braintrust.dev/docs) — organizing production logs, human feedback, and curated examples.
- [Run evaluations with the Braintrust CLI](https://www.braintrust.dev/docs) — current instructions for running and querying evaluations.
- [Discover production patterns with Topics](https://www.braintrust.dev/docs) — classifying production traces by task, sentiment, and intent.
