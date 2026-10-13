# Context Graphs for Explainable, Decision-Aware AI Agents

| Field | Details |
| --- | --- |
| **Speaker** | Andreas Kollegger, Zaid Zaim |
| **Affiliation** | Neo4j |
| **Type** | Talk |
| **Date** | 2026-05-28 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=abvQEhvRI_c) |
| **Tags** | `agent-engineering` `rag-context-and-search` |

## Summary

Zaid Zaim and Andreas Kollegger of Neo4j explain **context graphs** — graphs that connect an agent to organizational knowledge, memory, and the rules behind decisions. The framing is that language, reasoning, and creativity are not enough; an agent also needs the facts and context of the organization it works in.

They describe three kinds of memory a graph can hold: short-term (the current conversation), long-term (organizations, people, and things), and reasoning (the context for deciding what to do). A graph represents this as nodes and relationships, which makes the connections between data explicit — and the most useful context often lives in those connections.

The second half turns to decision-making. Using an OpenClaw-style agent told to keep the fridge stocked with Red Bull, Kollegger shows how a simple instruction collides with the real world when rent is due. You can patch prompts forever, but the deeper problem is how an agent should decide when it hits a case its instructions did not anticipate.

His answer is a workflow modeled on human decision practice: frame the decision, consult precedent and business rules, weigh risk against value, propose rather than act, let an authorized agent decide or escalate, then record the reasoning as precedent. The framework is general, but the criteria for each step are domain-specific.

## Key Takeaways

- **Context graphs extend knowledge graphs** with memory and the policies — the "why" — behind a decision, not just the knowledge an agent needs.
- **Three memory layers:** short-term (conversation), long-term (organizations, people, things), and reasoning (what justifies a task decision).
- **Graphs make connections first-class,** which matters because useful context often sits in relationships, not isolated facts.
- **Agentic GraphRAG** retrieves context by translating natural language into Cypher (the graph query language) and traversing the graph.
- **Autonomous agents hit cases their instructions did not cover.** Patching prompts one by one does not solve the general problem of deciding well.
- **Frame each decision** with its objective, the causality that led to it, and its environment — purchasing and medicine carry very different stakes.
- **Precedent is evidence, not permission.** Prior decisions guide consistency, but current rules can justify overruling them.
- **Analysis produces a proposal, not an action.** A separate agent checks authority and certainty, then acts or escalates.

## Detailed Notes

### Give the agent the knowledge it is missing · 00:38
- Zaid Zaim opens with the challenge of using knowledge graphs to fill the gap between what an agent can reason about and what it actually knows.
- Agents are strong at language, reasoning, and creativity, but those do not supply an organization's facts and tools.
- He frames three kinds of memory:
  - **Short-term** — captures the conversation with the user, for immediate context.
  - **Long-term** — captures organizations, people, and things, giving a persistent picture.
  - **Reasoning** — captures the context needed to decide what to do for a task.
- The representation is a **graph**: nodes (entities) connected by **relationships**, which makes deep connections between different data explicit.

### Context graphs add the why · 02:55
- Context graphs stay inside context engineering, but broaden what the agent receives: alongside knowledge, tools, and content, they supply **rules and policies**.
- Knowing what an agent can do is only half the problem; a decision also needs a reason it should do it.
- Every organization is already a network — finance, products, suppliers, people — and those relationships matter once the question shifts from retrieving a fact to deciding something (for example, whether someone is eligible for a certain amount of money).
- The memory graph ties it together: short-term memory holds conversations and state, long-term memory gives the broad picture, and reasoning connects a proposed task to the policies that justify it.

