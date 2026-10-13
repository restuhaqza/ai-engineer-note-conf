# Mergeable by default: Building the context engine to save time and tokens

| Field | Details |
| --- | --- |
| **Speaker** | Peter Werry, Brandon |
| **Affiliation** | Unblocked |
| **Type** | Workshop |
| **Date** | 2026-05-03 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=5ID22ACI7IM) |
| **Tags** | `coding-and-developer-tools` `leadership` `apis-mcp-and-protocols` `rag-context-and-search` `developer-workflows-and-testing` |

## Summary

Peter Werry (Unblocked) argues that a coding agent begins a task like a new employee: it can read the code but does not know which code matters, what the team expects, or why the existing implementation looks the way it does. A **context engine** exists to supply the context a task needs and to exclude the rest. The talk works through three common myths about how to build one and then builds a piece of it live with the audience.

The myths are that naive retrieval over documents is a context engine, that a pile of MCP (tool-calling) servers gives understanding, and that a bigger context window solves the problem. Werry's counterargument is that **access does not equal understanding**. A related failure, borrowed from radiology, is **satisfaction of search**: an agent finds one plausible answer in Notion, Confluence, or the code, stops looking, and misses the decisive constraint in a Slack thread or incident report.

The workshop then builds a **social engineering graph** that maps who reviews whose pull requests and where expertise sits, and shows how that graph acts as a pivot into deeper context. Along the way the presenters are candid about lessons learned: optimize for understanding, not access; surface conflicts the engine cannot resolve rather than hiding them; and never cache a generated answer and serve it again.

## Key Takeaways

- **Agents start at ground zero.** A context engine supplies the context a task needs and, importantly, none it does not.
- **Access is not understanding.** Wiring up MCP servers or a vector store does not explain the relationships between sources or the reasons behind decisions.
- **Beware satisfaction of search.** An agent can stop at the first plausible answer and miss the real constraint in a Slack thread or incident report.
- **Permissions travel with evidence.** Context from private channels is only used for people who already have access, and answers stay private.
- **Planning is where context pays off most**, followed by review; ticket enrichment, production triage, incident management, and engineering support also benefit.
- **Do not cache answers.** Code, docs, and reasons change; feeding old generated answers back in spreads mistakes.
- **Corrections become memory.** Feedback is distilled into "task memory," weighted by an expert graph.
- **One task comparison:** the same migration took about 2.5 hours with vendor MCP servers (four correction loops) versus 25 minutes with Unblocked, and 21M versus 10M tokens.

## Detailed Notes

### Agents start where a new employee starts · 01:11
- Werry defines **context engineering** as supplying the context an agent needs, and none it does not, in an optimized way, so it works in line with the organization's practices.
- Four years ago, the developer was the context engine: paste in the issue, explain constraints, inspect the result, and redirect the agent when it misunderstood — sometimes as basic as "not the JavaScript, the Python."
- A new employee builds the same knowledge gradually: exploring the code, finding a mentor, and living through incidents and outages. Those experiences preserve not just implementation details but why the team rejected an approach or kept an awkward dependency.
- He walks an adoption curve: in 2022, autocomplete-era context windows were about 8K tokens, so tools used the code around the cursor; ASTs, language servers, and caller retrieval added structure; then parallel agents with MCP and skills; then background agents and agent teams.
- The human cost grows too: managing several tasks means switching between their assumptions and unfinished decisions. Moving work to the background needs a system that already understands the code and the organization.
- He references rising coding capability (he mentions Mythos, which he had not yet tried). Without context, a capable agent can complete a whole task incorrectly, leaving a corrective loop — what he calls a **doom loop**.

### A migration that erased a needed dependency · 09:43
- The first concrete example is a migration to Anthropic's adaptive thinking. The older integration passed an explicit token budget to control the thinking window.
- An agent connected to MCP servers implemented the new behavior but removed code that still supported legacy dependencies. It could find the implementation without understanding why those older paths remained.
- With the context engine connected, he reports the agent recovered those reasons, changed the correct locations, and preserved compatibility for callers using the old behavior.
- The lesson: the engine had to make the "why" available, not just the "where."

