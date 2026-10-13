# Benchmarking semantic code retrieval on Claude Code

| Field | Details |
| --- | --- |
| **Speaker** | Kuba Rogut |
| **Affiliation** | turbopuffer |
| **Type** | Expo session |
| **Date** | 2026-06-03 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=zKk7sDMGDEQ) |
| **Tags** | `architecture` `coding-and-developer-tools` `rag-context-and-search` |

## Summary

Coding agents usually find code by grepping and reading files. The alternative is to build a searchable index of what the code means, then retrieve from it. Kuba Rogut, a deployed engineer at turbopuffer (a serverless full-text and vector search database), built a semantic search tool for Claude Code and measured whether it actually helps.

His frame is that **embeddings are cached compute**: embedding code up front stores a reusable representation of meaning so agents do not rediscover the same areas of a repository in every session. Cursor made this bet and reported real gains; Claude Code instead found iterative grep-based search simpler and better.

The experiment compares three setups — stock Claude Code, windowed reads of 50 lines, and windowed reads plus semantic search — against a human-labeled benchmark that measures the *retrieval path*, not just whether the final code works. Semantic search sharply improved precision, but recall stayed mixed, and different tasks favored different tools. His conclusion: the winning approach is lightweight, complementary retrieval tools, and an agent that knows when to use each.

## Key Takeaways

- **Two ways to navigate a repo:** lexical grep follows names and imports; semantic search follows similarity of behavior. They find different code.
- **Embeddings are cached compute.** Indexing stores reusable meaning, whereas grep-and-read work repeats across every session and every agent.
- **ContextBench measures the journey.** It uses human annotations of which files, lines, and symbols an agent should inspect — not just whether the task was solved.
- **A read limit matters.** Reading whole 1,000-line files hid the differences between strategies, so reads were capped at 50 lines to make the numbers meaningful.
- **Precision jumped:** stock Claude Code hit 65% file precision; windowed reads plus semantic search reached about 87%. Windowed reads alone also improved it.
- **Recall was mixed.** Stock Claude Code had the highest file recall because it explores broadly; adding semantic search slightly lowered aggregate recall over the 50 tasks tested.
- **Task type decides the winner.** Semantic search won when related behavior used different keywords (e.g. across several ORMs); grep won when an early search found a good identifier and imports led the way.
- **A tool is not a policy.** Claude Code is built for grep, so bolting on a semantic tool does not give it reliable judgment about when, how, or with what query to use it.

## Detailed Notes

### Why index code when an agent can grep? · 00:16
- turbopuffer is a serverless full-text and vector search database built on object storage; Rogut works with it as a deployed engineer.
- At recording time, Claude Code relied on agentic filesystem search (grep-and-read) by default. Boris Cherny (a Claude Code creator) noted early versions used a local vector database, but iterative search proved simpler and worked better.
- Cursor took the opposite path, indexing codebases into turbopuffer for semantic retrieval. Rogut cites roughly 24% relative answer-accuracy improvement for Composer and about 12.5% averaged across models (results predate Composer 2).
- Cursor also ran an online A/B test measuring user outcomes, not benchmark accuracy: 2.6% higher code retention with semantic search on codebases of at least 1,000 files, and 2.2% more dissatisfied follow-ups when it was disabled.
- Small aggregate numbers can hide large gains, because many requests have little use for semantic search.

### Embeddings amortize repeated discovery · 02:44
- Question posed: how does metadata filtering work in this codebase? A grep approach starts broad, reads matches, and searches again when they do not answer.
- Rogut's illustrative trace spends about 6,000 tokens on this discovery — one example, not an average — and the same work recurs in other sessions and other agents on the same repository.
- The indexed route moves work earlier: split code into chunks, embed and store them, then retrieve relevant chunks at runtime.
- **Embeddings are cached compute** in that sense; they cache a reusable representation of meaning, not a complete answer. The agent still reads and reasons about what comes back.
- Indexing is not free: changed code needs the index maintained. Payoff depends on how often the representation is reused versus the cost of building and maintaining it.

### Give Claude Code a semantic retrieval tool · 04:12
- **TurboGrep** implements a small version of the pipeline: walk the filesystem, parse and chunk with `tree-sitter`, embed with a Voyage code model, and upload to turbopuffer.
- Claude Code then gets a search tool backed by that index; at recording time V1 was open source and V2 was pending release, so the public repo should not be assumed to reproduce the exact demo.
- The demo uses the Django repository: the agent calls the `TPuff` search tool for a password reset token generator, gets relevant content, and explains the implementation.
- A convincing trace shows the integration works in that example, but not yet that retrieval improves across tasks.

### Measure the context found along the way · 05:02
- Rogut separates Cursor's internal context benchmark from the public **ContextBench** work, which uses human annotations of files, lines, and symbols an agent should inspect to do a task well.
- This evaluates the retrieval trajectory rather than just the final code. Retrieval precision and recall are not the same as answer accuracy or task completion.
- Three conditions were compared:
  - **Stock Claude Code** — default search and reads.
  - **Windowed reads** — reads limited to 50 lines at a time.
  - **Windowed reads + semantic search** — same limit plus the `TPuff` Search tool.
- The 50-line cap exists because reading an entire thousand-line file sweeps in annotated context without showing whether the agent located it precisely, making measurements noisy.

