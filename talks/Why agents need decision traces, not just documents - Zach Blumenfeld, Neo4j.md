# Why agents need decision traces, not just documents

| Field | Details |
| --- | --- |
| **Speaker** | Zach Blumenfeld |
| **Affiliation** | Neo4j |
| **Type** | Talk |
| **Date** | 2026-05-29 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=B9h9ovW5H9U) |
| **Tags** | `architecture` `observability-and-reliability` `apis-mcp-and-protocols` `rag-context-and-search` `industry-applications` |

## Summary

Retrieval-augmented generation (RAG, fetching text to answer questions) gives an agent facts, but often not enough to make a defensible decision. Zach Blumenfeld of Neo4j argues that a financial agent asked to approve or reject a credit request needs something extra: the trail of how similar decisions were made before.

He calls this a **context graph**. It is a knowledge graph that links facts (customers, transactions, policies) to **decision traces** (why earlier decisions went the way they did) and reusable precedents. Neo4j frames its product as a knowledge layer with a graph database at the core.

Blumenfeld demonstrates a Neo4j-backed agent stack, then shows an open-source generator that scaffolds a working context-graph app in one command. He admits the hardest open problem is deciding which traces deserve to become experience in the first place.

## Key Takeaways

- **A knowledge base answers questions; a context graph supports decisions.** Facts alone yield a risk score, not a clear reject/accept with reasons.
- **Connect three kinds of data:** entities (things that exist), events (decisions, approvals), and context (policies and reasoning).
- **Decision traces link into causal chains,** so an agent can follow how earlier steps built on each other.
- **Search uses two kinds of similarity:** semantic (related meaning) and structural (similar shape of how a decision was made).
- **Graph embeddings** turn connected records into vectors, so structurally similar past decisions become findable by vector search.
- **Neo4j Agent Memory** combines three memories: short-term (session), long-term (resolved entities), and reasoning (decision traces).
- **`create-context-graph`** scaffolds a full app (frontend, backend, graph) from one command, with a choice of domains and agent frameworks.
- **Open problem:** the system makes past decisions retrievable, but nothing yet decides which of them are *good* enough to reuse.

## Detailed Notes

### What does an agent need to make a decision? · 00:16
- Blumenfeld frames the problem with a **financial analyst agent** reviewing a credit-limit increase.
- A knowledge base (RAG or graph retrieval) supplies customer info, transactions, and policies — enough to assign a risk score.
- A **context graph** adds past decision traces and precedents, so the agent can recommend accept or reject *and explain why*.
- The added value is history about **how earlier decisions were made**, not more facts about the current customer.
- A graph separates three components: entities, events, and context.
- The reasoning can come from an AI system's memory or from human employees who decided before.

### Following the financial agent's retrieval · 04:04
- The demo uses generated data simulating a CRM and support system.
- **Claude** runs the agent, **OpenAI embeddings** power vector retrieval, **Neo4j** stores connected data and vectors, and **Next.js** is the frontend.
- The recorded walkthrough exists because the venue's internet was unreliable.
- Asking a question triggers a **series of tool calls**; the UI shows graph data, then decision traces, then a reject recommendation beside the graph.
- A `Find Precedents` tool looks past the customer's own records into the structure of the graph.
- Retrieval combines **semantic** similarity (related content) with **structural** similarity (how prior decisions were made).

### Searching for a similar decision structure · 07:23
- A **text-vector index** finds content that is semantically related to a phrase like "fraud rejection".
- A **graph embedding** is a vector derived from connected nodes, not only from words.
- Encoding graph structure into vectors makes *structurally* similar decision records findable by vector similarity.
- The companion implementation uses **FastRP** node embeddings; you do not have to embed each whole trace as one document.
- This surfaces past decisions and tickets as precedents, including patterns that are hard to recover when relationships are buried inside documents.
- Neo4j's **Graph Data Science (GDS)** generates the graph embeddings; the talk does not specify score weights for semantic vs structural similarity.

