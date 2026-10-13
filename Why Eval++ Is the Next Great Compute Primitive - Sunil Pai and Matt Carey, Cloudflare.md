# Why Eval++ Is the Next Great Compute Primitive

| Field | Details |
| --- | --- |
| **Speaker** | Sunil Pai and Matt Carey |
| **Affiliation** | Cloudflare |
| **Type** | Talk |
| **Date** | 2026-06-08 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=SKDJo2CopRs) |
| **Tags** | `apis-mcp-and-protocols` `rag-context-and-search` `safety-and-governance` `developer-workflows-and-testing` |

## Summary

Cloudflare engineers Sunil Pai and Matt Carey walk through the building blocks behind their agent platform: Durable Objects for addressable, persistent state; Dynamic Workers for running untrusted, model-generated code safely; and the Agents SDK that connects clients to a durable agent identity. The session starts from a small broken example — a serverless hit counter that forgets its count — and grows into a general design for stateful, resumable, multiplayer agents.

Their larger claim is that executing generated code can be fast, safe, and cheap enough to revisit "eval," something programmers were long told to avoid. They call Dynamic Workers "Eval++" because generated code runs inside an isolated Worker where the host grants only specific capabilities.

Along the way they preview Code Mode, where a model writes code against an API instead of calling many individual tools, and discuss practical applications: resumable streaming, collaborative conversations, plugin isolation for a CMS, and an in-progress Cloudflare coding harness.

## Key Takeaways

- **Durable Objects are addressable state.** Requests and WebSockets for a given ID reach the same logical object — Cloudflare calls the model "stateful serverless."
- **The hit counter breaks because serverless isolates have no memory.** A counter held in the handler works locally, then vanishes once deployed.
- **MCP pushed the team toward stateful infrastructure.** Early remote MCP servers needed a persistent client–server connection.
- **Dynamic Workers run a code string** inside an isolated Worker, with no separate deployment per generated program.
- **Start from restriction, then add capabilities.** Generated JavaScript has no network or APIs until the host explicitly grants them; explicit grants beat an outbound blocklist.
- **Eval++ is a pitch, not a proven result.** The speakers invite people to try it rather than presenting comparisons.
- **Resumable streaming falls out of durable state.** Reconnect to the same agent and continue the stream, across tabs and devices.
- **Not everything has to be Cloudflare.** Heavier work can delegate to containers, browsers, or other providers.

## Detailed Notes

### Why the hit counter breaks · 00:44
- Pai and Carey started building seriously on **Durable Objects** a bit over a year before this talk; Pai jokes it is a badly named but powerful technology.
- The demo: a request handler increments a counter. It works locally, then fails on deploy.
- The reason is **no state**: an isolate spins up, handles the request, and disappears, so most requests see a fresh counter. Under traffic, an isolate might get reused.
- The usual repair is a database. Durable Objects change the **unit of computation**: everything addressed to a given ID reaches the same object.
- Cloudflare calls this **stateful serverless** — serverless scaling with a clear owner for state. Stable identity does not mean one in-memory instance lives forever.
- Latency example: roughly **15 ms in London**, compared with about 16.67 ms per frame at 60 FPS (the speaker gives no measurement setup).
- **tldraw** is the showcase: draw on a shared board across several phones and the changes appear together.
- Agents need the same traits: an address, persistence, background work without an incoming request, hibernation, and connections to other services.

### From objects to agents and MCP servers · 04:24
- The **Agents SDK** wraps that execution model in an application interface: import from `agents`, extend the class, schedule startup work, and expose callables; React hooks and plain JavaScript clients connect the frontend.
- Scheduling example, deliberately mundane: every Friday at 9 PM, compile Git history, a wiki, and Notion notes into an update for a manager — with a few spelling mistakes added.
- Model calls alone do not give coordinated tool execution, resumability, or cross-tab sync. Cloudflare built a backend for **Vercel's AI SDK** to cover those (higher-level chat now lives in `@cloudflare/ai-chat`).
- **MCP** (a standard way for agents to call tools) was another fit: early remote servers needed a stateful connection, which Durable Objects handle naturally.
- Carey names **PayPal, Sentry, Twilio, Linear, and Intercom** as early MCP servers hosted this way. The original spec also supported other transports.
- Working on those servers led the team toward Code Mode.

### Run a code string in a separate Worker · 06:54
- A normal **Worker** is a deployed serverless function. A **Dynamic Worker** takes a string of code — from a customer, a user, or an LLM — and runs it in its own isolate, with no separate deployment.
- Reactions split: one person insisted enterprises would never run code they had not inspected; an attendee from Lockheed Martin loved exactly that capability. The execution boundary is the crux.
- A Dynamic Worker is an **isolate, not a VM** — no full filesystem or operating environment. Carey says they can spin up billions on demand, without presenting a capacity test.
- Practical use: skip intermediate formats. Instead of form input → JSON → DSL → code, let the user or model supply code directly.
- The sandbox **starts from restriction**: only JavaScript runs, with no fetch or application APIs until the host grants them.
- The host can expose a few APIs, allow requests only to `github.com`, or narrow it to a path like `github.com/xyz`. A policy can itself be code.
- **Explicit capabilities beat a blocklist alone.** Give code the operations it needs and keep environment variables and credentials out of reach.
- Denied network access is a configured policy, not an immutable property — the host decides what to grant. Executing a program is not the same as giving it the application's authority.

