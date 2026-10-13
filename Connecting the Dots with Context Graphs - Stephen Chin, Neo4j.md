# Connecting the Dots with Context Graphs

| Field | Details |
| --- | --- |
| **Speaker** | Stephen Chin |
| **Affiliation** | Neo4j |
| **Type** | Talk |
| **Date** | 2026-05-16 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=eW_vxrjvERk) |
| **Tags** | `rag-context-and-search` `finance` `healthcare` |

## Summary

Stephen Chin, who leads developer relations at Neo4j, argues that agents making real business decisions need more than relevant documents. They need connected domain knowledge, lasting memory, and a record of why past decisions were made. He calls that combination a **context graph**.

His opening image is the red pill from The Matrix. Most enterprises keep knowledge scattered across Slack threads, customer messages, and separate systems, so an agent asked to review a pull request never sees the business context behind it. Connecting those sources — enterprise data, previous decision traces, and tool calls — gives the agent what he calls a system of reasoning. He notes that Gartner has added context graphs to its AI hype cycle.

Chin builds the idea in layers. A knowledge graph stores people, things, and companies as nodes joined by relationships (and can also carry embeddings for similarity search). Retrieval then grounds answers in those connections: in a healthcare example, a graph-grounded answer pulls in a patient's smoking history and past operation, details a similarity search missed. Persistent memory adds short-term, long-term, and reasoning traces so later work can build on earlier decisions.

He demonstrates two open-source projects: a podcast memory explorer and a financial-services app that traces loan decisions. The value he emphasizes is a path from an agent's answer back to the evidence behind it.

## Key Takeaways

- **Context graphs unify three things** — a knowledge graph, vector retrieval, and durable agent memory — so an agent can use relationships between facts.
- **Relevant documents are not enough.** Agents also need connected domain knowledge and the reasoning behind earlier decisions.
- **Knowledge graphs model facts explicitly.** Nodes are people, things, or companies; relationships (lives with, drives, owns) are first-class, not rebuilt through table joins.
- **Graphs and LLMs are complementary.** The model supplies language, reasoning, and creativity; the graph supplies organized context and enrichment.
- **Memory comes in three layers.** Short-term (the current run), long-term (knowledge across tasks and customers), and reasoning memory (recorded decision provenance).
- **Reasoning memory is not hidden chain-of-thought.** A useful trace records the evidence, actions, and rationale available to the app — supporting repeatability and compliance.
- **Similarity finds an entry point; relationships carry you onward.** Embeddings (like FastRP) locate a starting node, then traversal and community grouping (Louvain) follow connections.
- **The graph can write back.** Each interaction's output and trace can be stored, becoming evidence for later decisions.

## Detailed Notes

### When agents review work without its context · 00:40
- Chin opens with an audience poll: who has their pull requests reviewed by an agent? Most hands go up.
- He describes an uncomfortable position: engineers have powerful AI tools but limited control over what those tools know.
- His Matrix metaphor: the blue pill is knowledge fragmented across siloed systems; the red pill connects enterprise data, previous decision traces, and tool calls into a shared system of reasoning.
- If you ask agents to make critical business decisions with scattered information, they cannot give good answers.
- He cites growing interest: Gartner now lists context graphs on its AI hype cycle, and Foundation Capital framed them as a major investment opportunity.

