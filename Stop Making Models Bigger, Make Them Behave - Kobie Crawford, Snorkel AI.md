# Stop Making Models Bigger, Make Them Behave

| Field | Details |
| --- | --- |
| **Speaker** | Kobie Crawford |
| **Affiliation** | Snorkel AI |
| **Type** | Talk |
| **Date** | 2026-06-10 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=TNwJ1LMiENk) |
| **Tags** | `data-and-model-adaptation` `evals` `reasoning-and-models` `finance` |

## Summary

Snorkel AI developer advocate Kobie Crawford asks a deliberately narrow question:
can a 4-billion-parameter model beat a 235-billion-parameter one at financial
analysis? The research was done with UC Berkeley's rLLM and Agentica teams. His
title is a hook, not a claim that big models are bad — the point is that the right
data aimed at the right problem can win without adding parameters.

The story starts with a failure. Given a question about YouTube ads revenue growth
from 2023 to 2024, a Qwen3-235B model queries a table that does not exist, guesses
again, then answers anyway with a hallucinated number. Reasoning was never the
problem; the model never inspected what it could actually query. Crawford's team
calls this the "Terence Tao effect" — bringing world-class math to a job that only
needs disciplined SQL and simple arithmetic.

His fix is reinforcement learning (RL — training by rewarding good behavior)
on expert-curated data. Using GRPO (a method for scoring several attempts per
prompt), a 4B starting model, the rLLM framework, and a self-contained FinQA
environment, the training run cost under $500. The trained model then discovers
tables, inspects the schema, hits an error, corrects it, and answers from real
data.

It beats the 235B model and nearly doubles its own single-attempt success rate.
On the harder multi-table FinQA Reasoning set it rises from 13.9% to 26.6%.
Surprisingly, training only on easy single-table questions gave the biggest gain.
Crawford's takeaway: find the specific behavior that is failing, and train that.

## Key Takeaways

- **A small model can win when the bottleneck is behavior.** A 4B model beat a 235B model at financial tool use after targeted RL.
- **The failure was tool use, not reasoning.** The large model never checked which tables existed and hallucinated when queries returned nothing.
- **The "Terence Tao effect":** a financial analyst needs disciplined SQL and simple arithmetic, not maximum mathematical brilliance.
- **Expert-curated, verifiable data is the foundation.** Domain specialists write tasks the available data can actually answer.
- **RL does not have to be expensive.** The GRPO training run cost under $500 — that is the run cost, not the whole research budget.
- **Success requires recovery, not just correctness.** The trained agent asked for a wrong column, read the error, and fixed it itself.
- **Easy training transfers to hard questions.** Single-table-only training gave the best uplift, and it carried over to multi-table problems.
- **Use rubrics to diagnose.** Split a response into yes/no questions to find *where* it failed; the optimizer still trains on one scalar reward.

## Detailed Notes

### A 4B model versus a 235B giant · 00:57
- Crawford is a developer advocate at Snorkel, which brings domain experts into the process of assuring data quality.
- The research question is concrete: can a 4B model outperform a 235B model on tool use for financial analysis?
- The work was a partnership with the rLLM and Agentica teams at UC Berkeley.
- The goal is to improve one behavior, not to argue that large models are useless.

### From prototype to something you can deploy · 03:35
- When an enterprise app underperforms, dropping in a bigger model is the obvious move — and it adds serving load and inference cost.
- Production also demands reliability, safety, security, and predictable behavior; size alone does not deliver those.
- After a successful proof of concept, teams ask harder questions: can it run on premises, and can we avoid exporting sensitive data?
- In finance and healthcare those constraints can make a smaller model attractive on their own, independent of benchmark scores.
- Crawford's intervention is RL: train the tool-use behavior rather than treat the problem as missing knowledge inside the weights.