### Starting with a generated application · 08:34
- **Create Context Graph** is a generator in the spirit of `create-react-app` or `create-next-app`.
- The entry point runs as a single `uvx create-context-graph` command.
- You supply an app name, a domain, an agent framework, and often demo data.
- Scaffolding is only step one: you still install dependencies, seed the graph, and start the app (a few minutes; he switches to a running instance onstage).
- Once running, retrieval uses **Cypher**, Neo4j's graph query language.
- Scaffold choices are independent: **agent framework** (Pydantic AI demoed; also OpenAI, LangGraph, CrewAI, Strands, Google ADK), **domain** (22 built-in templates, e.g. healthcare and financial services), and **capabilities** such as graph-native queries, decision-trace retrieval, MCP server generation, and multi-turn chat.

### From conversation history to connected memory · 13:08
- **Neo4j Agent Memory** is the API underneath, spanning three memories that the speaker treats as all necessary.
- **Short-term** memory is conversation history and session context.
- **Long-term** memory holds extracted entities, resolved across repeated mentions over time.
- **Reasoning** memory holds the decision traces inside the graph.
- A staged text pipeline turns raw text into a graph: start with **spaCy**, add **GLiNER** for entity extraction, fall back to an LLM, then a separate merge/dedupe/enrich strategy.
- Finding a mention is not the same as resolving it to a known entity — the later stages make short-term text into long-term entities, and the schema connects conversations to entities to reasoning traces.

### Exploring the system — and representing time · 14:56
- The financial-demo blog links to a hosted application for exploring concepts before a local setup; Agent Memory integrates with Microsoft Agent Framework and Google ADK.
- First audience question: can a causal chain carry time?
- **Yes for storing time:** events can have timestamps and be linked by relationships like `caused` or `next`.
- **Unclear for using time:** Blumenfeld does not know whether recent events get more weight yet; he calls it a possible future development.
- His framing point: representing time is a different thing from using recency to rank precedents.

### Preparing a schema for existing information · 16:53
- The **ontology** (the graph's node and relationship types) is prepared beforehand and guides both extraction and mapping.
- Many predefined domains use a **POLE**-style model (Person, Object, Location, Event); Agent Memory's documented POLE+O adds Organization.
- For a large existing collection, the suggested path is the generator's **custom-domain** route: describe the intended ontology and the data, and produce a schema.
- Then it branches by input type: **unstructured text** → guide entity extraction with the schema; **structured CSV/tables** → map fields into graph writes with Cypher.
- He cautions the project is still rough, and that the workflow starts from a description of the data — it does not promise automatic discovery of a full schema from any archive.

### Who decides which traces become experience? · 18:41
- Final audience question reaches past retrieval: as the graph grows, how do you tell good decisions from bad?
- Sub-questions: should people give thumbs-up/thumbs-down? Can the system tell if a new trace is novel or already represented?
- Blumenfeld says these mechanisms are **still being worked out**.
- In the first demo, a user can ask the agent to store decisions from a prior conversation — he does not think it stores them unless prompted.
- Writing new decision traces in Create Context Graph is also still under development; quality or sentiment scores are possibilities he raises, not shipped features.
- The unresolved step is deciding what to keep, how to judge it, and when new decisions should change the memory future recommendations rely on.

## Resources

- [AI's trillion-dollar opportunity: Context graphs](https://foundationcapital.com/ideas/context-graphs-ais-trillion-dollar-opportunity) — Foundation Capital essay on decision traces and reusable precedents.
- [Hands on with context graphs and Neo4j](https://neo4j.com/blog/agentic-ai/hands-on-with-context-graphs-and-neo4j/) — the financial-agent walkthrough and implementation path.
- [Context Graph Demo](https://context-graph-demo.vercel.app/) — hosted app for exploring financial decision context.
- [Create Context Graph](https://github.com/neo4j-labs/create-context-graph) — open-source generator for domain-specific graph applications.
- [Neo4j Agent Memory](https://github.com/neo4j-labs/agent-memory) — graph-backed conversation, entity, and reasoning memory.
