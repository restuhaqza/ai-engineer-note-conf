# Self-Driving Products: From Product Signals to Pull Requests

| Field | Details |
| --- | --- |
| **Speaker** | Joshua Snyder |
| **Affiliation** | PostHog |
| **Type** | Talk |
| **Date** | 2026-06-10 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=zMiSRliEzv4) |
| **Tags** | `developer-workflows-and-testing` `rag-context-and-search` `safety-and-governance` `evals` `apis-mcp-and-protocols` `coding-and-developer-tools` |

## Summary

Joshua Snyder of PostHog describes a pipeline that turns observability data — analytics, session replays, errors and customer messages — into pull requests instead of dashboards to read. The idea is that a product signal could produce a prepared change rather than another alert.

The premise is that today's loop is slow. A metric changes, someone notices hours or days later, files a Linear issue, and eventually writes and ships a fix. Snyder wants a background agent to investigate as soon as the signal arrives and open a GitHub PR, so developers review a candidate fix instead of raw logs and errors.

The pipeline has five stages: ingest signals, group related ones into a report, research the cause and the owning repository, assess whether it is actionable, then execute code and iterate until CI checks pass. Agents run the Claude Agent SDK inside Modal sandboxes, and they pull extra context from PostHog's MCP server plus external tools such as Linear and Notion.

Snyder ends with four lessons. Evals must use representative production data, embeddings should match the problem rather than the input format, an agent will "fix" a vague report unless a gate withholds it, and cost optimization should come after you understand which steps are expensive. At recording time the system is in alpha.

## Key Takeaways

- **Turn signals into PRs, not dashboards.** The target is a prepared change ready for review, or a safe change shipped behind a flag.
- **Group before you code.** An error, a Slack message and a session replay can all describe one underlying problem.
- **Screen for prompt injection first.** An LLM classifier drops signals that try to give the agent malicious instructions.
- **Match embeddings to meaning.** Raw signal embeddings grouped by format; embeddings of LLM-generated queries grouped by the actual problem.
- **Research needs product, code and org context.** The agent pulls extra data via PostHog's MCP server and grounds itself with Linear and Notion.
- **Add an actionability gate.** Route reports to "gather more evidence", "needs human judgment" or "immediately actionable" — and let it reject vague ones.
- **Iterate until the PR is green.** CI failures and review comments rehydrate a sandbox snapshot and trigger another run.
- **Evaluate on real data, then optimize.** "Tokens are free" is a provocation, not a fact, but premature cost-cutting hides what a pipeline can do.

## Detailed Notes

### From dashboards to pull requests · 00:25
- PostHog collects product analytics, session replay, web analytics, error tracking and experiments — several views of what users experience.
- Today: a signal changes a dashboard metric, someone notices later, files a Linear issue, then writes a patch, reviews it and ships it. Snyder calls this slow and uninteresting.
- Alternative: the signal triggers a background agent that investigates and opens a GitHub PR. The developer reviews it, or a low-risk change ships immediately behind a feature flag.
- The intended handoff is a prepared change, not merely an alert.

### The five-stage pipeline · 02:57
- PostHog ingests trillions of events a month; much of that is noise, and different observations can describe a single failure.
- Stages: **ingest** signals; **group** related ones into a report; **research** the cause and the owning repository; **assess** actionability; **execute** the fix and iterate until checks pass.
- Research and assessment sit between detection and implementation because an observed symptom is not yet a well-defined coding task.
- The event volume describes PostHog's platform, not the throughput of the agent pipeline.

### Screen unsafe content, then normalize · 03:56
- Some sources are public, which opens a prompt-injection path: an attacker could trigger an error whose text tells the agent to publish customer data.
- An LLM classifier at the top of the pipeline drops signals it judges unsafe. This is a stated safeguard, not a guarantee that accepted content is safe.
- Accepted signals are normalized into one shape: source product, type, content, and an importance weight.
- `type Signal = { sourceProduct: string; type: string; content: string; weight: number; }`
- The pipeline then embeds the content of the signal.