### Precision improves with more selective retrieval · 06:24
- **Precision** = relevant retrieved items divided by all retrieved items. Read 10 files and 8 are required, and file precision is 80%.
- Reported approximate results: stock Claude Code 65% file, 33% line, 43% symbol precision; windowed reads plus semantic search reached about 87% file precision.
- Windowed reads alone also improved precision, so the full gain over stock cannot be attributed to semantic search alone.
- Rogut describes default Claude Code as highly exploratory: it reads broadly before settling. His rounded reading is roughly one in three baseline reads outside the target set, one in five with windowed reads, and one in eight with semantic search added.
- Calling those reads "wasted" is shorthand for being outside the benchmark's annotations, not proof that every exploratory read was useless.

### Recall reveals what selective search can miss · 07:30
- **Recall** changes the denominator: relevant retrieved items divided by all required relevant items. Ten required files, five read but only three required, gives 30% file recall.
- Reading fewer irrelevant files can raise precision without raising coverage.
- Stock Claude Code had the highest file recall because broad exploration encounters more target files.
- Line recall was weaker: reaching the right file does not mean inspecting the right parts of it.
- Windowed reads and windowed reads plus semantic search had roughly similar aggregate recall, with some decreases when semantic search was added. The aggregate covers 50 tasks, not the full public benchmark.

### Behavioral similarity and import tracing find different code · 08:38
- Splitting tasks by which condition wins reveals differences the aggregate hides:
  - **Semantic search** helped when relevant files implemented related behavior with different keywords — e.g. related handling spread across several ORMs or libraries, where lexical search missed some implementations.
  - **Windowed grep** helped when an early search found a good identifier, because the rest of the work followed explicit links such as imports.
- These are different ways to navigate a repository: one follows similarity of behavior, the other follows names and explicit code relationships.
- The task breakdown supports choosing tools based on that structure rather than declaring one search method universally better.

### A useful tool still needs a tool-selection policy · 09:43
- The experiment improved precision, but the agent also has to decide when and how to use the new tool.
- Rogut's interpretation: Claude Code is oriented toward grep, so attaching a semantic-search tool does not automatically give it a reliable policy for choosing it, phrasing a query, or deciding when to fall back to lexical search.
- He contrasts this with Composer's built-in retrieval behavior. That is a plausible explanation for why an added tool may fall short of an integrated system, not a causal result isolated by this experiment.
- The two evaluations also measure different things — Cursor's answer accuracy versus this context retrieval — so their gains cannot be compared directly.
- His design principle: provide lightweight, complementary ways of finding context. Even very large context windows benefit from selecting relevant information before the model reasons over it.

### What semantic search means here · 11:07
- In the audience Q&A, semantic search here means vector similarity search: embed the code, embed the query text, and search stored vectors in turbopuffer to retrieve code chunks.
- The embedding model is [voyage-code-3](https://blog.voyageai.com/2024/12/04/voyage-code-3/).
- Rogut names Cursor, Anthropic, and Notion as turbopuffer customers, but a customer relationship does not reveal the retrieval architecture of each product — it does not by itself establish semantic search inside Claude Code.

### Comments help a chunk explain itself · 11:58
- Asked how it performs on poorly written code, Rogut gives no controlled benchmark answer, only a pattern from reviewing trajectories.
- Examples with useful function comments and inline documentation tended to retrieve better, because comments supply behavioral context that both the embedding model and the coding agent use.
- The hard part is not producing a vector but making the chunk's meaning available to the representation: syntax shows what operations happen, not why the code exists or which user-facing behavior it supports.
- This stays a qualitative observation, not a measured rule that comments always help.

### Bridge the gap between a question and raw code · 12:58
- A follow-up points to a representation mismatch: a natural-language query and raw code can express the same behavior very differently.
- The audience proposes a parent–child arrangement where a query-like description points to a child holding the actual code. Rogut clarifies this experiment did not do that preprocessing — it embedded raw code chunks.
- Authentication flow is the audience's example of a human-level concept that may not appear directly in implementation text. A description at that level could be a useful retrieval target while the linked code remains the evidence to inspect.
- Rogut mentions Cursor's own embedding model as a way to translate between code and human queries but says he cannot explain its internals. A suggestion that Cursor generates comments and embeds them with code is unverified in the discussion; Rogut agrees it could work without confirming it is Cursor's method.

### Where shared vector retrieval earns its place · 14:22
- On long-term memory and database size, Rogut answers in terms of workload placement rather than when to encode or compact memories.
- If all relevant information can be downloaded locally, grep is convenient and needs little setup — though the agent's repeated searching and reading still consumes compute.
- Shared workloads change the math: a Notion-like knowledge base with awkward cross-information relationships is hard to gather and grep as local files, so a shared vector index can serve multiple agents.
- Multimodal data extends the case further: grep cannot recover the semantic content of raw video, audio, or images — matching a filename is not the same as finding what the media contains.
- Local lexical search stays useful when material and identifiers are readily available; as agents share larger or more varied information spaces, storing reusable semantic representations offloads repeated work.

## Resources

- [Improving agent with semantic search](https://cursor.com/blog/semsearch) — Cursor's offline answer-accuracy evaluation and online A/B test.
- [TurboGrep repository](https://github.com/turbopuffer/turbogrep) — public code-search CLI with a semantic-query example.
- [ContextBench: A Benchmark for Context Retrieval in Coding Agents](https://arxiv.org/abs/2602.05892) — human-annotated retrieval benchmark.
- [Voyage code embedding model announcement](https://blog.voyageai.com/2024/12/04/voyage-code-3/) — background on voyage-code-3.
