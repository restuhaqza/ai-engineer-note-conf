# Fighting AI with AI

| Field | Details |
| --- | --- |
| **Speaker** | Lawrence Jones |
| **Affiliation** | incident.io |
| **Type** | Talk |
| **Date** | 2026-05-17 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=L2r6vLlLgs8) |
| **Tags** | `coding-and-developer-tools` `developer-workflows-and-testing` `evals` |

## Summary

incident.io builds an incident-response platform and is working on "AI SRE" (site
reliability engineering — keeping production systems healthy): systems that
investigate production problems automatically. One investigation can run hundreds
of queries across logs, metrics, and traces, then point at a likely cause and fix.

Founding engineer Lawrence Jones starts from a hard question. Reading an
investigation report is easy, but judging whether it is *correct* is not —
establishing that often takes an engineer about an hour. Behind every report sit
hundreds or thousands of prompts, so the real problem is understanding the system
that produced the answer.

His answer is to debug AI with AI. The talk walks through three techniques: prompt
evals as small unit tests, turning debugging interfaces into downloadable file
systems that coding agents can read, and repeatable analysis pipelines that turn an
aggregate score into an explanation.

The through-line: internal debugging tools deserve the same deliberate use of AI as
the product itself.

## Key Takeaways

- **An eval is a unit test for a prompt:** give input, run the prompt, grade the output against explicit criteria.
- **Keep the criteria separate.** Two independent criteria (pirate style vs. meaning preserved) make behavior testable without pinning one exact wording.
- **Production interactions make giant fixtures.** A real incident can produce about 2 MB of YAML, which can fill a coding agent's context; a small CLI that edits one case at a time fixes this.
- **Package prompt repair as a runbook.** The loop: add a failing eval, change the prompt until it passes, check regressions, then simplify.
- **Find the guilty component first.** Real systems are many prompts, and a red–green eval loop only helps once you know where to change.
- **Export debugging UIs as a file system.** Coding agents navigate files well, so a downloadable, grep-able package beats UI tools they can't use.
- **Backtests explain trends.** A daily batch gives a number like "86% accurate root-cause analysis"; exporting it lets agents cluster failures and say *why* it moved.
- **Bulk files beat an MCP layer or computer use here** — Jones frames this as his engineering judgment, not a measured comparison.

## Detailed Notes

### How do you know an investigation is right? · 00:17
- incident.io pages engineers, helps run incidents, and handles customer communication; customers include Netflix, Etsy, and Skyscanner.
- The bigger goal: automate production investigations, from major incidents down to routine tickets.
- An investigation runs hundreds of telemetry queries (logs, metrics, traces, past incidents) and cross-references them with the codebase.
- Judging one report is slow — often about an hour of reconstructing the incident before you can say whether it was useful.
- Behind the report sit hundreds or thousands of prompts across many customer environments with different failure modes.
- The real debugging problem is understanding the system that produced the answer, not reading the answer.

### Start with a prompt you can test · 03:01
- Jones' earlier LDX talk covered the base concepts: prompts, evals, scorecards, traces, datasets, backtests.
- An **eval** supplies input, runs the prompt, collects the output, and applies grading criteria to pass or fail.
- Evals live in YAML files beside the Go code that defines the prompts; the AI work is written in Go at incident.io.
- The demo prompt translates a message into pirate speak, with two criteria: pirate style, and meaning preserved.
- A colorful translation that changes the request fails; a faithful translation with no pirate style also fails.
- Keeping the criteria separate makes the intended behavior clear without forcing one acceptable wording.

### Make large eval fixtures editable · 05:37
- Real incident fixtures are messy: the trigger may be nearly an entire incident report.
- A "steal from production" button exports a bad interaction straight into the repo as an eval, so you test the exact failing input.
- That solves capture, not maintenance.
- A good unit test is focused; a production export can bury the expectation in a huge YAML document.
- A fixture of about 2 MB can exhaust a coding agent's context before useful work starts.
- The internal **Eval Tool** gives a smaller interface: list cases, and edit, replace, or add one.
- With that, the agent works on the relevant fixture and can also follow a runbook for changing prompts.

