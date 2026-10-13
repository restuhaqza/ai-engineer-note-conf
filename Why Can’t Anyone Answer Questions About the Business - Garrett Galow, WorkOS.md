# Why Can’t Anyone Answer Questions About the Business?

| Field | Details |
| --- | --- |
| **Speaker** | Garrett Galow |
| **Affiliation** | WorkOS |
| **Type** | Expo session |
| **Date** | 2026-06-11 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=iUWwcG-C8OU) |
| **Tags** | `agent-engineering` `rag-context-and-search` `enterprise` |

## Summary

Garrett Galow leads product at WorkOS, whose enterprise features sit behind logins
at companies like Cursor, Anthropic, and OpenAI. His talk is not about that
product. It is about an internal tool, **Studio**, built so colleagues can answer
business questions themselves instead of filing a request for someone with SQL
skills and data access.

The problem is familiar. Dashboards and Retool apps answer questions someone
already anticipated. The moment a request needs one extra field or a deeper
breakdown, it becomes a ticket: explain the question, wait, check the result, ask
for one more layer, and end with a one-off answer in Slack. Studio makes that
investigation self-service and, when a question recurs, turns the answer into a
reusable tool.

The core idea is a **widget**: sandboxed code (UI, API calls, and a query) that
runs without the model. The agent's job is to investigate once and generate that
code; after that, refreshing runs the code and re-queries the data. The model only
returns when someone asks for a change.

Galow walks through a content-attribution question and a support question, then
explains what made Studio reliable: sequencing, layering, and validation. He also
answers audience questions on schema cleanup, trust in generated queries, access
control, and Opus cost.

## Key Takeaways

- **Dashboards answer anticipated questions; everyone else waits in a queue.** Studio makes that investigation self-service.
- **A widget is code, not a chat.** Once generated, refreshing a widget re-runs JavaScript and the query — no LLM call.
- **Business meaning must be supplied.** How a customer is represented and which joins connect tables come from a guidance layer, not the raw schema.
- **Guide the agent in steps.** Check connections, ask for missing context, follow a checklist, then inject tool guidance only when a tool is invoked.
- **Validate before shipping.** Run each Snowflake query and confirm it returns rows before hard-coding it into a widget.
- **Filter semantic rules matter.** Excluding deleted entities and inactive records answers the real question; omitting them quietly produces a wrong number.
- **No RAG database was needed.** Studio invokes tools directly with added context rather than retrieving from a vector store.
- **WorkOS spends on quality.** Opus outperforms cheaper alternatives enough that the team will not trade away answer quality.

## Detailed Notes

### A question becomes a queue · 00:25
- WorkOS supplies enterprise platform features, including password and enterprise identity-provider login.
- Engineers, go-to-market, and support teams all need to understand how customers use the product.
- Retool apps and dashboards are rigid: someone asks for an extra field or a new question the dashboard does not cover.
- The requester explains the question and waits; someone with SQL or data access answers, and often gets asked for one more layer.
- The result is a one-off answer shared in Slack, ready to be repeated next time.
- **Studio** is an internal workspace where employees answer questions and build their own apps and dashboards.

### Which content leads to new teams? · 02:44
- Galow starts with a prepared prompt about content attribution: which content leads to the most new team creations.
- WorkOS calls its customers "teams" and buys podcast and Google ads that bring visitors to its site, blog, and docs.
- Studio checks which internal resources it can use — it has connections to Linear, Notion, and Snowflake — and investigates Snowflake.
- The request path: a Studio dashboard or a Slack bot, an API that parses the request, and a **LangGraph** agent using **Opus**.
- An integration proxy connects the agent to Snowflake, Linear, and Notion; a guidance layer explains how to use them.
- That guidance carries business meaning the schema alone may not: how a customer is represented and how tables join across a sprawling estate.
- **Convex** persists state across sessions so the workspace remembers past work.

### Turn the answer into a tool · 05:23
- The results list homepage, pricing page, blog posts, changelogs, and docs — a view of content associated with signups, not proven causation.
- Galow asks Studio to build a table that shows the data across time slices for weekly syncs.
- Studio calls the result a **widget**: bundled sandboxed code for the UI, the API interactions, and the query.
- A pre-built version shows content and team signups across time spans and reruns the query on use; it can go fullscreen for sharing.
- The newly generated version differs slightly — it adds content-category filters, so someone can narrow to blog posts and study which ones drive signups.
- The workflow has moved from a one-off answer to a reusable interface for deciding where to focus.