### Three myths: RAG, MCP servers, bigger windows · 10:48
| Approach | What remains unresolved |
| --- | --- |
| Naive RAG over documents | Conflicts and task-specific relevance |
| A collection of MCP servers | Relationships and reasons across sources |
| A larger context window | Evidence selection and interpretation |

- With naive retrieval (RAG, or retrieving documents by similarity), an agent can search repeatedly, burn its token budget, and reach compaction without finding the decisive information. In a large organization, broadly relevant results may come from the wrong repo or a team with different conventions, so personalization is part of retrieval quality.
- On long context: early long-context models were good at needle-in-a-haystack lookup — if you already knew what you wanted, they could find it. Werry distinguishes that from reconciling sources and choosing the right organizational action.
- Even a much larger window leaves a selection problem. An organization's material can exceed the window, and fitting more of it in does not decide which statements are current, mistaken, or relevant. He extends the thought experiment to tens of millions of tokens; the design question is how to select trustworthy evidence, whatever the capacity.
- The opposite failure is stopping too soon. **Satisfaction of search**, borrowed from radiology, is when finding one explanation causes the reader to stop looking and miss another abnormality. Agents similarly stop at a plausible answer in Notion, Confluence, or code while the key constraint sits elsewhere.
- The iceberg image: code that compiles is the visible tip. Below it are the user's original intent, approaches the team already rejected, experiments that failed, and information lost when code or documents were deleted — which require history, not just the current repo.

### Turn connected evidence into usable context · 15:48
- A context engine needs to know who is asking, their team, their collaborators, and who understands the relevant code. It must distinguish resolvable conflicts from disagreements that need a human, and when it cannot decide, it should expose the conflict and learn from the clarification.
- **Permissions follow the evidence.** For private Slack channels, Unblocked uses their contents only for a questioner who already has access, and an answer built on that private information stays private. Repository restrictions and secret projects need the same discipline.
- The architecture has many inputs (planning tools, docs, conversations, source code, PRs) and many outputs (agents via MCP or CLI, apps via API, code review in source control, messaging integrations).
- Building relationships starts with easy links, such as a Slack message containing a PR URL. Understanding a practice takes more: historical PR comments can be distilled into guidance, repeated patterns combined into organizational memories, and those memories loaded when an agent meets similar code.
- Conflict resolution evolved through failed shortcuts. Preferring recent material failed when a newer document misstated the system; preferring code on `main` improved things but cannot answer a question about intended future behavior. Historical evidence shows what to avoid; expert discussions show where the system is meant to go.
- Targeted retrieval and personal relevance work together: PR and contribution history identify the repos a person usually works in, letting the engine retrieve deeply there and more broadly elsewhere without excluding useful evidence.
- In an initial larger-task comparison, Werry reports the baseline repeated mistakes whose history it had failed to recover while the context-enabled run avoided them. He explicitly warns against trusting the quality scores Claude generated for the comparison.

### Where context pays off: planning, review, support · 23:31
- Three implementation lessons. First, providing a knowledge graph and integration-specific tools did not make an agent understand the organization. Second, simplistic conflict-resolution rules hid disagreements the engine could not settle; those cases needed to become visible. Third, never cache a completed answer and serve it for a similar question — code, docs, and reasons change, and feeding old generated answers back in makes mistakes self-reinforcing.
- The workflows differ in where evidence pays off:
  - **Planning and review:** planning is the strongest intervention point; at review time, organizational motivation supplements checks for broken behavior and security.
  - **Ticket enrichment:** an agent fills gaps in a feature request using existing organizational context.
  - **Production triage:** related past issues give a starting point for a new problem.
  - **Incident management:** Sentry and Datadog signals connect to code, past incidents, and Slack discussions.
  - **Engineering support:** the engine answers product questions from customer-success and sales teams inside shared channels.
