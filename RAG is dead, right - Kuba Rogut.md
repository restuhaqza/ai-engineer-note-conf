# RAG is dead, right??

| Field | Details |
| --- | --- |
| **Speaker** | Kuba Rogut |
| **Affiliation** | turbopuffer |
| **Type** | Talk |
| **Date** | 2026-06-09 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=UM6sFg_jdlE) |
| **Tags** | `coding-and-developer-tools` `rag-context-and-search` |

## Summary

Kuba Rogut, a deployed engineer at turbopuffer (a full-text and vector search database), takes on the claim that RAG is dead. His argument: the slogan is really about one narrow implementation — a single vector lookup — not retrieval-augmented generation as a whole.

RAG simply means retrieving useful context and then generating a response. The retrieval step is not limited to embeddings; it can use semantic similarity, BM25 full-text search, grep, glob patterns, regular expressions or plain filters. The same confusion surrounds "agentic search", which is often used as shorthand for grepping files even though it describes an agent using tools to progressively find and reason over context.

Cursor is the worked example. It indexes codebases for semantic search and reuses existing team indexes instead of re-embedding everything every time. It also reports real accuracy gains from the approach, while Claude Code took the other path and stayed with iterative grep-based search.

Rogut's framing is that embeddings are cached compute: index once, then retrieve cheaply at runtime. Modern retrieval happens many times inside a reasoning loop, and even very large context windows still need a way to select the right tokens.

## Key Takeaways

- **"RAG is dead" conflates RAG with one vector-search pipeline.** The term covers any way of finding context before generating.
- **Retrieval is broader than vectors.** Semantic search, BM25 full-text search, grep, glob, regex and filters all count.
- **Agentic search is broader than filesystem grep.** It means giving agents tools to iteratively find and reason over context.
- **The loop is search, read, assess, repeat.** The feedback between reading and searching is what makes it agentic.
- **Cursor reuses codebase indexes.** **Merkle trees** detect what changed, a **simhash** picks a similar existing index, and **content proofs** keep reuse secure.
- **The reported numbers:** about 12.5% higher answer accuracy on average across models on Cursor's internal benchmark; online, 2.6% higher code retention and 2.2% fewer dissatisfied requests with semantic search.
- **Embeddings are cached compute.** One-time indexing turns repeated discovery into lightweight runtime queries.
- **Big windows still need selection.** As Rogut puts it, you want the right million tokens, not a trillion at once.

## Detailed Notes

### What is supposed to be dead? · 00:24
- Rogut works at turbopuffer, which builds a full-text and vector search database on top of object storage.
- Social media posts claim RAG is dead and agentic file search is all anyone needs.
- Against that, a search-interest chart rises in 2023, flattens through 2024, and climbs again around mid-2025 — evidence of continuing interest, though not of retrieval quality.
- The more useful question: has the debate confused RAG with one particular implementation of it?

### Retrieval methods and the agentic loop · 01:37
- The narrow view of RAG: embed a corpus, search it with a query embedding, and pass the results to an LLM.
- But RAG does not require vectors. Retrieval can use semantic similarity, BM25 full-text search, grep, glob patterns, regular expressions or ordinary filters.
- Agentic search has the same naming problem. Because Claude Code and Codex expose filesystem search, the term becomes shorthand for grepping files.
- Rogut's broader definition: an agent using tools to progressively find and reason over external context.
- The loop runs search, read what was found, assess whether it answers the question, and search again if a gap remains. The feedback between reading and searching is the distinguishing feature.

### Cursor reuses codebase indexing work · 03:23
- Cursor, one of turbopuffer's earliest customers, prepares a codebase for semantic search by parsing files, splitting them into chunks, and computing embeddings.
- Doing that from scratch for every developer repeats work the team has already done, since colleagues mostly open the same repositories and branches overlap.
- The reuse mechanism separates detecting change from finding a reusable index. Rogut summarizes it with **Merkle trees**, hash trees that expose differences in file contents; Cursor also uses a derived **simhash** to select a similar existing index.
- After reuse, changed content is updated while unchanged chunks keep their cached embeddings.
- Security matters because a shared index may contain files a client does not have. Cursor's account explains **content proofs** that exclude results for files the client cannot prove it possesses.