### Why was this user blocked? · 07:34
- WorkOS Radar protects against bots and bad actors, creating a recurring support question: why was this user blocked?
- Previously that meant navigating dashboards or sharing SQL queries.
- Galow opens a Studio widget with the queries or API connections already attached and searches his own email; it runs a real database query.
- The conversation history shows the iteration: a first attempt with an issue, a "keep going" request, and a fix for a visual bug in the type column.
- The result lists his Cursor logins and whether each attempt was blocked, including a deliberate self-block in a test environment.
- Support also uses Studio directly in Slack to pull a customer's sessions; recurring questions become shared widgets.

### Sequence, layer, and validate · 09:55
- **Sequencing** controls how the agent approaches each request: check tools are connected, decide if the question has enough context (ask if not), follow a checklist to pick tools, then inject tool guidance at invocation time.
- For Snowflake the guidance is substantial — how teams connect to environments and the resources they use.
- Injecting guidance at invocation time avoids filling the context window with instructions for every tool up front.
- **Layering** combines the base Studio prompt, defaults, and organization rules, and preserves a tool's context when someone edits it.
- Studio also tells the model to distrust possibly outdated knowledge of WorkOS products and consult primary sources like current docs.
- **Validation** runs each generated Snowflake query and checks it returns data before embedding it; valid SQL that returns zero rows leaves the question unanswered.
- WorkOS runs evals in both staging and production and treats them the same, so development matches what teammates use. Galow does not detail the eval design.

### Teach the joins — and the business rules · 12:59
- Asked whether WorkOS cleaned up its Snowflake estate first, Galow says no.
- One customer-to-user relationship needs four joins; new employees used to copy a join block, while Studio receives that knowledge in its guidance.
- Descriptive column names help the model read a schema, but explicit relationship instructions still matter.
- Radar shows why: its data separates login attempts from detections, and the guidance explains how those tables join.
- There is no RAG database in this implementation — it invokes tools directly with extra context.
- One questioner asks what happens when a wrong generated query becomes a shared widget everyone treats as truth; Galow answers with trust-but-verify, not a formal audit process.
- Semantic rules a plausible query can omit: exclude deleted entities, filter to active records, and remember that a count grouped by customer ID still needs those filters.
- He reports that embedding these rules removed many problems, and that remaining errors tend to be easier to spot. He gives no measured accuracy.

### The model builds the widget; JavaScript runs it · 15:48
- A widget can combine data from multiple connected tools into one interface, and WorkOS is adding more connections.
- The generated artifact is JavaScript that calls the underlying services through the tools.
- **Refreshing a widget does not ask the LLM to repeat its investigation** — it executes the code and re-queries the data.
- The model returns only when someone requests a change, such as adding a column.
- User inputs become query arguments handled by that code, so searching an email does not need the model to reinterpret it each time.
- Example: a small `createLoginSearch` helper with injected `queryAttempts` and `renderRows` functions; both search and refresh use the same query path with no model inference.
- Whether the query expresses the *correct* business definition remains a separate question from runtime reliability.

### Access boundaries and the cost of answers · 17:03
- At the time of the talk, Studio integrations are per user: each employee connects their own Snowflake, Linear, and Notion accounts.
- Galow calls that inconvenient and notes a broader problem: someone may need selected Salesforce data without a Salesforce account.
- **WorkOS Pipes** powers the third-party integrations under the hood.
- WorkOS is building organization connectors so one person configures a connection and sets default access rules — for example, read-only in Linear by default, with edit or admin access by role.
- That permission layer is still work in progress during the talk.
- On cost: generated widgets run code, so each refresh avoids another LLM charge.
- For the questions and generation that do use the model, WorkOS is willing to pay for quality; Galow says Opus performs well enough that switching to cheaper models would sacrifice too much. He gives no version, benchmark, or cost figures.

## Resources

- [LangGraph Graph API overview](https://docs.langchain.com/oss/python/langgraph/graph-api) — how the agent is composed.
- [Convex Overview](https://docs.convex.dev/understanding/overview) — the database used to persist state across sessions.
- [WorkOS Pipes](https://workos.com/pipes) — managed third-party connections and OAuth token handling.
- [WorkOS Radar](https://workos.com/radar) — the bot and account-abuse product behind the support example.
