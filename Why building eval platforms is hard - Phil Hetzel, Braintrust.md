# Why building eval platforms is hard

| Field | Details |
| --- | --- |
| **Speaker** | Phil Hetzel |
| **Affiliation** | Braintrust |
| **Type** | Track keynote |
| **Date** | 2026-04-28 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=_fQ7Z_Wfouk) |
| **Tags** | `evals` `rag-context-and-search` `infrastructure-and-deployment` |

## Summary

Phil Hetzel leads solutions engineering at Braintrust. Before that he spent 12 years in consulting, where he watched clients build convincing generative-AI proofs of concept that never reached production. The missing ingredient, he argues, is confidence: evaluation builds it while you experiment, and observability checks whether it still holds once real users show up.

He walks through the stages teams pass through as they mature. The smallest useful setup is a spreadsheet and a for-loop, which is a fine place to start but soon becomes **documentation rather than experimentation**. A custom UI and a real database persist results and invite more people in, but if users can only inspect what already happened, it is still a reporting tool. Experimentation arrives when they can change a configuration and compare the outcome — a playground, plus an SDK path for technical users.

The interesting shift is that evaluation and observability become the same systems problem. Production traces reveal the failure modes you never imagined, which then become offline eval examples, improving the agent in a continuous loop. That loop makes the platform a tracing and logging system too, and agent traces are unusually nasty: large, fast, text-heavy, and read in very different ways. Hetzel's conclusion is blunt — the UI is easy to vibe-code, but the data layer underneath is the hard part.

## Key Takeaways

- **Confidence is the deliverable.** Evaluation builds it before launch; observability confirms it continues to hold afterward.
- **Start with a spreadsheet and a for-loop.** It has no barrier to entry and is a real first step — just know it documents rather than experiments.
- **A nicer UI is not an experiment.** Letting users inspect results is reporting; letting them change something and compare is experimentation.
- **Build scoring around failure modes, then find them in production.** Real traces reveal behavior your imagined examples miss.
- **Evaluation and observability are one system.** The flywheel observes, analyzes, evaluates real examples offline, then improves and repeats.
- **Building the platform creates ongoing maintenance.** The initial UI may be small; keeping the full loop running is a continuing job.
- **Traces are a distinct data workload.** Large, semi-structured, fast-arriving, and read both for instant inspection and for aggregate analysis.
- **The next user may be a coding agent.** Headless workflows need real data access, such as SQL, so agents can pull aggregate eval results and improve the agent themselves.

## Detailed Notes

### What keeps a working prototype out of production? · 01:33
- Hetzel's consulting background (four years at KPMG, eight at Slalom, where he led the global Databricks unit) exposed a recurring gap: great prototypes, few production launches.
- He began using Braintrust as a user to help close that gap, then joined the company about a year before the talk.
- Braintrust has two related pillars. **Evaluation** builds confidence while a team experiments before deployment; **observability** checks that confidence still holds once real users interact.
- Both concern the same behavior at different points in the agent's life.
- LLM variability is the root difficulty: the flexibility that lets models reason across many problems also makes behavior hard to anticipate.
- As customers expect agentic experiences, that uncertainty becomes a product concern — brand, compliance, cost, maintenance, and systems risk.

### A spreadsheet and a for-loop · 06:08
- The smallest useful eval setup needs three ingredients: a way to execute the agent, input examples to initiate runs, and a results UI (a spreadsheet is fine).
- An "input example" is not necessarily a question; it is whatever information is needed to invoke the agent.
- The visible pieces are the tip of an iceberg. Underneath sit several kinds of expertise: product and AI engineers shape behavior, systems engineers make it run, and subject-matter experts judge whether responses make sense.
- An eval platform must support that collaboration rather than assume engineers can evaluate in isolation.
- Start with the loop anyway: every time the agent changes, rerun the examples and record outputs.
- But it degrades into **documentation rather than experimentation**. Comparing runs across a growing sheet is cumbersome, aggregate analysis is limited, and human scoring does not scale.
- The sheet also excludes people: nontechnical contributors with the strongest domain knowledge may not join a manually maintained results document.
- Recreating or appending the sheet after every evaluation adds friction exactly where fast iteration is needed.

### A nicer UI is not yet an experiment · 10:47
- The next step is usually an engineer declaring they can just vibe-code Braintrust. Keep the for-loop, replace the sheet with a friendlier interface, and persist results in a database (Hetzel gives Neon as an example).
- That improves retention and lets the UI fit the reviewers — but if users can only inspect what happened, it is still mainly a reporting tool.
- Experimentation requires letting users change something and evaluate the consequences.
- A **playground** supplies a sandbox around an agent configuration, exposing selected parameters to adjust.
- Technical users also need an SDK-driven path; the UI should not be the only way to run an experiment.
- His example exposes the system instructions for an agent running outside the eval platform: a user edits them, compares two system-prompt configurations, and runs evals across both runs, surfacing scores.
- The platform can control an experiment without owning the agent's execution environment — it needs access to the configuration, a way to run the agent, and a way to compare results.

### Use production failures to choose the next experiment · 13:54
- A playground makes experiments possible but does not say what to test.
- **Build scoring functions around failure modes**, then use production traces to find which failure modes actually occur.
- This is where evaluation and observability merge. Real users expose behavior initial examples never anticipated.
- Origin story: an early customer piped production traffic into the database and ran a massive evaluation every hour. That workaround showed the need for direct tracing and observability instead of forcing live usage into an offline experiment.
- The resulting loop is continuous:

1. **Observe** the agent in production.
2. **Analyze** interactions to find failures worth fixing.
3. **Evaluate** using real examples brought back into an offline environment.
4. **Improve** the agent and repeat throughout its lifetime.

- The dataset stops being just imagined requests and becomes a way to bring observed behavior back into development.
- This widens scope: the platform is now a tracing and logging system as well as an offline eval platform. Real interactions give higher-signal examples, making offline evaluation closer to rerunning production safely.
- The same scoring functions can run against observability traffic as **online evals**, supporting alerts.
- Organizational consequence: someone must operate and extend the platform as agent development changes. "If you build it, you have to manage it."

### The data layer behind the trace viewer · 17:17
- Agent traces put unusual pressure on storage: lots of text, a mix of semi-structured and unstructured content, and rapid arrival under real usage.
- Illustration of the mismatch: putting a one-gigabyte trace into a single Postgres row invites serious performance problems. It is a design warning, not a measured limit.
- Two access patterns must be served at once.

| Access pattern | User expectation | Data-layer requirement |
|---|---|---|
| Inspect a recent trace | See what just happened | Low-latency ingestion and visibility |
| Analyze many traces | Understand behavior in aggregate | Efficient analytical reads |

- Braintrust's former architecture used separate persistence layers, including an open-source data warehouse for aggregate analysis, stitched together with `BTQL` — a domain-specific language Hetzel says nobody liked, including the team.
- A third aggregation layer ran DuckDB in the browser. It worked for a while, then coordinating the layers became complex.
- Text-heavy customers exposed the limits: a company like Notion sends huge amounts of unstructured data and wants full-text search across traces, which that combination struggled to serve.
- **Agent quality is a systems problem, not just a UI problem.** Users expect full-text search across millions of traces; a polished interface cannot deliver that if the data platform cannot retrieve and analyze the content.
- A **span** (one part of a trace) can itself be large — Hetzel contrasts traditional spans of a few kilobytes with agent spans he has seen at 10–20 megabytes due to their context.
- Large unstructured payloads plus fast ingestion, aggregate reads, and low-latency inspection make a hard joint workload. None of these is new alone; satisfying them together is the distinctive problem.

### The next user may be a coding agent · 21:13
- The data platform also decides whether evaluation can support automated improvement.
- In one workflow, a coding agent pulls aggregate evaluation data into its context and changes the evaluated agent within the same session — which requires useful, viewer-independent access to the data.
- Hetzel presents SQL access as important for making that practical.
- Teaching schema example: an `eval_results` table with `agent_version`, `failure_mode`, and `passed` columns lets an agent investigating a candidate version group failures by category.

```sql
SELECT
    failure_mode,
    COUNT(*) AS evaluated_cases,
    SUM(CASE WHEN passed = FALSE THEN 1 ELSE 0 END) AS failed_cases
FROM eval_results
WHERE agent_version = 'candidate_b'
GROUP BY failure_mode
ORDER BY failed_cases DESC;
```

- That gives a coding session a compact set of failure categories to investigate before proposing a change. The table is a teaching schema, and SQL access does not imply full PostgreSQL compatibility.
- He reports growing interest in **headless eval workflows**: users want Codex or Claude Code to improve their agent without visiting the platform UI at all.
- The backend must serve agents as well as human analysts. He shows an architecture slide but passes over implementation detail for time.

### Find unknown usage patterns and govern collection · 22:46
- Retrieving known failures is only part of the job; a mature platform should surface behavior you did not know to look for.
- **Topic modeling** helps understand how people use an agent without reading a mountain of traces, so engineering time goes where it matters.
- As that analysis becomes part of development, both humans and agents need to consume it.
- Operating at scale adds non-functional requirements: **role-based access control** to govern platform data and capabilities, and **data masking** for sensitive content in collected data.
- **Automatic tracing** through an AI proxy or gateway captures LLM interactions centrally, making tracing part of the request path rather than an optional developer action.
- The gateway becomes a point of central governance — further platform responsibilities beyond running experiments and presenting results.

### Keep media review in the trace · 24:11
- Dynamic prompts can include files and videos, not just interpolated strings; teams often end up building custom solutions around eval tools.
- Hetzel narrows the closing question to multimodal inputs, outputs, and traces.
- His answer separates media storage from trace review: put media in object storage, reference it from the trace, and render it inside the trace interface.
- A reviewer can play audio or video where they already inspect behavior, without leaving the platform.
- Storing an attachment should not be confused with indexing all of its contents for search.
- Prompt ownership is a separate decision: prompt management can live in Braintrust, or it does not have to — just as the playground can evaluate an agent running outside the platform.

## Resources

- [Log multimodal attachments](https://www.braintrust.dev/docs/instrument/attachments) — attaching media and large JSON objects to traces.
- [Query Braintrust with SQL](https://www.braintrust.dev/docs/reference/sql) — current SQL and legacy BTQL syntax, with sandbox and CLI options.
- [DuckDB full-text search](https://github.com/duckdb/duckdb-fts) — DuckDB's official extension for indexing and searching text.
- "How Brainstore works" — Braintrust's write-up on trace ingestion, indexing, and querying.
- "Introducing Brainstore" — the original architecture announcement with workload-specific benchmarks.
