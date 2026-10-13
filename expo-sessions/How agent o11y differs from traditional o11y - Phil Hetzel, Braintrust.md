# How agent o11y differs from traditional o11y

| Field | Details |
| --- | --- |
| **Speaker** | Phil Hetzel |
| **Affiliation** | Braintrust |
| **Type** | Expo session |
| **Date** | 2026-05-28 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=XBaznoTRDFI) |
| **Tags** | `agent-engineering` `rag-context-and-search` `observability-and-reliability` `evals` `coding-and-developer-tools` |

## Summary

"o11y" is shorthand for observability — the practice of watching how software behaves once it is live. Phil Hetzel, who leads solutions engineering at Braintrust, argues that AI agents need a different kind of observability than ordinary applications. Classic tools still answer the operational questions: is the service up, how fast is it, are there HTTP errors. That work still matters, but it cannot tell you whether the agent's answer was actually any good.

Agents are non-deterministic (the same task can take different paths), so a quick, error-free response can still be wrong. Judging quality means keeping the real evidence: the context the app gathered, the tools the agent chose, the system prompt, and the answer itself. That evidence is bulky and text-heavy, which turns trace storage and search into a genuine systems problem rather than a metrics dashboard.

Hetzel walks through how Braintrust handles it: a purpose-built database, a write-ahead log for near-instant visibility, a full-text index for searching inside trace text, and one SQL-like query path. He also stresses that the people best placed to spot a bad answer are often domain experts, not infrastructure engineers, so the improvement workflow has to include them.

His closing framing is that evaluations and observability are the same machine used two ways, and that a healthy quality workflow keeps room for both the failures you know about and the ones you haven't met yet.

## Key Takeaways

- **Keep your old monitoring.** Grafana/Datadog-style tools still cover uptime, latency, and errors; agent observability adds a layer, it doesn't replace them.
- **Fast does not mean correct.** A prompt reply can still fail the task, so quality checks are needed: grounding, tool choice, and alignment with the system prompt.
- **Traces get huge.** Hetzel reports customer traces larger than 1 GB and single spans as big as 20 MB, mixing structure with a lot of unstructured text.
- **Text search is a first-class feature.** "Find every trace that mentions X" needs a full-text index, which metrics and status codes can't provide.
- **Two read patterns must coexist.** People want a fresh interaction almost instantly, while developers want SQL/CLI access to query and auto-improve at scale.
- **Domain experts matter.** Clinicians, nurses, advisors, and lawyers can read traces and improve agents because prompts are plain language.
- **Humans seed the automated scores.** Experts grade and explain; those explanations become reusable scoring functions.
- **Evals and observability are one system.** Evals run in batch with known inputs; observability handles real-time unknown inputs — sharing infrastructure lets production traces become test cases.

## Detailed Notes

### Why prototypes stall before production · 00:52
- Hetzel spent 12 years in consulting and systems implementation, including leading Slalom's global Databricks practice.
- He noticed customers were prolific at building AI proofs of concept but poor at getting them into production.
- He used Braintrust as a customer first, then joined to lead solutions engineering.
- He frames agent quality as two questions: is the live agent performing as expected, and can you trust a new version when you change it?

### Keep the operational basics · 02:48
- Teams ask why they need another tool when they already run Grafana or Datadog; those still answer availability, latency, and 400/500 error questions.
- Braintrust itself uses Datadog to watch for website errors — agent observability doesn't remove that need.
- The vocabulary carries over: a *metric* is a measurement over time, a *trace* is one full workflow interaction, and a *span* is one step inside it.

### A fast response can still be wrong · 04:56
- Ordinary application code deliberately fixes its control flow; an agent can take different paths to the same task, which makes it both useful and hard to characterize.
- Time to first token, total tokens, duration, and latency are still worth tracking, but a quick reply can still fail.
- Quality adds questions: was the answer grounded in the gathered context, were the expected tools used, and does it follow the brand rules in the system prompt?
- Answering those needs far more of the trace to be kept than a duration or a status code.

### Big traces, instant inspection · 07:22
- Agent traces combine a semi-structured hierarchy with large amounts of unstructured text.
- Hetzel reports customer traces over a gigabyte and individual spans as large as 20 MB — cited as examples, not typical sizes.
- As usage grows, engineers and product managers still expect to see an interaction almost immediately.
- Two access patterns must both work: fast interactive inspection, and programmatic SQL/CLI queries used to improve the application automatically.

### Building a database for text-heavy traces · 09:53
- Braintrust designed a database specifically for agent traces instead of reusing a conventional store.
- New data lands in a write-ahead log so it becomes visible quickly, while background indexing keeps filters and analytical queries fast. Here "visible" means not waiting on indexing, not zero ingestion latency.
- They forked Tantivy — a Rust full-text search library, closest in spirit to Apache Lucene — to index the text inside traces, for example finding every trace that contains "Amazon".
- A unified SQL-like query layer ties the log, the indexes, and text search together, and streams results to the UI.

### Domain experts belong in the loop · 11:45
- Traditional observability serves systems and product engineers; agent quality pulls in people close to users or to the domain.
- Hetzel names clinicians, registered nurses, wealth advisors, and lawyers who read traces to improve agents.
- Natural-language prompts give non-technical contributors a direct way to influence behavior.
- He attributes the strongest agent teams to collaboration between technical and non-technical people.
- Braintrust also rolled out topic modeling over incoming traces about a month earlier: a lightweight LLM plus embeddings and clustering to surface user intent, sentiment, and issues.

### From expert judgment to automated scores · 15:24
- An audience question separates functional quality (Braintrust's focus) from non-functional performance; tracing also yields the technical metrics for free.
- Human annotation is more than a grade, and the workflow has a clear shape:
  - inspect the interaction and decide whether it went well;
  - record *why* it got that grade;
  - use those justifications, often with an LLM, to identify the failure mode;
  - turn that failure mode into a reusable automated score.
- The written justification is what connects domain expertise to repeatable checks.

### Evals, observability, and unknown failures · 17:12
- Hetzel treats evals and observability as the same problem: evals run in batch with inputs known in advance, observability handles inputs that arrive from users in real time.
- Sharing one system lets a live interaction become a test case — capture the trace, add it to an offline dataset, then experiment on it.
- Asked about the custom database, he says Braintrust previously used ClickHouse and moved away, needing text indexes it did not adequately have there at that time. He frames this as a historical reason, not a benchmark.
- Final point: scores catch the failures you have learned to recognize, while open-ended analysis helps you find the next ones. A quality workflow needs both.

## Resources

- [Brainstore architecture](https://www.braintrust.dev/blog/brainstore) — how the object storage, write-ahead log, and Tantivy index fit together (article).
- [Tantivy](https://github.com/quickwit-oss/tantivy) — the Rust full-text search library Braintrust forked (repository).
- Introducing Braintrust Topics — automated grouping of production traces by user task (article).
- Inside the Topics pipeline — how trace summaries become embeddings, clusters, and named topics (article).
- Build evaluation datasets from production traces — assembling versioned test sets (documentation).
- Filter and query Braintrust logs — searching production logs via filters, SQL, and the CLI (documentation).
- ClickHouse full-text search availability — the state of native text search in ClickHouse (article).
