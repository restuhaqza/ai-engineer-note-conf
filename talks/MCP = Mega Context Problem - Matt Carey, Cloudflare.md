# MCP = Mega Context Problem

| Field | Details |
| --- | --- |
| **Speaker** | Matt Carey |
| **Affiliation** | Cloudflare |
| **Type** | Talk |
| **Date** | 2026-04-25 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=YBYUvGOuotE) |
| **Tags** | `developer-workflows-and-testing` `apis-mcp-and-protocols` `coding-and-developer-tools` |

## Summary

Matt Carey, who works on MCP and agents at Cloudflare, asks a simple question:
how do you give an AI agent access to an *entire* API? He starts from function
calling, where the model asks the app to run a function and gets the result back.
MCP (Model Context Protocol — a standard way for agents to reach external tools)
then let services publish tools once so many agents could reuse them. That works
fine when there are eight tools.

It breaks down when you try to cover a whole API. Every tool's description and
parameter schema takes up room in the agent's context (the limited text it can
consider at once). Cloudflare's OpenAPI spec came to roughly 2.3 million tokens,
and turning endpoints into plain tools still needed about 1.1 million. The company
then split its API into product-specific MCP servers, but that pushed the choice
onto users and left gaps — 16 servers covering only about 2,500 to 2,600 endpoints.

Carey's fix is **progressive discovery**: load capabilities only when they are
needed. He walks through three ways to do that — using a CLI, searching a tool
catalog, and letting the model write code against typed SDKs (what Cloudflare calls
Code Mode). The last option is his favorite, but it means running untrusted code,
so the rest of the talk is about creating a safe place to run it.

Using Cloudflare's dynamic Workers (small code sandboxes called V8 isolates), he
shows how a host can decide exactly what a generated program may touch — filesystem
secrets, the network, or neither. He closes with predictions: APIs must survive
heavy generated traffic, clients will save reusable mini-scripts, and MCP will
likely become a simple toggle built into web frameworks.

## Key Takeaways

- **Naive tooling doesn't scale.** Exposing one tool per API endpoint can blow past any model's context window before the agent does useful work.
- **Progressive discovery is the answer.** Capabilities should be revealed on demand, not dumped into context up front.
- **Three discovery routes:** a CLI's command tree, tool search over a catalog, and letting the model write code against types.
- **Code Mode keeps the spec as the source of truth.** TypeScript types are generated from OpenAPI, and the model composes operations into a program.
- **Running generated code is the hard part.** An agent that can act can also leak secrets, loop forever, or mine crypto.
- **The sandbox is programmable.** A dynamic Worker can be given or denied secrets and network access by flipping host-side settings.
- **Coverage is not permission.** Representing 2,000+ operations does not mean the current user may run them; that's what authorization is for.
- **Rate limiting must survive generated workloads.** Many sandboxes can loop API calls at once, so the service needs protection, not just the sandbox.

## Detailed Notes

### Making every API a tool for agents · 00:32
- **Function calling** is the base layer: the model emits a call, the application runs it, and the result returns to the model so it can answer.
- Before remote MCP, each agent bundled its own integrations; if one developer built Gmail tools, the next had to rebuild them.
- Remote MCP offered a shared surface: a provider implements standardized tools once and many agents consume them.
- With only about eight tools, this arrangement looked manageable — the problems start as you add more.

### Full coverage blows up the context window · 02:39
- Tool descriptions and parameter schemas consume context *before* any useful work happens.
- Cloudflare's **OpenAPI spec** (a machine-readable description of its API) was about **2.3 million tokens**; naive tools still needed roughly **1.1 million**.
- Loading one tool per endpoint was therefore impractical, even for the biggest models.
- Their first fix split the API into **product-specific MCP servers**, which cut context per integration.
- But that shifted the selection burden to users and left incomplete coverage — e.g. six tools for a suite with around 30 endpoints.
- Cloudflare reached **16 MCP servers** while its API had about 2,500–2,600 endpoints; more servers did not fix the mismatch.

### Discover capability only when needed · 04:46
- **Progressive discovery** means not eager-loading every capability into the model's context.
- MCP is just a protocol — it does not require dumping all tools in at once.
- **CLI route:** the agent inspects commands step by step. In the Wrangler demo it runs `wrangler`, then `wrangler d1`, then `--help`, then the actual list.
- Each step reveals enough to choose the next; Carey says this mostly works, and cites OpenClaw as a popular CLI example.
- The catch: the agent needs **shell access**, which is not always available.
- **Tool search:** keyword matching pulls candidate tools into context. In Carey's illustrative example, ~2,100 tokens enter context but only ~500 belong to the tool actually used.
- Search narrows the catalog well but not always to exactly one operation.