### What semantic search contributed · 04:46
- Cursor's internal **Cursor Context Benchmark** reports about 12.5% higher answer accuracy on average across models when semantic search is available — a vendor-reported internal evaluation, not a public benchmark.
- Rogut highlights almost 24% for the earlier **Composer** model, before Composer 2; the accessible article text reports a maximum of 23.5% without naming the model. These are relative improvements, not percentage-point changes.
- An online A/B test reports 2.6% higher code retention with semantic search on repositories with at least 1,000 files.
- It also reports 2.2% more dissatisfied requests without semantic search, using the semantic-search group as the baseline.
- Small online numbers are expected because not every request uses the added tool, which dilutes the average effect.

### Embeddings as cached compute · 06:06
- Claude Code is the counterexample. Rogut cites Boris Cherny saying early versions tried RAG with a local vector database and dropped it in favor of agentic search.
- That records a design choice; it is not proof that semantic retrieval is wrong for every coding agent, and Rogut offers no detailed technical explanation.
- His framing: embeddings are **cached compute**. Some work an agent would otherwise repeat while exploring a codebase can be moved into reusable preprocessing.
- He contrasts an illustrative Claude-Code-style trace (repeated runtime discovery) with a Cursor-style trace (upfront indexing, then retrieval). These are illustrations, not a controlled benchmark of the two products.

### Finding how metadata filtering works · 07:03
- Example question: how does metadata filtering work in this codebase?
- A filesystem agent starts by finding candidate files, reading them, and deciding whether it knows enough. In a TypeScript repository a first search might be `grep -Rni --include='*.ts' -- 'metadata' src/`.
- That discovery repeats across sessions and developers. Rogut describes roughly 6,000 tokens for the discovery sub-step in his illustration — not for an entire task.
- The indexed path moves some work earlier: parse, chunk and embed once, then answer a lightweight query like "How is metadata filtered?".
- The upfront index is not maintenance-free, and the example gives no universal speedup or break-even threshold.

### From one lookup to retrieval during reasoning · 08:38
- The pattern being displaced is the simplest early RAG pipeline: one vector search, put the results in the context window, generate an answer — associated with 2023 and early 2024.
- Among more sophisticated customers, Rogut sees agents making many retrieval calls across several reasoning steps.
- Semantic search and full-text search become tools the agent chooses as needed, fetching context for the current question and using what it learns to guide the next search.
- **Retrieval becomes part of reasoning** rather than a fixed prelude, so the investigation can narrow or redirect instead of trusting the first lookup.

### Larger context still requires selection · 09:42
- Rogut recalls Jeff Dean's point about long-context models: even an enormous context window leaves a need for staged retrieval that narrows a large collection down to what matters.
- His phrasing of that idea: you do not need a trillion tokens at once, you need the right million.
- He reports customers with trillions of tokens in turbopuffer, while the useful subset for a single task might be ten thousand, a hundred thousand, or a million tokens.
- Those are different quantities: the corpus available to search and the context worth supplying to the model.
- As context windows grow, the retrieval system still has to decide what belongs inside them.

## Resources

- [Improving agent with semantic search](https://cursor.com/blog/semsearch) — Cursor's offline answer-accuracy evaluation and online A/B test.
- [Securely indexing large codebases](https://cursor.com/blog/secure-codebase-indexing) — how Cursor reuses team indexes and synchronizes changed files.
- **Hybrid Search** (turbopuffer documentation) — a practical guide to combining vector and BM25 retrieval and merging ranked results.
- **Owning the AI Pareto Frontier — Jeff Dean** — podcast discussion of long context and staged retrieval.