- Teams package this access as repository-hosted skills, a ticket-enrichment action taking an issue ID, or an incident-timeline workflow, and also connect Claude Code to the engine in CI with an API credential.
- A review example: Unblocked told an author to reduce duplicated tests — a practice distilled from earlier PRs, one of which the same author had written. The system reflected the team's convention back to its source.
- The presenters' target: generated code should feel like it came from a colleague with years of experience on the team.

### What the engine is; permissions and synthesis · 32:05
- Asked what the artifact actually is, Werry says it is all of these: agents reach it through MCP or a CLI; people ask questions in a dashboard personalized to their identity and recent work; Slack supports conversational and automatic answers. No single interface defines it.
- Karpathy's "LLM wiki" and decision traces come up as possible components. A compiled wiki exploits an agent's ability to traverse a filesystem-like structure, but usefulness still depends on how it represents organizational memories, practices, and experts — the points from which retrieval branches.
- Werry limits scope: the product is engineering-focused. Engineering integrations can help business users understand product behavior, but the absence of a Salesforce integration limited sales-domain coverage at the time.
- **Synthesis raises a harder permission question than retrieving one document,** because a summary may combine sources with different audiences. His starting point is compartmentalized synthesis: build knowledge inside boundaries such as a repository, then correlate it with material available to the intended audience.
- When synthesized information crosses boundaries, attach group identifiers and check them during retrieval. GraphRAG makes the risk easy to see because it builds summaries at multiple levels of a graph, and a higher-level summary can combine facts that originated under different access rules.
- Under the hood: a procedurally built knowledge graph (he compares it to a PageRank-like structure of entity relationships), vectorized data, LLM distillation of material such as PR history, and procedural tools for runtime fetching. Conflict handling happens in two stages — ingestion tags that relate and rank evidence, and a runtime judge given criteria for the conflicts that remain relevant.

### The social graph: mapping collaboration and expertise · 39:45
- An audience question about organization-specific meanings (for example, different definitions of revenue) leads Werry to the references delivered with an answer, to both the human and the agent. A user can reply with a correction or mark the answer unhelpful and explain why — partly manual, but repeated feedback creates signals.
- He reports hundreds of feedback items from teams of 20–30 people and hundreds more from teams of 100–200, without specifying a collection period, and notes that natural-language correction is an interaction people readily understand.
- The workshop turns to a smaller, inspectable component: the Social Graph Builder. Participants join a Slack workspace, post their GitHub identity for an invitation, and enter the GitHub organization (a prerequisite for the Unblocked link). The tool analyzes repository activity locally with no uploads; the public version stores the graph locally, while optional Anthropic labeling can send member names and code-area metadata.
- The first view is a reviewer–author relationship grid for Unblocked. From there, it infers teams; its history starts at the beginning of 2025, and Werry recommends a shorter first run because a long scan is slow. AI is optional and used to label teams; without it, labels come from the code areas members work on most. An experts view groups people by project area and path, and coverage reflects both active contributions and the presence of a strong contributor.
- Running the project is deliberately small: point Claude Code at the repo and ask it to read the `README`; specify the repo to analyze; bound the scan with a date range or PR limit (for example, the last 300 PRs); then ask the agent to bootstrap and run it.
- The graph's more important role is as a **retrieval pivot**. Knowing the expert for a code area points the engine toward that person's past work, role, Slack decisions, and PR comments — Werry calls the distillation step "bottling the expert." For a new employee in unfamiliar code, this gives three layers:

| Layer | Contribution |
| --- | --- |
| Semantic search | Related source material |
| Prebuilt memories | Distilled recurring knowledge |
| Expert learnings | Direction for further investigation |