### Retrieve graph context with GraphRAG · 05:56
- The agentic **GraphRAG** (retrieval-augmented generation over a graph) flow:
  1. The agent checks whether the requested knowledge exists in its knowledge source.
  2. If it is missing, it queries the graph database.
  3. A tool such as **Text to Cypher** translates natural language into Cypher (Neo4j's query language).
  4. Graph traversal retrieves relevant content for the answer.
- The goal is a better-grounded, more reliable answer. He describes the mechanism and its aim; no measured reliability gain is claimed.

### Restock the fridge or protect the rent? · 06:57
- Andreas Kollegger makes the problem concrete with OpenClaw-style agents: give an agent a credit card, an Amazon account, and the goal of keeping the fridge stocked with Red Bull.
- Ordering more satisfies the instruction — until rent is due and the purchase would leave too little money.
- You can add a prompt rule for that conflict, then another for the next one; the meta-problem is how the agent should decide when a new conflict appears.
- It gets worse at scale: one agent becomes several specialists that must collaborate and stay aware of each other.
- His proposal is to make familiar human decision practices explicit as an agent workflow — implementable in LangGraph or as skills. The core work is turning implicit judgment into instructions an agent can follow.

### Frame the decision, then weigh precedent and rules · 09:24
- The workflow starts with **framing** — the moment an agent hits uncertainty and enters a deliberate decision subprocess instead of just picking the next action.
- Three parts to the frame:
  - **Objective** — what this decision should resolve.
  - **Causality** — what prior reasoning or events made it relevant now (he suggests leading with this).
  - **Environment** — the setting the decision operates in.
- The environment changes the stakes: ordering Red Bull and a medical decision involving a life cannot share assumptions about acceptable error. Humans infer this from experience; the agent needs it stated.
- Local framing feeds global context: look first at comparable past decisions for consistency, then bring in the business's hard and soft rules (formal process documents or informal notes in Slack and Google Docs).
- Key caution: **precedent is evidence, not automatic permission.** An earlier decision may still be right, or a changed rule may justify overruling it. Give the agent both before it analyzes alternatives.

### Assess risk before optimizing value · 11:32
- With the frame and global guidance in place, the agent does risk–value analysis — and Kollegger argues risk deserves most of the attention, with more scrutiny for serious decisions.
- First risk check: **reference class validation** — establish what actually matters to the affected parties rather than assuming. (A reference class is the group whose circumstances and needs define what is important here.)
- His example: a drug that is right for 99% of a population might be fatal for the other 1%, so a broadly appropriate recommendation cannot settle an individual case until you know which group the person is in. The proportions are illustrative, not clinical.
- Two more questions: **reversibility** (can the action be undone?) and **cost of error** (what happens if it fails?). A recoverable mistake allows a different process from one whose consequences cannot be recovered.
- The value side also needs an explicit objective: maximize availability, preserve the budget, or protect savings? Maximize a benefit or minimize a cost? Without that direction, the agent substitutes general knowledge for the user's actual goals.

### Propose, then authorize · 13:47
- **The output of analysis is a proposal, not an action.** In Kollegger's preferred design, a focused agent produces alternatives with pros and cons, then hands them off.
- A second agent decides whether it can act, checking its **authority** and **certainty**:
  - Authority and sufficient certainty → rank the options and act.
  - Insufficient certainty → escalate for judgment.
  - Insufficient authority → escalate to an authorized actor (a human or a higher-privilege agent).
- Oversight is a subprocess: declining to act locally is a valid branch, letting someone with the needed judgment or authority make the call.
- He favors compartmentalized, highly focused agents working together over a single agent doing everything.

### Record the outcome as future precedent · 14:51
- The workflow ends in a decision or a deferral. Deferral — not enough information or certainty to act — is a legitimate state.
- When a decision is made, record the process: what was considered, what was not, the decision itself, and the actions taken. These connected records go into the graph for tracing and accountability.
- For the Red Bull case, a Cypher record can distinguish a **considered option** from an executed action. In the example the decision is marked `deferred`, the option is `proposed`, and `actionsTaken` is empty — so a later reader cannot mistake deliberation for a purchase.
- Future agents retrieve that record as precedent. This is the self-learning loop: experience flows into the next decision process through graph memory. No model-training step is involved — the improvement comes from reusing recorded decisions and their context.
- The framework is general, but each step needs domain-specific definitions of relevant context, acceptable risk, applicable policy, and authority. It can be implemented in LangGraph, Google ADK, or skills.
- They invite people to contribute new domains to their growing catalog. The work lies in making each domain's implicit judgment explicit enough for an agent to use.

## Resources

- [Neo4j GraphRAG retrieval guide](https://neo4j.com/docs/) — Python examples for graph retrieval, including natural-language-to-Cypher.
- [OpenClaw documentation](https://docs.openclaw.ai/) — setup and usage for the autonomous agents referenced in the example.
- [Implementing human oversight with LangGraph interrupts](https://docs.langchain.com/oss/python/langgraph/interrupts) — pausing an agent workflow for approval.
- [Google Agent Development Kit](https://google.github.io/adk-docs/) — open-source toolkit for building agents.
- "Why AI agents need three types of memory" — Jim Webber on knowledge, conversations, and reasoning memory.
- [Neo4j GraphAcademy](https://graphacademy.neo4j.com/) — free hands-on courses on Neo4j, Cypher, knowledge graphs, and graph-based apps.