### A revenue question reasoning alone cannot solve · 06:13
- A financial analyst mostly retrieves records with SQL and does simple arithmetic; broad mathematical brilliance is not the job.
- The demo asks Qwen3-235B about year-over-year YouTube ads revenue growth from 2023 to 2024.
- It queries a table that does not exist, never inspecting the environment to see what tables are available.
- After the failure it guesses again, gets nothing usable, and answers anyway — Crawford calls the number hallucinated.
- The breakdown happens before any growth calculation: no revenue values were ever retrieved.

### Expert data and a cheap training loop · 09:10
- Training starts with a dataset built with financial-analysis practitioners, PhD-level specialists, and industry veterans.
- Verification checks that each task fits the application, can be answered from the available data, and has a verifiable answer.
- A plausible financial question is not enough if the environment cannot supply the evidence to solve it.
- The stack combines GRPO, a 4B starting model, the rLLM framework, and the team's FinQA environment.
- The run cost under $500; Crawford frames targeted RL as a tractable option for teams already considering self-hosted small models.

### Package the tools, split the benchmarks · 12:03
- FinQA packages a specific tool set plus the data needed to answer its financial questions.
- The deployed environment is self-contained; Crawford compares it to Harbor and OpenEnv.
- That self-containment applies to the environment package — it does not prove that all serving or reward judging runs offline.
- Two separate evaluation sets: FinQA (290 samples, general financial Q&A) and FinQA Reasoning (79 samples, harder multi-table questions).
- These are evaluation counts, not training examples; the OpenEnv README marks its 290-question dataset as evaluation-only.

### The trained agent inspects, then recovers · 13:42
- The trained 4B model beats the 235B comparator on this evaluation, with pass@1 (success on one attempt) roughly doubling from its own baseline.
- The released model, rLLM-FinQA-4B, is fine-tuned from Qwen3-4B-Instruct-2507.
- Back at the revenue question, it starts by discovering tables with `get_table_names` — a tool the larger model also had but did not use.
- It inspects the schema next, then issues a query; that query asks for a missing `revenue` column, but the agent reads the error and corrects it.
- The winning trajectory: discover tables, inspect schema, query, repair the invalid column, then answer from retrieved data.
- Once the two revenue values are in hand, the arithmetic is tiny — a short SQL expression with a guard against dividing by zero.

### Easy training, harder questions · 16:23
- An ablation compared three task-selection regimes: single-table only, mixed, and a curriculum that adds multi-table tasks gradually.
- Single-table-only training produced the largest uplift; adding harder tasks, immediately or progressively, did not beat it.
- The benefit carried over to the harder multi-table FinQA Reasoning benchmark.
- FinQA Reasoning pass@1: Qwen3-4B-Instruct-2507 at 13.9%, Qwen3-235B-A22B at 18.9%, and rLLM-FinQA-4B at 26.6%.
- Single-table training does not mean a single tool call — the successful revenue example still needs a sequence of interactions.
- Crawford reads the transfer as evidence that tool use was the core bottleneck.

### Rubrics find the behavior worth training · 18:30
- A plain correct/incorrect score says whether the agent succeeded but not where the process broke.
- Suggested rubrics break a response into answerable checks: did it discover tables, inspect schema, respond to the error, ground its answer?
- These distinguish a retrieval failure from an arithmetic failure and guide which data to generate next.
- In Crawford's GRPO setup the optimizer still receives one scalar reward — rubrics diagnose, the scalar reward drives training.
- He points to a Snorkel companion study and the Agentica partner post for detail; there was no time for Q&A.

## Resources

- [How Tool Discipline Let a 4B Model Outsmart a 235B Giant](https://snorkel.ai/blog/how-tool-discipline-let-a-4b-model-outsmart-a-235b-giant-on-financial-tasks/) — Snorkel's companion write-up of the study.
- [rLLM-FinQA-4B model card and weights](https://huggingface.co/rLLM/rLLM-FinQA-4B) — the trained financial agent, with benchmark scores and tool details.
- [OpenEnv FinQA environment](https://github.com/huggingface/OpenEnv/blob/main/envs/finqa_env/README.md) — the containerized financial tool-use environment.
