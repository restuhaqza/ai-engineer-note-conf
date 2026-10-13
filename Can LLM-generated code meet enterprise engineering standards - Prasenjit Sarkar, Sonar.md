# Can LLM-generated code meet enterprise engineering standards?

| Field | Details |
| --- | --- |
| **Speaker** | Prasenjit Sarkar |
| **Affiliation** | Sonar |
| **Type** | Talk |
| **Date** | 2026-05-31 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=NuePCNMpWGc) |
| **Tags** | `coding-and-developer-tools` `safety-and-governance` |

## Summary

Coding agents now write a lot of production code. Prasenjit Sarkar (Sonar) asks a simple but uncomfortable question: if you only look at benchmark pass rates, do you actually know whether that code is secure, maintainable, and safe to keep in an enterprise codebase?

To answer it, Sonar ran more than 4,444 distinct Java programming assignments from an open dataset through several models and analyzed the results with SonarQube Enterprise. The headline finding is that functional correctness hides big differences. Models that score well on tests can still generate far more code than needed and carry very different numbers of security issues per million lines.

He argues that test pass rates (from benchmarks like HumanEval, MBPP, and SWE-bench) only show that tested requirements were met. They say nothing about reliability, architecture fit, technical debt, or the effort the next developer will spend. Newer models also tend to write more lines and more branching, and their remaining bugs get subtler.

Sarkar closes with Sonar's ACDC framework — agent-centric development cycle: **guide, verify, solve**. The idea is to improve the inputs before generation, check the output inside the agent's own loop, and return leftover findings to a remediation agent that proposes fixes for humans to review.

## Key Takeaways

- **Passing tests is not enterprise readiness.** Accuracy on a benchmark does not tell you whether code is secure, maintainable, or aligned with your architecture.
- **Same workload, very different volume.** For the same ~4,444 Java assignments, the models shown produced anywhere from ~307,000 to ~1.2 million lines.
- **Higher pass rate can come with more code.** Gemini 3.1 Pro High led the shown pass rate (84.17% from SWE-bench), yet its 307,000 lines for the corpus were still dense in defects.
- **Security findings vary by model.** Sarkar flags Claude Sonnet 4.6 as the highest security-issue density in his snapshot, at about 300 issues per million lines.
- **Complexity matters as much as length.** Newer models trend toward more lines and higher cyclomatic and cognitive complexity, so review effort grows.
- **Mistakes are getting subtler.** As models improve, total bugs and vulnerabilities shift toward finer issues that are harder for a human to notice.
- **Where defects come from matters.** He points to mixed-quality training data, insecure patterns, hidden logic errors, probabilistic generation, and missing project context.
- **Fix it in the loop, not only at CI.** ACDC's verify stage uses SonarQube Agentic Analysis over MCP (a standard way for agents to call tools) to check generated code before commit.

## Detailed Notes

### Who checks the code after the agent writes it? · 00:20
- The workflow has shifted: humans write instructions in English and review what the agent produces.
- Tooling moved from IDEs (VS Code, JetBrains) to AI editors (Cursor, Windsurf, Antigravity) to agentic platforms (Codex, Claude, Devin, Gemini CLI).
- Sarkar cites a March 2026 Pragmatic Engineer survey where 55% of respondents said they regularly use AI agents — a read on that survey's readers, not all developers.
- The review questions stay concrete: is the output maintainable, secure, and readable?

### Functional correctness leaves engineering questions open · 02:00
- Vendors promote pass rates on HumanEval, MBPP, and SWE-bench, which measure tested functional correctness.
- Missing from those scores: security, real-world reliability, architectural fit, engineering discipline, and accumulated technical debt.
- Sonar's evaluation covers 4,444+ distinct Java assignments, with generated solutions analyzed in SonarQube Enterprise.
- The dataset is open source; the analysis results were made public (Sarkar corrected an early slip calling the analysis itself open source).

### A high pass rate can accompany substantial defects · 03:38
- Gemini 3.1 Pro High showed an 84.17% pass rate — a SWE-bench figure that does not map onto Sonar's Java corpus.
- On the Java corpus it generated about 307,000 lines, with 614 bugs and 210 security issues per million lines.
- Claude Sonnet 4.6 is identified as the highest security-issue density in the shown set: ~300 issues per million lines over ~627,000 lines.
- GPT-5.4 and GPT-5.4 Pro High were associated with roughly 1.2 million lines, without the totals being separated.
- More output is not itself better: it gives reviewers and maintainers more to inspect.