### Group by the underlying problem, not the format · 05:09
- A null-pointer exception and a Slack message saying checkout is broken look different but may be the same failure.
- Grouping collects signals into a **report**. Their weights feed the report's importance, and once it crosses a threshold the pipeline promotes it and starts a research agent.
- The first attempt failed: embedding raw signals clustered by structure. A checkout error and an onboarding error grouped because both looked like errors, while the onboarding Slack message stayed with other Slack messages.
- Fix: ask an LLM what each signal is about, generate search queries from it, then match those queries in embedding space.
- Comparing representations: raw signal embeddings emphasize shared format; embeddings of generated queries emphasize the problem described.
- Snyder reports this made grouping substantially better but gives no numerical evaluation.

### Research with product, code and org context · 07:28
- The research agent runs the **Claude Agent SDK** inside a **Modal sandbox**.
- Its starting evidence is the promoted report, but it can investigate further through **PostHog's MCP server** — for example pulling logs for a report that already contains an error and a replay. Snyder credits that extra evidence with making research more accurate.
- It also receives codebase context and external MCP servers. Linear and Notion were especially useful for grounding the investigation in organizational context.
- Output: a problem summary, a priority, and a proposed reviewer identified with `git blame` — preparing both the technical explanation and the human handoff.

### An actionability gate before code · 08:45
- A researched problem does not automatically become a coding task. The gate routes it according to what is missing:
  - **Not enough evidence** → back to the pool to gather more signals.
  - **Needs human judgment** → a review inbox for the morning.
  - **Immediately actionable** → the coding agent.
- Returning a report to the pool preserves the opportunity to act later.
- Source matters: specific error reports (Sentry-style) give agents well-defined problems, while Slack messages and replays often describe broader difficulties with several plausible fixes.

### Iterate until the PR is green · 09:59
- For an actionable report, the execution stage clones the repository into a sandbox and runs the Claude Agent SDK to build a fix.
- The first push is not the endpoint: a failing CI check or a new PR comment triggers another sandbox run.
- The pipeline saves a **sandbox snapshot** and rehydrates it to continue; feedback can come from a reviewing agent as well as other PR comments.
- The loop continues until checks pass, aiming to replace morning CI failures with review-ready PRs.
- Green checks are the target of the loop; they do not by themselves prove a change is correct or reach auto-merge.

### Four lessons from building it · 11:01
- **Evals first.** Local vibe checks on the team's own data missed the variety of customer data. Representative evaluations and production visibility are needed to tell a real improvement from a familiar-input illusion.
- **Embed the right thing.** With mixed formats, structural similarity competes with semantic similarity. Improving results meant changing what is fed to the embedding model, not the clustering method.
- **Specificity is required.** Give an agent a generic "onboarding is broken" report and it will fix something, producing noisy PRs. The gate must be allowed to withhold a task even when the agent could patch it.
- **Optimize later.** Snyder's line "tokens are free" is deliberately provocative and immediately corrected. Early cost worry led the team to avoid or delay agents, which limited discovery.
- Running the same problem repeatedly (100 runs is his illustration, not a required sample size) surfaced patterns that could later become a one-shot LLM call or a faster trained model.

### Deploy, measure, and learn · 13:36
- At recording time the system is an alpha pipeline: signals become grouped reports and merge-ready PRs, with rollout planned over the following months.
- The larger ambition: developers spend their day on features while the system handles routine bugs and experiments on pricing or onboarding.
- A proposed extension: for easy changes, an agent approves and deploys behind a feature flag; if results are poor, roll back the flag and remove the change from the codebase later.
- The next improvement is learning from outcomes after a PR — a rejection, a deployment problem, or an error resolved in production each give different feedback for the next change.
- Closing advice: if a product already produces a lot of user-behavior data, experiment with agents on that evidence and see what they find.

## Resources

- [Claude Agent SDK overview](https://code.claude.com/docs/en/agent-sdk/overview) — building agents with tools and MCP connections.
- [Modal Sandbox snapshots](https://modal.com/docs/guide/sandbox-snapshots) — saving and restoring sandbox state.
- [PostHog MCP](https://posthog.com/docs/model-context-protocol) — connect clients to query product data and investigate errors.
- **What we wish we knew about building AI agents** (PostHog engineering, related) — lessons on agent harnesses, evals and production.
- **Securely deploying AI agents** (PostHog docs, related) — controls for agents handling untrusted content.
- **PostHog's self-driving product** (PostHog overview, update) — the current product page describing the automation.