### Give prompt repair a repeatable procedure · 07:24
- Packaged as a runbook or skill, the repair loop is:
  1. Inspect the prompt and the requested behavior.
  2. Create an eval that demonstrates the current failure.
  3. Modify the prompt until that eval passes.
  4. Run the other evals to check for regressions.
  5. Consolidate and simplify the prompt.
- The last step matters: a series of individually reasonable additions slowly makes a prompt hard to maintain.
- Demo: a real prompt that turns human requests into Loki log queries; the agent adds an eval and iterates to an acceptable pass rate.
- The talk does not give the repeat count or the acceptance threshold.
- This makes a *known* prompt safer to change; it does not yet say which prompt caused a bad interaction.

### Find the error inside the hierarchy · 08:51
- A production interaction rarely belongs to one prompt; the incident chatbot maps to a hierarchy of prompts, tools, and agents.
- Roughly ten agents are visible, and the full graph does not fit on screen.
- A customer can flag a bad response without knowing which component produced it.
- Investigations add another layer: each timeline step expands into a trace, and a "green" block can contain hundreds of prompts and tool calls.
- A subtle intermediate error can propagate until the final root-cause analysis is simply wrong.
- The team's human UIs expose these details, but engineers lack time to follow every branch, and coding agents can't use those UI tools.

### Download the debugging interface as files · 10:47
- Insight from Claude Code: agents are good at navigating file systems with standard tools.
- incident.io exports everything behind a debugging UI as a file system, dropped into a sandboxed Claude Code session.
- The package includes the prompt inputs and a self-documenting structure, plus the application codebase.
- Traces can be written as ASCII text, preserving the relationships the UI showed visually.
- Practical session flow:
  1. Download the bad interaction into the sandbox.
  2. Ask the agent to explain what it thinks went wrong.
  3. Describe the desired behavior and ask what should change.
  4. Make the change using the codebase.
  5. Verify it with the eval runbook.
- Diagnosis and repair happen in one session: the human sets intent, and the agent traces the implementation.

### Turn backtest scores into explanations · 12:37
- The team runs thousands of investigations daily across hundreds of customer accounts.
- A **backtest** batches investigations against incident.io's own account and customer accounts to track improvement.
- One example reports 86% accurate root-cause analysis; the talk gives no dataset size, grading rubric, or evaluation window.
- The aggregate can't say why the score rose or fell, or what a specific customer needs.
- The investigation batch is exported as files and fed to a Claude Code analysis pipeline.
- An internal repo called **Scrapbook** holds Markdown playbooks describing how to inspect the evidence.
- The process first fans out (about 25 agents in parallel), then clusters findings into cohorts with similar failures.
- The output explains patterns behind an account's performance and proposes improvements instead of restating the score.

### Carry the analysis through to a production change · 14:43
- Good pipelines keep their work: subagents save incremental findings to files so the analysis can stop and resume.
- Giving the analysis the codebase lets it link a diagnosed failure to an implementation location and propose a concrete change.
- The coding agent can then implement that proposal in the same session and check it with the red–green eval process.
- Example: a merged pull request (`#51443`) feeds previous hypothesis-review feedback into the next hypothesis build.
- The handoff is tangible: investigation evidence → feature discussion → code change → deploy and test in production.
- Jones' closing point: use AI in internal tools as deliberately as in the product, and make debugging evidence usable by coding agents.
- He favors bulk file downloads because an agent can `grep` through them, judging an MCP layer or computer use as less effective for this workflow.

## Resources

- "Becoming AI engineers" — Jones' earlier LDX talk on evals, debugging tools, and engineering practice (mentioned).
- "Why we built our own AI tooling suite" — incident.io's internal Workbench account (related).
- "You can't vibe code a prompt" — worked examples of prompt tuning that breaks existing cases (related).
- "Query Loki" — Grafana documentation for LogQL, the query language used in the demo (mentioned).
- "Building AI skills like checklists" — later walkthrough of a daily AI spend-analysis skill (update).