### Where unreliable output can come from · 05:27
- **Mixed-quality training data:** sound implementations sit alongside weaker code from open source and elsewhere.
- **Insecure patterns:** vulnerable examples can be learned next to secure ones.
- **Hidden logic errors:** subtle mistakes in training code can lead to generated code that fails or misbehaves.
- These are proposed explanations; the evaluation does not isolate each cause.
- Probabilistic generation means the same prompt can yield different code and different volumes.
- Without project context, an agent lacks company conventions, codebase structure, and architectural constraints — and it often can't explain why it chose a bad implementation.

### Read the leaderboard across several dimensions · 06:45
- The Sonar LLM Leaderboard tracks 53+ models and versions, including thinking configurations like Gemini 3 Pro High vs. Gemini 3 Pro.
- Sarkar stresses that model selection is about the exact version and mode, not just the family.
- Alongside pass rate he examines issue density, generated lines, cyclomatic complexity, and cognitive complexity.
- In the shown snapshot all five models exceed 80% functional correctness, so the other measures are what separate them.
- Volume grows with generations: GPT-5.2 High wrote ~1 million lines, versus under 250,000 for the older model he labels GPT-4.0.

### Measure paths through code and the effort of understanding it · 08:36
- **Cyclomatic complexity** counts control-flow paths: decisions and loops (`if`/`else`, `for`, `while`) create alternative routes through a function.
- **Cognitive complexity** is a Sonar measure of how hard code is for a person to read and maintain.
- He contrasts a nested loop that totals positive values with a guard-clause version: both return the same result, but the guard version is easier to follow.
- Counting branches alone cannot capture maintainability, which is why the two measures are used together.
- The takeaway: the cost is not just how many lines an agent writes, but how much branching and reasoning those lines impose on the next developer.

### Guide generation before verifying its output · 10:32
- ACDC (agent-centric development cycle) has three stages — **guide, verify, solve** — over an inner loop around generation and an outer loop around committed changes.
- **Sonar Sweep** cleans problematic training data at its source; it was in private beta at the time.
- **Sonar Context Augmentation** supplies relevant project context to the agent so output fits the existing codebase; it was already in open beta.
- The point is to improve both the examples a model learns from and the project information available when it generates.

### Return analysis findings to the agent before committing · 11:41
- The verify stage puts SonarQube analysis inside the agent's working loop.
- **SonarQube Agentic Analysis** was in open beta, reachable through MCP by tools like Claude, Codex, and Gemini CLI.
- Sarkar reports 1–5 seconds for pre-commit analysis versus 1–5 minutes for CI (workload conditions not specified).
- Workflow: generate code, request analysis before committing, return issues to the agent to repair, then commit and open the PR.
- The pre-commit loop does not replace PR analysis — it just lets the agent fix problems earlier.

### Propose repairs, check them, and return them for review · 12:39
- The solve stage handles issues that still reach the pull request and fail the quality gate.
- The **SonarQube Remediation Agent** (open beta) can be asked to fix issues in a PR.
- It also works on existing tech debt: select issues in the dashboard, assign them to the agent, and get one PR per issue.
- Before returning a fix, the agent re-runs analysis and compilation and discards changes that create issues.
- These are proposed changes: developers review, approve what they accept, and merge — no automatic merge on the model's confidence.
- Closing claim: guidance, analysis, and remediation live inside the tools where code is generated, submitted, and reviewed.

## Resources

- [Sonar LLM Leaderboard](https://www.sonarsource.com/the-coding-personalities-of-leading-llms/leaderboard/) — model pass rates alongside code volume, complexity, and static analysis.
- [AI Tooling for Software Engineers in 2026](https://newsletter.pragmaticengineer.com/p/ai-tooling-2026) — Pragmatic Engineer survey of 906 readers.
- The Coding Personalities of Leading LLMs — Sonar's report explaining its Java evaluation.
- Sonar's Agent Centric Development Cycle — the guide/verify/solve framework announcement.