### Represent the relationships explicitly · 03:18
- A knowledge graph represents people, things, and companies as nodes with properties, connected by relationships.
- His example: Dan lives with Anne and drives her car — living together, driving, and owning are distinct relationships, and the car is one shared node.
- The same connections can be written in **Cypher** (Neo4j's graph query language); a query can follow either the driver or the owner to the same car.
- Embeddings can be attached on top, enabling similarity search alongside explicit connections.
- Complementary roles: LLMs bring language, reasoning, and creativity; graphs bring organized knowledge, context, and enrichment.
- Storing relationships together makes them retrievable, visualizable (like a whiteboard diagram), and analyzable for hidden patterns.

### Retrieve the patient's plan, not advice about the disease · 04:37
- The retrieval example: "What was the care plan associated with Andrea Jenkins' emphysema?"
- A baseline LLM gives generic advice (prevent further lung damage) because it only knows general medical knowledge.
- Vector RAG (retrieval-augmented generation, where a system fetches relevant text before answering) adds patient details and suggests respiratory therapy and breathing exercises — still fairly generic.
- Graph-grounded retrieval pulls in the patient, diagnosis, and operation history, producing medication management, smoking-cessation counseling, and pulmonary rehabilitation.
- The difference is the connected background: a smoking history and a prior operation that a similarity search lost. He notes this is an illustrative comparison, not a clinical evaluation.

### Add memory across interactions and decisions · 06:09
- Grounding handles the current question; persistent memory preserves what has already happened so later work can build on it.
- **Short-term memory:** the conversation, the current agent pipeline, and in-progress state — persisting these keeps the run informed.
- **Long-term memory:** knowledge across longer tasks, procedures, and customer interactions; it depends on a domain model for processes and entities.
- **Reasoning memory:** recorded decision provenance explaining why a recommendation was made, and providing precedents for future calls.
- He is precise that a decision trace records evidence, actions, and rationale available to the app — not a model's hidden chain-of-thought. Retained provenance helps repeatability, compliance, and debugging.

### Use similarity to enter the graph, then follow connections · 08:13
- Graphs are a representation both software and people can work with: an LLM can generate Cypher or extract structure from documents, and a person can inspect nodes much like a whiteboard sketch.
- Relationships are first-class objects, not connections reconstructed through many table joins; Chin says multi-hop traversal is performant (no benchmark shown).
- **FastRP** produces graph embeddings; vector lookup can give a starting point in the graph; **Louvain** community grouping helps organize connected regions.
- The two mechanisms differ: similarity locates an entry point, while traversal follows explicit connections from it.
- The open-source **Neo4j Agent Memory** package combines short-term, long-term, and reasoning memory in one context graph, tied to explainable decisions and ACID-compliant storage (a database guarantee for reliable transactions). He invites contributions; the repo's current APIs may differ from the recording.

### Explore connected information in a podcast · 09:57
- Podcasts make a good test case: dense and heavily interconnected across people, organizations, topics, and places.
- Lenny's Podcast Memory Explorer loads a podcast corpus and applies the three memory layers to answer questions about it.
- The agent gets tools built on the Neo4j Agent Memory APIs; in the demo it returns locations mentioned in an episode, with coordinates, which could feed a map.
- Because the data lives in a graph, the app can aggregate all locations tied to an episode rather than returning only similarity matches — a holistic view that can be navigated and queried dynamically.

### Write decision context back into memory · 11:32
- Cross-domain business questions need not just facts but an understanding of why an earlier choice was made.
- A context graph differs from a traditional audit log by keeping decision context: entities and relationships link evidence that would otherwise be buried in Slack, email, or informal conversation.
- The architecture is a feedback loop: (1) retrieve context using knowledge graphs, vector search, and data-science algorithms; (2) work through the request using that domain knowledge and earlier traces; (3) record the new output and its trace back into memory; (4) reuse the accumulated history on later queries.
- The output of one interaction becomes evidence available to the next.

### Connect financial records to decision provenance · 12:58
- The financial-services demo gives the graph a concrete domain model: people and organizations as entities; decisions, transactions, and approvals as events; and applied policies, risk factors, and employee reasoning as context.
- Architecture: a support-ticket system, a CRM, and internal business data are exposed through 10 MCP tools. (MCP is a standard way for agents to call tools.)
- Claude agents orchestrate the work, OpenAI embeddings supply vectors, and Neo4j stores the context — so orchestration is separate from embedding generation. The app keeps a domain graph and a reasoning graph, surfaced through a simple Next.js UI.
- The demo asks whether Jessica Norris should be approved. It queries her history (bank account, related margin trades) and shows the model-issued Cypher queries and the graph being traversed and populated, making the information inspectable.
- The connected record reveals a prior rejection — exactly the kind of earlier decision that disappears when systems stay separate.
- The model recommends against the loan and lists reasons, risk factors, earlier decisions, and fraud-detection patterns. Chin stresses this is a recommendation shown to a human user, not a demonstrated denial — the user can inspect the context before deciding.
- For developers, the value is a path from an agent's answer back to the information used to produce it. Chin frames that grounded information as the practical business payoff of context graphs.

### Try the memory layers yourself · 16:10
- Chin points to Neo4j's free education program and a GraphAcademy course on context graphs.
- The course provisions a free Aura (hosted Neo4j) instance behind the scenes, so learners get a graph database to experiment with before touching a production or enterprise system.

## Resources

- Foundation Capital, "AI's trillion-dollar opportunity: Context graphs" — https://foundationcapital.com/ideas/context-graphs-ais-trillion-dollar-opportunity
- Neo4j Agent Memory — https://github.com/neo4j-labs/agent-memory
- Lenny's Podcast Memory Explorer — https://github.com/neo4j-labs/agent-memory/blob/main/examples/lennys-memory/README.md
- FastRP (Fast Random Projection) documentation — https://neo4j.com/docs/graph-data-science/current/machine-learning/node-embeddings/fastrp/
- GraphAcademy: "Context Graphs: Agent Memory with Neo4j" — https://graphacademy.neo4j.com/courses/genai-context-graphs