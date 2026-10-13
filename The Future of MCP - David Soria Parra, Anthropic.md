# The Future of MCP

| Field | Details |
| --- | --- |
| **Speaker** | David Soria Parra |
| **Affiliation** | Anthropic |
| **Type** | Talk |
| **Date** | 2026-04-19 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=v3Fr2JR47KA) |
| **Tags** | `apis-mcp-and-protocols` `agent-engineering` |

## Summary

David Soria Parra, who works on MCP at Anthropic, gives a status update and
roadmap talk. MCP (Model Context Protocol, a standard way for agents to connect
to tools and data) has grown fast: he reports roughly 110 million monthly
downloads, reached in about half the time it took React. It started as a small
local spec and now includes remote connections, authorization, and new primitives.

His central argument is that 2026 is about putting agents into production, and
that production agents need *several* kinds of connectivity, not one. Coding
agents have it easy (local, verifiable, a developer nearby). A financial analyst
or marketer needs an agent that can connect many SaaS apps and a shared drive, so
connectivity becomes the product.

He argues for a stack that combines Skills (reusable domain knowledge), MCP
(protocol-level interoperability), and CLIs or Computer Use. No single method is
the universal answer; the right choice depends on the environment and the task.

## Key Takeaways

- **MCP apps ship an interface and tools together.** A server can deliver a UI
  that compatible clients (Claude, ChatGPT, VS Code, Cursor) render, plus tools
  the model can call.
- **There is no single connectivity solution.** Good agents mix Skills, MCP, and
  CLIs or Computer Use.
- **Progressive Discovery moves the burden to the client.** Load tools only when
  the model needs them instead of dumping every definition into context.
- **Programmatic tool calling (Code Mode) reduces latency.** Let the model write
  a script instead of routing every step through inference.
- **Stop converting REST APIs one-to-one.** Design MCP servers around how a person
  or agent would actually complete a task.
- **Structured Output helps composition.** Type information about return values
  makes it easier to chain operations.
- **The protocol still needs work.** Stateless transport, async tasks, richer
  SDKs, enterprise auth, server discovery, and Skills over MCP are all on the
  roadmap.
- **Client support is uneven.** Extensions create a compatibility tradeoff; not
  every client can render every capability.

## Detailed Notes

### From tool protocol to shared application infrastructure · 00:15
- An **MCP application** is an agent shipping its own interface over an MCP
  server, not a plugin, an SDK, or something rendered client-side.
- Take one server and drop it into Claude, ChatGPT, VS Code, or Cursor and it
  works, because both sides share the *semantics* to render it.
- The same server can ship an app *and* tools: a human uses the interface, a model
  uses the tools.
- A year and a half ago, MCP was a small local-only spec with a few tools. It now
  has remote connections, centralized authorization, elicitation, and tasks.
- **~110 million monthly downloads.** This includes dependencies pulled by
  OpenAI's Agents SDK, Google's ADK, LangChain, and many frameworks.
- Public examples include WhatsApp, Blender, Linear, Slack, and Notion servers.
  Most servers, though, connect private company systems behind closed doors.

### Production agents require multiple kinds of connectivity · 03:59
- 2024 was demos, 2025 was coding agents. Coding agents are the easy case: local,
  verifiable, a compiler to check, and a developer nearby to fix things.
- 2026 is the start of general knowledge-work agents for analysts, marketers, and
  similar roles.
- Their key need is **connectivity**: connecting roughly five SaaS apps and a
  shared drive.
- If someone claims one method solves all connectivity, they are likely wrong.
  The answer is "it depends."
- The three main options:
  - **Skills** — capture specific, reusable domain knowledge in a simple file.
  - **MCP** — the connective tissue for platform independence, rich semantics,
    authorization, governance, and features like MCP apps.
  - **CLIs / Computer Use** — great for local agents that compose in bash and
    where a sandbox and code execution can be assumed.
- CLIs are strong when tools like GitHub and Git are already in the model's
  pre-training. MCP wins when you need a UI, long-running tasks, resources,
  enterprise controls, or no sandbox.

### Make the client responsible for progressive discovery · 07:12
- Do not blame MCP for context bloat. A protocol just moves information; the
  **client or harness** decides how it reaches the model.
- Early implementations loaded every tool up front, so context grew with every
  integration.
- The fix is **Progressive Discovery**: give the model a mechanism like Tool
  Search and load tools only when they become relevant.
- This replaces an always-present catalog with on-demand discovery driven by the
  model's current needs.
- Available in Anthropic's product and API, usable via competing APIs, and
  buildable directly.
- The talk shows a large drop in tool-context usage in Claude Code, but no exact
  number is given. The lesson: scalable connectivity needs deliberate client
  design.

### Compose tools through code and design servers for agents · 09:36
- A second client-side pattern is **Programmatic tool calling**, also called
  **Code Mode**.
- Calling a tool, reading the result, calling another, and repeating puts
  inference inside every orchestration step. That adds latency.
- Better: have the model write a script that coordinates the operations.
- Provide an execution environment such as a **V8 isolate**, **Monty**, or a Lua
  interpreter so generated code can compose, filter, and continue on its own.
- MCP's **Structured Output** describes return values and gives type information
  that makes composition easier. Without it, a cheaper model can extract the
  expected structure.
- **Stop converting REST endpoints one-to-one into MCP tools.** Design around how
  a task is actually accomplished.
- Execution can live on the client or the server; the Cloudflare MCP server is an
  example of offering an execution environment instead of isolated tools.
  Benefits cited: lower token usage, lower latency, better composition.

### Strengthen transport, identity, discovery, and extensions · 13:49
- **Stateless Transport Protocol (Google-backed):** the current HTTP streaming
  approach is hard for hyperscalers to scale. A stateless design would let MCP
  servers deploy like ordinary REST services on Cloud Run or Kubernetes.
- **Async tasks:** an experimental foundation for agent-to-agent communication.
  Few clients support it yet.
- **SDKs:** TypeScript SDK v2 and Python SDK v2, based on lessons learned. He
  credits **FastMCP** as a better Python experience than the one his team shipped.
- **Cross App Access:** log in once with a company identity provider (Google,
  Okta) and use MCP servers without signing in again.
- **Server Discovery:** define how agents, browsers, and crawlers find MCP servers
  via well-known URLs instead of parsing websites.
- **Extensions** trade compatibility: MCP apps need web-capable clients, since a
  CLI struggles to render HTML. **Skills over MCP** would let server authors ship
  and continuously update domain knowledge. A rudimentary version already works
  with a "load skills" tool.
- The direction is not to replace CLIs, Computer Use, or Skills, but to make them
  interoperate. He invites feedback; MCP now runs as an open community with a
  foundation, Discord, and issues.
