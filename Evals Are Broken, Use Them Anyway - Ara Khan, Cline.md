# Evals Are Broken, Use Them Anyway

| Field | Details |
| --- | --- |
| **Speaker** | Ara Khan |
| **Affiliation** | Cline |
| **Type** | Talk |
| **Date** | 2026-06-06 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=QuuIywMG4s8) |
| **Tags** | `coding-and-developer-tools` `agent-engineering` |

## Summary

Ara Khan (Cline, an open-source coding-agent company) argues that most people
are wrong about evals — short for *evaluations*, automated tests that score how
well a model or agent performs a task. One camp treats a leaderboard number as
objective truth, assuming two models with similar scores behave the same way.
The other camp ignores numbers entirely and goes on vibes.

His middle path is that evals are genuinely useful without being definitive.
He gives practical heuristics for reading other people's numbers, then shows how
Cline grew tired of trusting vibes and built its own eval from real user tasks.

The talk's core insight is that an agent eval measures three things at once: the
model, the agent *harness* (the surrounding prompts and tools), and the quality
of the tasks themselves. Resources and reasoning settings also move the score.
His closing advice is to "hill climb" — pick a benchmark, improve honestly — but
keep checking that the product still feels good to use.

## Key Takeaways

- **Two wrong camps.** Treating a benchmark score as truth, or ignoring scores
  for pure vibes, are both mistakes. Useful evaluation sits in between.
- **Scores are approximations.** Do not take a model lab's headline number at
  face value; supplement it with your own trials.
- **Stay current, don't be first.** Wait a couple of weeks after a release, then
  switch only if the model still holds up.
- **Seek recent, precise, relevant evals.** Standardized benchmarks age; older
  ones may no longer measure frontier coding.
- **Grade the result, not just the answer.** An agent's reply is one artifact of
  a long process that must leave the system actually working.
- **Isolate every task.** Each task needs its own environment, repo, and agent;
  otherwise you measure provisioning failures, not the agent.
- **Read failures as a repair queue.** Group failed runs by cause to find levers
  that fix many tasks at once.
- **Don't overfit.** Maximizing a score can become cheating; a good number is not
  a substitute for a product that works well.

## Detailed Notes

### When the same score does not mean the same model · 01:42
- Khan opens with a dashboard showing GPT-5.4 and Gemini 3.1 Pro Preview side by
  side with similar scores. The numbers make them look interchangeable.
- They are not the same in practice: a scalar score describes one particular
  evaluation, not the whole model.
- He points to a same-day tweet criticizing Meta for "benchmark maxing" —
  headline numbers that do not survive real-world use.
- The opposite camp anthropomorphizes models ("I like talking to her"). That
  preference says something about the interaction, not about reliability.
- **Evaluations are useful without being definitive.** There are three levels:
  interpret others' evals, improve your agent against an eval, and build your own
  eval for a specific use case. He covers the first two.

### Use external scores to decide what to investigate · 03:37
- **Heuristic 1:** treat lab numbers as approximations — sometimes useful,
  sometimes not. Do not believe them at face value.
- **Heuristic 2:** stay current without being the earliest adopter. He shows an
  Epoch aggregate index where the frontier leader changes every few months.
- His advice: let a new release sit for a couple of weeks, see what early users
  expose, then switch if it still holds up. You don't need to test every release.
- **Heuristic 3:** look for evaluations that are recent, precise, and relevant.
- He cites OpenAI's write-up arguing SWE-bench Verified (a benchmark built from
  real GitHub issues and patches) no longer measures frontier coding, due to
  flawed tests and contamination.
- He objects to elementary exercises standing in for real software engineering.
  The essay notes his Fibonacci and matrix examples express that broader point,
  not Terminal-Bench's actual task format.

### From intuition to tasks users actually perform · 06:24
- Improving an agent has two parts: an engineering problem (build reliable test
  infrastructure) and a philosophy problem (choose the right tasks).
- Coding has an effectively unlimited space of requests and failure modes, so a
  finite suite can only approximate the real work.
- Cline saw that available standardized evals did not match day-to-day
  programming. Like the Codex team and others, it leaned on "vibes" at first.
- Eventually dismissing measurement stopped being enough, so Cline built tasks
  itself.
- The raw material came from users who opted in to share their Cline usage. The
  team paid for examples, then did heavy manual effort to clean them into
  problems another coding agent could attempt.
- Real-world provenance gave relevance; careful preparation made them gradable.

