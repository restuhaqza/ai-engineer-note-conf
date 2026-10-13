# Stop babysitting your agents: context for mergeable code

| Field | Details |
| --- | --- |
| **Speaker** | Brandon Waselnuk |
| **Affiliation** | Unblocked |
| **Type** | Expo session |
| **Date** | 2026-05-26 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=BiG2ssibKGc) |
| **Tags** | `coding-and-developer-tools` `apis-mcp-and-protocols` `rag-context-and-search` `leadership` |

## Summary

Brandon Waselnuk of Unblocked starts from a familiar feeling: typing `claude` in
a terminal summons a brilliant software engineer who knows nothing about your
organization. It does not know which services already exist, which decisions
still apply, or why the team rejected an obvious approach last month. Filling
that gap is the "babysitting" he wants to eliminate.

His answer is a **context engine** — a system that gathers and interprets
organizational knowledge for coding agents. It retrieves from systems of record,
adds runtime signals, reasons across sources, and returns a small,
token-efficient response. It also understands *who* is asking, using a **social
graph** of people, repositories and working relationships.

Waselnuk walks through three assumptions that failed while building it: that
granting access would produce understanding, that conflicting sources could stay
hidden, and that a correct answer was safe to cache. Each turned out wrong. The
session ends with a live demo: told to research first, the agent returns a
research packet, then sends explore agents to the right code, producing a plan
for a Zendesk integration that a senior engineer largely accepted.

## Key Takeaways

- **Access is not understanding.** MCP connections give paths to information,
  not the reasoning to know where to look or when to stop.
- **Naive RAG stops too early.** "Satisfaction of search": the agent finds one
  plausible pattern and treats it as the answer.
- **The social graph is the pivot.** Knowing who is asking narrows an
  underspecified request.
- **Conflicts must be surfaced, not hidden** — return the code *and* the
  contradicting message with its attribution.
- **Permissions belong inside retrieval.** The requester's OAuth identity should
  govern what the engine returns.
- **Don't cache finished answers.** A correct answer describes one moment in
  time.

## Detailed Notes

### A brilliant engineer on their first day · 00:16
- Before agents, engineers built their own context. On day one you had skills but
  little local knowledge.
- Team interactions, rejected pull requests and experience gradually taught you
  what works *here*.
- Becoming effective meant asking good questions, gathering accurate context and
  discarding what does not help.
- With agents, the human often filters and then translates the result into
  instructions for the agent.
- The goal: code that feels like it came from someone who has been on the team
  for years.

### From curated files to a runtime context engine · 02:07
- Waselnuk credits an adoption ladder to Basim Eldes:
  - Start with autocomplete and agentic IDEs, where a human initiates and
    guides the work.
  - Move to **curated context**: `CLAUDE.md`, `AGENTS.md` and repositories of
    corporate knowledge agents can read.
  - Reach a **context engine**.
- Curated files make instructions reusable but need maintenance and lack live
  system data.
- A context engine pulls static sources across SaaS apps and systems of record,
  adds runtime signals, reasons across them, then returns a small response.
- Getting good context up front makes every later agent choice better: for a new
  integration it can explain the factory pattern and conventions before the agent
  explores code.
- The framing, attributed to Karpathy: the gap is **context, not intelligence**.

### Three myths: access does not create understanding · 04:49
- **Myth 1 — naive RAG over docs is context.** It fails through *satisfaction of
  search*: after finding one anomaly that seems to explain the symptoms, a
  radiologist stops looking. The agent equivalent is finding one plausible
  pattern and treating it as the answer.
- **Myth 2 — enough MCP connections is enough.** Connectors are pipes; they do
  not provide understanding or reasoning across the data.
- **Myth 3 — a large context window solves it.** A million-token window did not
  make the agent reason over it, because there are no entities or relationships;
  he extends the argument to a hypothetical hundred-million-token window.
- The resulting gap: the agent sees whether code compiles but misses
  architectural constraints. Grepping burns tokens, and the session's
  understanding disappears when the terminal closes.
- Comparison: the same prompt with direct MCP access to every needed SaaS tool
  produced output that compiled and passed checks, yet a senior engineer judged
  it wrong and potentially system-breaking. **Code that passes checks can still
  violate the system's design.**

### Social graphs, conflict resolution, and permissions · 08:19
- The engine must understand who is asking; for a Zendesk request the useful
  starting points are the engineer's repositories, PR history and collaborators.
- A **social graph** provides those connections. At customers with twenty
  thousand members, the same request can mean different work depending on who
  asks.