- Contribution counts are only a first signal. Werry says the initial approach of comparing PR contribution counts with review counts produced inaccurate team clusters, so the workshop algorithms occupy a middle ground and the full product adds richer evidence: vectorized source code (people in nearby clusters are related by proximity), a further LLM layer that distills historical work and Slack conversations, and weighting against the procedural graph.
- Slack and Teams need different signals because messages lack the review relationships in source control. The engine can summarize a channel's subject, vectorize that summary, and associate it with frequent contributors — but activity alone is insufficient. His counterexample is a highly vocal junior engineer whose volume does not imply more expertise than a quieter person's occasional intervention. Relating discussion back to what PRs followed, and which merged, is what assesses impact.

### Corrections, benchmarks, freshness, and privacy · 1:20:42
- A consultant asks to see the actual SaaS product. Werry shows the dashboard for asking questions about code and the organization and describes an internal "source mark" engine that tracks code changes over time, including moves between files. He suggests a vendor demo or connecting the product to one's own organization, and mentions a full-product enterprise trial.
- On feedback: rather than merely adjusting weights or confidence scores, the system constructs **task memory**. Repeated corrections are distilled, and the expert graph weights the sources — a correction from an established expert receives more weight than one from a new engineer whose trustworthiness is not yet established.
- The stored representation differs from what the agent sees. Memories live in database tables and draw on multiple sources; at runtime the system hydrates them into an agent-facing form presented as files. Some material is **seed context** that must be supplied before the agent can reliably choose its next retrieval step, and expert context provides that initial direction.
- Asked about an official benchmark, Werry acknowledges internal benchmarks but no public evaluation for this organizational-context problem. He cites Boris Cherny on Claude Code evaluation and a colloquially named "Shitpost" benchmark to make the point that user sentiment matters. He reports a sentiment score around 60 on a scale from −100 to 100 (roughly 0.75–0.8 normalized), without specifying the measurement protocol.
- He roughly estimates context collection at about 90% of an agent task's time (no stated protocol). Input processing and time to first token were comparatively well optimized; output tokens were the major performance cost, so focused context should reduce unnecessary output loops.
- The benchmark task was implementing Anthropic's adaptive-thinking mode:

| Adaptive-thinking migration | Reported wall time | Human corrections |
| --- | --- | --- |
| Claude with vendor MCP servers | Roughly 2.5 hours (also stated as 2h 25m) | Four loops |
| Claude with Unblocked | 25 minutes | None |

- These are results from a single task comparison with no repeated-trial protocol; the mechanism is the avoided rework after an initially incorrect implementation. The same task used 21 million tokens without the context engine and 10 million with it, with avoided correction loops accounting for part of the reduction. He expects little benefit for a generic task such as centering a div.
- On freshness: many integrations update through webhooks, others use cron-based polling. Frequent updates are needed, but not full rebuilds — the social graph supports incremental updates, and a single change rarely transforms the expertise graph unless the organization is brand new. Best-practice distillation runs roughly weekly because practices change more slowly than activity.
- On privacy: discussions happen at the organizational level, with more friction in government and banking. He notes that Slack discussions can contain reasoning organizations consider more sensitive than source code itself. Unblocked offers an on-premises option, but he recommends cloud because integrations and patches update more readily; the example is a network-isolated bank where administrating the platform requires vendor staff inside the network or training the bank's own administrators.

## Resources

- [Social Graph Builder](https://github.com/unblocked/engineering-social-graph) — a Python CLI that maps GitHub review relationships, team clusters, and expertise
- "Social Graph Builder open-source release" — Unblocked's announcement explaining the collaboration-graph approach (update article)
- "Unblocked context for agents" — the MCP and CLI release announcement (update article)
- [Claude thinking controls](https://platform.claude.com/docs/en/build-with-claude/thinking-steering-and-cost) — current documentation for adaptive reasoning and effort settings
- [GraphRAG indexing](https://microsoft.github.io/graphrag/index/overview/) — Microsoft's overview of graph extraction and hierarchical summarization
- "Gemini 1.5 technical report" — Google's report on long-context retrieval and reasoning (related paper)