### Grade the repair, not just the answer · 08:55
- A bounded factual question is easy to grade: send a prompt, get an answer,
  score it. Khan jokes about guessing how many toes a cat has. The point is the
  small answer space, not the guess.
- An agent task is different. Given a broken MCP server (a service an agent
  calls), an agent might read files, search docs, install dependencies, configure
  the environment, run scripts, and finally run tests.
- The reply is only one artifact. The evaluator must check whether the resulting
  system works — and whether the fix broke something else.

### Give every task an isolated environment and run them in parallel · 10:11
- Cline started using **Terminal-Bench**, credited to Stanford researchers. Its
  tasks cover race conditions, database failures, and infrastructure problems.
- A CLI interface lets different coding agents attempt the same tasks. Some take
  a couple of minutes; others take 30–40 minutes, exposing agents that circle or
  lose their way.
- The suite Khan describes has 89 tasks, matching Terminal-Bench 2.0's count.
- To run a task you need: an isolated environment (such as a VM), the repo plus
  full setup to reproduce the problem, and the agent installed inside it.
- This makes the environment part of the evaluation. Without it, a run may
  measure a provisioning failure instead of the agent's ability to repair.
- **Harbor** (from the Laude Institute) handles the infrastructure: define a
  standard Linux/CPU/memory config per task, then run the isolated environments
  concurrently. Backends include Daytona, local Docker, and Modal, which Cline
  uses. With full parallelism, the slowest task becomes the main limit.

### Turn failed runs into a repair queue · 12:58
- After a baseline run, inspect the failures instead of staring at the aggregate.
  Picture 50 failures out of 89 tasks.
- Have another agent read the failed runs' traces — every LLM call — and group
  them by cause. One task may fail because the agent skipped tests; another
  because its read-file tool was broken.
- Grouping makes recurring defects visible and points to changes that could help
  several tasks at once.
- Interpreting the next score means separating three things. What is tested:
  the model, the harness, and task quality. What can distort it: a strong model
  masking a weak harness, tools fitted to one model, or tasks measuring the wrong
  work.
- He illustrates model–harness fit with Anthropic models used through Cursor,
  Droid, and Claude Code — the same model can look stronger when its harness
  serves it better.
- **A useful result needs the model, harness, and task to line up.**

### Resources and reasoning both change the outcome · 14:51
- Khan reports an initial score of roughly 43% on Terminal-Bench.
- Cline then adjusted container CPU and memory, raised timeouts, and changed
  thinking behavior. He does not give a final score or enough detail to isolate
  each change's effect.
- So resource settings are part of the result's conditions, not incidental
  infrastructure.
- More thinking did not always help: he describes cases where it lowered quality
  and produced loops, including repeating a phrase for roughly 2,000 tokens.
- Reasoning is another variable to test, not a dial you can always turn up.
- Cline keeps an internal benchmark across model versions, including open-source
  models, to keep watching these differences.

### Improve the agent without overfitting, and keep the product experience · 15:38
- Benchmark improvement falls into three zones: fix obvious flaws (crashes,
  rate-limit failures), tune model-specific behavior, and stop before
  overfitting.
- Zone two is where most useful engineering happens. Prompting that works for
  Anthropic models may not work for the Codex or Gemini families. Tuning
  instructions and prompt length is how you fix a good model that underperforms
  in your agent.
- Zone three is a danger zone: optimizing purely to publish a high score can
  become cheating.
- His closing loop is **hill climbing**: get a score, improve the harness, repeat.
  But you have to do both — keep the objective score and the subjective "does it
  feel good to use" check.
- Cline was strong on Anthropic models and weaker on Gemini and other families.
  Hill climbing improved that support, letting people who preferred those models
  use Cline better — expanding usefulness rather than reinforcing a preference.
- He invites people to reach out to discuss these problems, including evaluation
  questions outside coding agents.

## Resources

- [Why SWE-bench Verified no longer measures frontier coding capabilities](https://openai.com/index/why-we-no-longer-evaluate-swe-bench-verified/) — OpenAI analysis cited by the speaker
- [Terminal-Bench: Benchmarking Agents on Hard, Realistic Tasks in Command Line Interfaces](https://arxiv.org/abs/2601.11868) — the paper behind the task suite
- [Run Terminal-Bench with Harbor](https://github.com/harbor-framework/docs/blob/main/examples/terminal-bench.mdx) — examples for selecting agents and filtering tasks
- Harbor resource configuration — documentation for task resource declarations
- Configure Modal CPU, memory, and disk — resource requests and limits for Modal