### Let the model write against typed code · 07:27
- The third approach, **Code Mode**, lets the agent write code against the API instead of picking one call at a time.
- **TypeScript types** are a concise way to describe inputs and outputs that a model can reason about; they are generated from the OpenAPI spec.
- Demo: a request to list Workers becomes code against a typed SDK, and running it returns the account's Workers.
- A workflow demo chains operations: deploy a Hello World Worker, protect it with Cloudflare Access, and apply a policy that lets only the speaker in.
- The model supplies the composition; API authors maintain the spec.
- **The OpenAPI specification stays the source of truth** — no separately maintained set of agent-specific wrappers.

### Running generated code needs an execution boundary · 09:06
- The approach did not catch on immediately; clients (the agents consuming MCP) were slow to implement it.
- The blocker was not generating code but taking responsibility for *running* it on a user's behalf.
- Unreviewed programs could read filesystem secrets, exfiltrate them over the network, loop forever, exhaust resources, or mine crypto.
- Giving an agent credentials makes it useful — and makes the execution boundary consequential.
- Syntactically valid code says nothing about what that code should be *allowed* to do.
- Existing compromises: **restricted languages** (JSON-based DSLs), **VM sandboxes**, and **code review**.

### A programmable sandbox with explicit authority · 11:15
- A **dynamic Worker** can be created from a source string and run inside a **V8 isolate** — a lightweight sandbox.
- In the demo the generated program runs on the backend, not in the browser.
- With **Node compatibility on**, printing `process.env` shows no secrets and reveals a Cloudflare global; turning compatibility off makes `process.env` unavailable and the probe errors.
- An attempt to call an external API is first rejected; flipping a server-side flag permits it.
- A more selective function can allow only particular domains — the approach used by Cloudflare MCP.
- The mechanism is **explicit authority**: the host decides which environment and network capabilities the program receives.

### Coverage and permission are separate · 13:01
- An embedded MCP client connects to the Cloudflare MCP server and asks for authorization through a login screen.
- After authorizing, Carey gets **read-only** access to his Cloudflare infrastructure via the API surface (account IDs shown are not secrets).
- The integration can span deploying Workers, adding Access, inspecting DNS, and prospectively sending email — over 2,000 endpoints.
- But the live session is read-only: **coverage determines what can be represented; authorization determines what this user may execute.**
- Those are two different problems, solved in two different places.

### Compact plans, saved scripts, and MCP as middleware · 14:13
- **Code is a compact plan.** One tool that accepts a program can carry more freedom than a single function call, including iteration and composition.
- Carey expects more isolated runtimes to emerge; he names **workerd**, **Deno**, and **Pydantic Monty**.
- He likens running user programs to submitting punch cards in the 1950s: users supply instructions, someone else's infrastructure runs them.
- **Services must survive the program:** a generated program can loop API calls, and many sandboxes can run at once, so APIs need strong rate limiting.
- On the client side, generating code lets users **save reusable mini-scripts** — e.g. a daily scrape that the agent repairs when it breaks.
- At scale (100 agents per person), permanent sandboxes per agent are costly; Carey expects more **stateless agent loops**.
- Final prediction: MCP becomes **middleware** — a framework flag like `mcp: true` — with a lightweight SDK small enough to fit into app bundles.
- Closing pointers: install the `agents` package, and read the Code Mode posts; the ~1,000-token figure describes the tool surface, not total task context.

## Resources

- [Code Mode: give agents an entire API in 1,000 tokens](https://blog.cloudflare.com/code-mode-mcp/) — companion blog post.
- [Code Mode: the better way to use MCP](https://blog.cloudflare.com/code-mode/) — Cloudflare's original write-up on generating TypeScript against an API.
- [Cloudflare MCP server](https://github.com/cloudflare/mcp) — source and connection instructions.
- [Cloudflare Agents SDK](https://github.com/cloudflare/agents) — SDK, examples, and docs (`npm i agents`); `@cloudflare/codemode` is a separate package.
- Dynamic Workers documentation — runtime-created Workers and their bindings.
- [Pydantic Monty](https://github.com/pydantic/monty) — a Python interpreter for untrusted code.
