# Task Fidelity Scaling Laws

| Field | Details |
| --- | --- |
| **Speaker** | Kobie Crawford |
| **Affiliation** | Snorkel AI |
| **Type** | Talk |
| **Date** | 2026-06-02 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=YYH0DMQr30A) |
| **Tags** | `reasoning-and-models` `evals` `rag-context-and-search` `data-and-model-adaptation` |

## Summary

Kobie Crawford, a developer advocate at Snorkel AI, asks a simple question: does the quality of a training task actually change what a model learns? Snorkel produces datasets for foundation models, and its long-running thesis is that data quality is critical. As models learn to act, that concern becomes task quality.

The team defined four criteria for an **accepted** task — achievable, non-trivial, functionally correct, and running in a reliable environment — and sorted tasks into accepted and rejected buckets. Accepted tasks turned out to be genuinely harder: about twice as many tool calls, lower pass rates and more output tokens. More importantly, their failures were "cleaner", caused by real difficulty rather than broken environments or faulty logic.

Then came the headline experiment: two reinforcement-learning runs with the same model, compute budget and task count, differing only in task quality. The rejected tasks improved the base model by roughly 1%; the accepted tasks by roughly 6%. Crawford calls that a five-times uplift and says it validates Snorkel's emphasis on expert-built data and environments — while being clear that this matched setup does not, on its own, prove the multiplier.

Audience questions pushed on the harder edges: impossible tasks add noise to benchmarks, underspecified tasks can look hard when the real problem is a hidden test requirement, and subjective domains may have several correct answers. Snorkel handles that with human annotators plus LLM (large language model) judges scoring against shared rubrics, and by measuring agreement.

## Key Takeaways

- **Data quality in agent training shows up as task quality.**
- **An accepted task is achievable, non-trivial, functionally correct and runs in a reliable environment.**
- **Accepted tasks were harder:** about twice the tool calls, lower pass rates and more reasoning tokens.
- **A low pass rate proves nothing on its own.** You must separate real capability failures from broken tasks.
- **Matched RL runs:** low-quality tasks improved the base model by about 1%, high-quality tasks by about 6%.
- **Underspecification creates fake difficulty.** The tests demand something the task never asked for.
- **Impossible tasks become benchmark noise** and can mask genuine model progress.
- **For subjective work, use rubrics** and compare agreement between humans and between humans and judges.

## Detailed Notes

### Does task quality actually matter? · 00:15
- Crawford introduces Snorkel AI as a data lab that produces datasets for foundation models to improve on; its roots run from Stanford research through an open-source library to customer datasets.
- Since the company started in 2019 the through line has been data quality. In agent training, that concern becomes **task quality**.
- Rather than assume better tasks mean better models, the team set out to test it empirically. The setting is Terminal-Bench-style work: a task definition inside a containerized environment where an agent takes actions to reach an objective.
- Architecture and harness choices also affect results; the question is whether task quality still matters underneath them, and how to isolate its contribution.

### Four criteria for an accepted task · 03:30
- Frameworks such as Harbor and OpenEnv provide containerized environments for evaluation, benchmarking and RL (reinforcement learning). Containers give reproducibility, isolation and parallel rollouts.
- But a reproducible container is not enough. Snorkel's research harness checks four things: the task is **achievable**, **non-trivial**, **functionally correct**, and runs in a **reliable environment**.
- Tasks that pass all four checks go into the **accepted** bucket; the rest go into **rejected**. Those buckets are the operational definition of higher and lower quality for the comparisons that follow.

### Accepted tasks are genuinely harder · 05:02
- Using Sonnet 4.5 and Codex variants (he mentions GPT-5.2, GPT-5.1 and a vague "4.0"), accepted tasks averaged about twice as many tool calls as rejected ones.
- Accepted tasks also had lower pass rates and needed more output tokens — read as more steps, more tool engagement and more reasoning effort.
- He is careful: these are behavioral indicators of difficulty in this comparison, not a definition of difficulty based on tool count.
- Crucially, a lower pass rate alone does not establish a better training task. Failures could reflect a real capability limit, or a task that failed for reasons that teach nothing.

