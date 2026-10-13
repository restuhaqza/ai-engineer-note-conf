# Judge the Judge: Building LLM Evaluators That Actually Work with GEPA

| Field | Details |
| --- | --- |
| **Speaker** | Mahmoud Mabrouk |
| **Affiliation** | Agenta |
| **Type** | Talk |
| **Date** | 2026-04-10 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=X4dEHRzBLmc) |
| **Tags** | `evals` `reasoning-and-models` `rag-context-and-search` `observability-and-reliability` |

## Summary

Mahmoud Mabrouk, co-founder and CEO of Agenta, shows how to build an LLM-as-a-Judge that actually agrees with human annotations. The talk opens with a familiar failure: a hallucination judge reports that everything is fine while customers complain. The judge's prompt offers no criteria or facts, so it cannot define "hallucination" any better than the app already does.

The fix is calibration — making the judge's decisions match reference judgments — and optimizing its prompt with GEPA, a genetic-style search over prompts. The exercise uses Sierra's τ-bench airline customer-support benchmark and 599 annotated conversation traces. A successful tool call is not enough: the agent can cancel a booking without checking whether it was eligible.

Mabrouk derives four narrow metrics from actual errors (policy adherence, response style, information delivery, tool calls) and builds one judge per axis, using binary verdicts with reasons rather than 1–5 scores. The written reason is the training signal: it tells the optimizer which rule is missing.

GEPA starts from a seed prompt, mutates and merges candidates, and uses Pareto-aware selection so prompts with complementary strengths survive. The recorded run lifts validation accuracy from 61% to 74% for a single rubric. His biggest lesson: inspect one small iteration before paying for a large search, and spend on a strong refiner plus a cheaper judge.

## Key Takeaways

- **A generic judge can't define "hallucination"** any better than your app already does.
- **Calibrate judges against human annotations.** Agreement is what makes their signal trustworthy.
- **Derive metrics from real errors** with subject-matter experts, not from generic scores.
- **Use one narrow judge per metric**, and prefer binary verdicts with reasoning over 1–5 scales.
- **The written reason carries the policy knowledge** the optimizer must learn; a bare label is not enough.
- **GEPA is a genetic-style prompt search:** seed → mutate/merge → evaluate → Pareto-filter.
- **Split data by task** so related runs don't leak across the training/validation boundary.
- **Reported result:** validation accuracy rose from 61% to 74%, but merging all strengths into one prompt stayed hard.

## Detailed Notes

### The monitor says everything is fine · 00:00
- Scenario: an agent is live, a hallucination judge is added, and the dashboard looks healthy.
- Customers complain anyway, and the traces confirm the failures.
- The judge's prompt basically says "rate whether this is a hallucination, make no mistakes."
- It has neither the facts nor the criteria to decide, so it cannot help.
- A useful evaluator needs a standard of correctness that matches the application.
- "Calibration" here means agreement with reference judgments, not calibrated probabilities.

### Why evaluation sets the speed of improvement · 01:18
- Offline loop: change the prompt or harness → run evals → keep wins → repeat. Evaluation is the bottleneck.
- Human review is high quality but slow; a misaligned LLM judge is fast but produces useless signal.
- Online evals need judges aligned with business goals to spot regressions and shifts in user behavior.
- The "data flywheel": inspect traces → turn edge cases into new evals → optimize the harness → observe again.
- Agenta is an open-source LLMOps platform spanning observability, prompt management, and evaluation.

### A cancellation can succeed and still break policy · 05:47
- Data: airline conversations from Sierra's τ-bench; the agent manages reservations, flights, and user info.
- Tools run under detailed rules, so a successful tool call is not a successful interaction.
- 599 traces, roughly 62% compliant / 38% non-compliant, across multiple models and trials.
- Assertions were converted into trace-level verdicts with explanations.
- Example: non-compliant because the agent approved a cancellation without checking eligibility.
- Mabrouk calls this imperfect demonstration data: small, complex, and unevenly distributed.