- Relationships also help interpret conflicting evidence:
  - Suppose code on `main` implements one behavior, but a Slack message has the
    CTO saying it was implemented wrong.
  - Returning only the code hides the correction; returning only the message
    hides what actually runs. The engine should return the code plus the
    correction and its attribution.
- The CTO's role is an authority signal, not proof a message is infallible;
  Waselnuk says truth-across-sources is not fully solved.
- **Permissions are a design requirement, not automatic.** Unblocked carries the
  OAuth identity model through MCP so the requester's access governs the answer
  — for example, returning a user's own private chats but never someone else's.
- The engine ingests sources and serves several interfaces: humans ask in Slack;
  agents ask the same way through a tool. Background and cloud agents especially
  need machine-answerable questions, or an overnight run returns a permission
  request instead of a pull request.
- He groups the requirements into six capabilities: unified system context,
  targeted retrieval, conflict resolution, data governance, personalized
  relevance and token optimization.

### What the naive vs. context-engine run showed · 11:47
- Waselnuk notes that Claude generated the comparison charts and numbers; the
  useful evidence is the engineering behavior.
- Both runs got the same prompt; one had all the needed direct MCP connections,
  the other used only the context engine.
- Reported differences:

  | Review concern | Direct MCP access | Context-engine run |
  | --- | --- | --- |
  | Existing fallback behavior | Missed the Bedrock fallback | No corresponding failure reported |
  | Existing callers | Broke custom callers | No corresponding failure reported |
  | Senior review | Judged unmergeable | Accepted subject to a nitpick |

- He associates the first result with prototype-like agent work: mocks and
  locally plausible changes that fail to preserve how the surrounding system
  works.
- This remains a speaker-reported comparison, and he does not treat the charts
  as a general benchmark.

### Three lessons from building the engine · 12:38
- **Access would become understanding.** Unblocked expected the model to handle
  interpretation once it could reach the data; over years of development that
  did not resolve the problem. Context has to be brought into the harness
  deliberately.
- **Conflicts could stay hidden.** Early versions let the agent choose when
  sources disagreed. Hiding the disagreement removed information needed to
  judge the choice, so they learned to surface and address conflicts.
- **A good answer was safe to cache.** Answer caching looked attractive for
  latency after getting positive feedback. But a correct answer describes the
  system at one point in time; return yesterday's answer to today's identical
  question and it may be wrong. He recommends against this caching approach,
  while allowing that some questions do have stable answers.

### One engine, many workflows — and the expert graph · 13:40
- The same question-answering capability supports people outside engineering.
- Unblocked's engine sits in customers' ask-engineering channels: it detects a
  question, scores confidence and replies automatically — the same evidence an
  agent would need.
- Other uses: ticket enrichment, triage, incident management and coding help.
- Teams customize by forking a **cookbook of skills** and adding their standard
  operating procedures, then let custom agents consult the same engine.
- Closing demonstration: the workshop's **social-graph component** (open-sourced
  later) builds a graph of the engineering organization.
  - People are nodes, working relationships are edges; node size reflects how
    much someone ships.
  - Selecting a person shows collaborators, whose code they review and where
    they work; labels use an Anthropic API key.
  - When a query is underspecified, identifying a person and following their
    collaboration edges narrows the codebases to inspect — offering a useful
    search scope rather than a demonstrated diagnosis.
  - It also exposes authorship and review heat maps and peer tables.

### Research first, then explore, then review · 17:12
- In the final demo Waselnuk asks how to build a first-class Zendesk
  integration, telling the agent to use Unblocked's MCP:

  ```bash
  claude 'Use the Unblocked MCP to research how to add a new first-class Zendesk integration. Then inspect the relevant code and write an implementation plan.'
  ```

- The key instruction is to research the organization's patterns *before*
  planning the change.
- The agent picks the historical `research_task` tool, builds a query from the
  MCP interface, and requests high reasoning effort (later `context_research`).
- When the research packet returns, explore agents now know where to look.
- The plan covers provider registration, the factory pattern, library modules
  and the client; he calls it largely correct but expects more refinement.
- The MCP stays available while the agent executes, so his sequence is:
  1. **Plan with context** — research patterns and constraints first.
  2. **Execute with access to further research** — let the agent return to the
     engine as new questions arise.
  3. **Review with context again** — assess the change against organizational
     knowledge alongside the code review itself.

## Resources

- [Social Graph Builder](https://github.com/unblocked/engineering-social-graph)
- [How Unblocked's Social Graph Builder works](https://getunblocked.com/blog/oss-social-graph-builder/)
- [Using Unblocked's context research tool](https://docs.getunblocked.com/changelog)
- [Configuring Data Shield](https://docs.getunblocked.com/team-settings/data-shield)