### Separate capability failures from broken tasks · 06:28
- The team categorized failures and compared their composition across the accepted and rejected buckets, highlighting logic errors and incomplete tasks (where which bucket leads reverses).
- The distinction that matters: a model that cannot reach a needed logical conclusion (a capability worth training) versus an environment that blocks any model from finishing (an execution defect).
- His interpretation is that accepted tasks produce **cleaner failures** — the agent fails because the work is genuinely difficult, and those attempts are useful samples. An environment defect produces a failed run without exposing a learnable skill.

### The training experiment: about 1% vs about 6% · 08:44
- With the behavioral and failure analysis in place, the team ran RL training twice on the same model, holding the compute budget and number of tasks constant and varying only which bucket supplied the tasks.
- Lower-quality (rejected) tasks improved the base model by roughly 1%; higher-quality (accepted) tasks by roughly 6%.
- Crawford calls the difference a five-times uplift. The talk does not specify the trained model, the evaluation set, the metric, or whether the figures are relative changes or percentage-point gains, so the rounded numbers alone do not establish the multiplier.
- The result supports Snorkel's emphasis on human expertise when generating datasets and RL environments, though this comparison does not isolate the experts' contribution from other aspects of task construction.
- He points to Snorkel's research page and its curated benchmarks, including an Agentic Coding Benchmark for Terminal-Bench-style tasks.

### Q&A: noisy benchmarks and hidden requirements · 11:56
- Asked whether a model could learn well from a mix of accepted and rejected tasks, Crawford has no direct result. He describes related internal work comparing Terminal-Bench 1 and 2 and SWE-bench variants.
- One finding: tasks that can never be completed add noise to benchmarks and can mask whether models are improving, since some stay unsolved because they cannot be solved as written.
- On how much difficulty comes from the input, he identifies **underspecification** as a common reason for rejection: the task omits a desired, testable outcome, but the backend tests still require it.
- Example from the bundle: a request to implement a function that returns each name once; the hidden requirement is first-seen order, while an alphabetical implementation fails the assertion. The repair is to state the requirement ("preserving its first occurrence"), which makes the test interpretable without changing the code.
- A related defect is an implicit testing dependency that appears neither in the spec nor in the model's context. Both cases separate missing information from genuine reasoning difficulty.
- An audience member notes that real problem solving is iterative, not one-shot; Crawford agrees, and says benchmarks deliberately pick backend-verifiable outcomes while hoping the learned skills transfer to unverifiable, multi-step work.

### Multiple valid answers and annotator agreement · 16:18
- For longer horizons and tasks without straightforward verification, Snorkel has projects across those areas but no single method to describe. Coding and math verify easily; more subjective domains need different machinery.
- Through its Open Benchmarks Grants program, Snorkel partners with people building benchmarks and evaluations in less verifiable areas, including an unnamed organization working on emotional, human-centric tasks.
- There, correctness need not collapse to one acceptable answer: several outcomes can be valid while scoring differently along a spectrum.
- For disagreement, especially across multi-step tasks, Snorkel combines human annotators with **LLM judges** and a shared **rubric** listing detailed criteria, covering both high-level qualitative judgments and more quantitative comparisons.
- Experts supply ground-truth information that informs what the judges should look for, and quality assessment compares agreement between humans and between humans and judges. He aims for high agreement without giving numerical thresholds; the guiding principle is that the evaluation process itself must be checked, even where explicit pass/fail tests exist.

## Resources

- [Introducing the Snorkel Agentic Coding Benchmark](https://snorkel.ai/blog/introducing-the-snorkel-agentic-coding-benchmark/)
- [Terminal-Bench 2.0: Raising the bar for AI agent evaluation](https://snorkel.ai/blog/terminal-bench-2-0-raising-the-bar-for-ai-agent-evaluation/)
- [Harbor](https://github.com/harbor-framework/harbor)
- [OpenEnv](https://github.com/huggingface/OpenEnv)
- [Terminal-Bench (paper)](https://arxiv.org/abs/2601.11868)
- [Snorkel AI Research](https://snorkel.ai/research/)
- [Open Benchmarks Grants](https://benchmarks.snorkel.ai/)