### Derive metrics from observed errors · 08:10
- Four steps: design the metrics, annotate the data, optimize the judge, validate the result.
- Metrics must come from the use case; generic scores like "hallucination" don't fit a support agent.
- Subject-matter experts review real conversations, following Hamel Husain's error-analysis workflow.
- Four categories emerged: policy adherence, response style, information delivery, and tool calls.
- Information delivery is separate from doing the action — an agent may act but fail to tell the customer.
- Build one judge per category; an all-purpose "success" judge is too complex to learn.
- Prefer binary compliant/non-compliant with reasoning over 1–5 scales; even binary is hard to calibrate.

### Annotations must explain the rule · 11:46
- Annotators get a named criterion and give a verdict plus supporting reasoning.
- The reason is the training signal: it tells the optimizer which rule is missing.
- Binary labels alone force the algorithm to infer both hidden rules and how to apply them.
- Some narrow tool failures are visible in the trace; complex policy failures usually are not.
- Metric design and annotation are the hardest parts, even though optimization gets more screen time.
- In this demo the explanations are AI-generated from assertions, not human-collected — keep the recommended human workflow separate.

### How GEPA works: mutate, merge, filter · 14:32
- GEPA is genetic-style: start from a seed, generate candidates, evaluate them, filter the pool, repeat.
- **Mutation:** run a judge on a trajectory, examine its input/output/feedback, and let an LLM reflect a better prompt.
- **Merge:** combine useful guidelines from two candidate prompts.
- The first iteration has only the seed; merging becomes possible as the pool grows.
- Candidates are tested on mini-batches, and improvers join the pool.
- **Pareto-aware selection** keeps per-trajectory bests, so a prompt good at a hard case isn't dropped for a lower average.
- The goal: a candidate set where every task is solved by someone, then merged into one rubric.

### Split data, seed smartly, teach reflection · 22:26
- Tooling: dotenv, LiteLLM, and GEPA; optimization ran ahead of the recording (roughly one to two hours).
- Split by benchmark task: 480 training traces and 112 validation traces (592 total).
- Repeated runs of the same task and model add redundancy, but none crosses the training/validation line.
- Seed judge: "assume compliant, switch only with a specific reason." Earlier generic seeds were nearly random and hard to fix.
- The seed scored 61% validation accuracy with very low non-compliant recall — intentionally biased toward compliant.
- The default reflection template underperformed, so a custom airline-aware template instructs the refiner to extract reusable policy rules.
- Allowed edits: add rules, restructure rules, and reword for clarity.
- The seed without the full policy pasted in performed better than one that copied it, likely avoiding a local optimum.

### Results, limits, and where to spend the budget · 32:46
- Reported validation accuracy: 61% → 74%; non-compliant precision and recall rose from near zero.
- A "compliance" figure fell from 98% to 64%, described inconsistently — treat it as loosely defined.
- Repo artifacts report different numbers (65.2% → 69.6%) not tied to the recorded run.
- Training accuracy improved about nine points, and the Pareto set covered 100% of training tasks (a property of candidates, not the chosen judge).
- The experiment achieved complete coverage but not successful consolidation: merging strengths into one prompt stayed hard.
- Model choices mattered: GPT-4o failed on this logic; the best combo was Gemini for reflection and Grok for judging.
- Suggested workflow: run one small iteration, read the reflection, check which candidates improve, fix the refinement prompt, then scale.
- Cost: roughly $200–$300 in tokens for the small experiments; refinement is upfront, judging is recurring.

## Resources

- [Judge the Judge workshop notebook, data, and experiments](https://github.com/Agenta-AI/judge-the-judge-talk-2026)
- [GEPA optimization library](https://github.com/gepa-ai/gepa)
- [Original τ-bench repository (Sierra)](https://github.com/sierra-research/tau-bench)
- [A Field Guide to Rapidly Improving AI Products (Hamel Husain)](https://hamel.dev/blog/posts/field-guide/)
- [Grok judge and Gemini reflection experiment](https://github.com/Agenta-AI/judge-the-judge-talk-2026/blob/main/experiments/exp_007_grok_gemini.py)