### Eval++ and generating the interface directly · 09:45
- **Code Mode** preview: the model writes code, then the platform runs it. Carey mentions exposing around 2,600 Cloudflare API endpoints through roughly 1,000 tokens of tool definitions (the endpoint count is his spoken figure).
- For decades programmers were told never to use `eval`, and Workers do not offer it. Pai calls Dynamic Workers **Eval++** because they add a separate isolation boundary and controlled capabilities.
- His claim is that a long-avoided branch of programming is worth revisiting when execution is fast, secure, and cheap — offered as a proposition, not a measured result.
- On generative UI: `json-render`-style systems have a model emit JSON that a renderer turns into an interface. Why not generate HTML or React directly?
- The missing piece is usually a **safe place to run untrusted rendering code**. Claude Artifacts is the browser-side example; moving that to a server raises the trust question.
- Two approaches: JSON-driven UI relies on an approved renderer, while generated HTML or React needs isolated execution with controlled capabilities.

### Refresh the page without losing the agent · 13:20
- The Agents SDK is an **execution environment**, not a library: it works with LangChain, Vercel's AI SDK, or another agent framework.
- Demo: ask an LLM for a long story, then refresh mid-generation. In a request-bound design you would have to coordinate storage, replication, and sticky sessions yourself.
- With the agent on a Durable Object, the browser **reconnects to the same identity**. If the stream is still running, the backend supplies the beginning and keeps delivering new bytes.
- The connection can be replaced without the browser owning the generation process.
- Cross-tab and cross-device sync come along for free. Pai argues AI should be a multiplayer experience — imagine sharing a chat link so two people work in the same conversation.
- Cloudflare handles the distributed coordination; the SDK team builds the JavaScript on top, so developers focus on the collaboration itself.

### Keep the coding loop in the cloud · 15:20
- An audience member asks about a cloud Claude Code interface and the awkward VM start, close, and resume lifecycle.
- Carey separates the **agent loop from its clients**: run the loop in the Agents SDK, then connect a terminal, chat interface, phone, iOS app, or web app to the same persistent backend.
- Every client sees synchronized, resumable state.
- Pai confirms Cloudflare is building such a **harness on Workers and Durable Objects** and hopes to ship it soon — work in progress at the recording.
- A light execution core need not do everything: delegate heavier work via **Containers and the Sandbox SDK**, connect **browsers** when needed, or use tools like **Daytona, Browser Use, or Lightpanda**.
- Workers and the Agents SDK become the persistent coordination point around these environments.

### Let the agent extend itself · 17:35
- Storage: **SQLite inside the Durable Object**, plus **R2** for larger data. At the recording, wiring them together is partly manual; Pai expects that to get easier.
- An OpenClaw-style system breaks down into: a heartbeat, a virtual filesystem, connections to services, secret control, and extensions.
- **Extensions are generated code run in a sandbox** — Pai says he built extensions that morning. Running the **pi** coding agent on Workers is a goal discussed with Mario, not a delivered integration.
- Languages: **JavaScript and Python are first-class** (Python rough edges still being polished); everything else targets **WebAssembly**. **Zig** is in production for undisclosed work, with smaller Wasm bundles than Go or Rust (no sizes given).
- For true native execution, spin up a container.
- Pai notes that if an LLM writes the code, language preference matters less — but application code still needs preparation, handled by **Worker Bundler** (runs inside a Worker, pulls npm dependencies, strips TypeScript and JSX, and produces runnable output, using Cloudflare's dependency cache).

### A virtual filesystem, isolated plugins, and the economics · 20:40
- `@cloudflare/shell` provides a **virtual filesystem** layered over Durable Object SQLite, with R2 for bigger files. It is not a full Linux or POSIX shell, and current docs mark the APIs experimental.
- **EmDash**, a CMS launched the previous week, runs on Workers and Durable Objects and runs **plugins through Dynamic Workers**. WordPress plugin incidents motivate isolating extension code instead of granting it broad authority.
- Portability and plugin isolation are separate claims: EmDash can deploy off Cloudflare, but support for the Dynamic Worker part on other platforms is still being developed.
- **vinext**, a Next.js implementation running on Vite, is also mentioned. More announcements were expected during Agents Week the following week.
- On cost: Pai credits infrastructure decisions from about a decade ago — placing hardware near ISPs instead of building massive data centers, buying bandwidth in bulk, and interconnection agreements that keep cross-boundary traffic cheap.
- In his account, free accounts come from that business structure rather than pure marketing. He cites a **$5/month** entry point and anecdotes of multimillion-dollar SaaS businesses on the platform, without usage details.

## Resources

- [Cloudflare Agents documentation](https://developers.cloudflare.com/agents/) — durable agent state, connections, and scheduling.
- [Cloudflare Agents repository](https://github.com/cloudflare/agents) — SDK source, examples, and issue tracker.
- [Dynamic Workers: architecture and launch](https://blog.cloudflare.com/dynamic-workers/) — runtime-loaded isolates with host-controlled capabilities.
- An entire API in 1,000 tokens — Code Mode applied to Cloudflare's API (related article).
- [@cloudflare/shell](https://github.com/cloudflare/agents/tree/main/packages/shell) — sandboxed JavaScript with a virtual filesystem over SQLite and R2.
- [Worker Bundler](https://github.com/cloudflare/agents/tree/main/packages/worker-bundler) — building and bundling app code at runtime.
- [Introducing EmDash](https://blog.cloudflare.com/emdash-wordpress/) — the CMS with Dynamic Worker-based plugins.