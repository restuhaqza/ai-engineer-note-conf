# Code Mode: Let the Code do the Talking

| Field | Details |
| --- | --- |
| **Speaker** | Sunil Pai |
| **Affiliation** | Cloudflare |
| **Type** | Talk |
| **Date** | 2026-04-19 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=8txf05vVVl4) |
| **Tags** | `coding-and-developer-tools` `apis-mcp-and-protocols` `safety-and-governance` |

## Summary

Sunil Pai (who built PartyKit, now working on agents at Cloudflare) presents
"Code Mode." The idea is simple: instead of the model making a long back-and-forth
series of JSON tool calls, ask it to write a small JavaScript program that runs
against a typed API. The model composes the whole task in one execution.

The motivation is scale. Regular tool calling works with a few tools, but once
you connect Google, Jira, a wiki, and more, hundreds of tool definitions flood the
model's context. Cloudflare's API alone has roughly 2,600 endpoints. Pai's
colleague Matt Carey exposed all of it through just two tools, `search` and
`execute`, cutting the tool-description footprint from about 1.2 million tokens
to roughly 1,000.

The talk goes further than saving tokens. If code can run against the state that
already exists, an agent can act *inside* a running application instead of
building a separate one. A user could get a custom interface generated on the fly.
Pai frames the whole thing as a safety problem too: generated code should start
with no authority and only receive the capabilities it needs.

## Key Takeaways

- **Tool-calling does not scale.** Hundreds of tool definitions crowd the context
  before the agent does any useful work.
- **Code Mode replaces round trips with one program.** The model writes JavaScript
  against a typed API; loops, state, sequencing, and parallelization happen in
  code.
- **Two tools can cover a huge API.** `search` finds operations in the OpenAPI
  spec; `execute` runs code against them. Cloudflare reduced ~1.2M tokens to
  ~1,000.
- **The model can inhabit existing state.** In one demo it played Tic-Tac-Toe by
  reading and appending to a drawing canvas's stroke array, with no game code.
- **Capability-based security is the key idea.** Start with an environment that
  can only execute code, then grant APIs and network access explicitly.
- **Default to no outgoing fetches.** Only the exposed APIs should be reachable.
- **You need observability.** If an agent makes a huge trade, you must recover the
  exact program that ran.
- **Your next users are agents.** Design discovery, types, Markdown docs, and
  error messages for code-generating clients.

## Detailed Notes

### When tool calling becomes choreography · 01:31
- With a couple of tools and short runs, tool calling is fine.
- Add many services and you get hundreds of tool definitions filling the context.
- Completing a task becomes a chain of exchanges: call, return, ask what next.
  Composition gets awkward and every round trip adds latency.
- Pai's alternative: ask the model for code that runs against an exposed
  environment.
- Code brings typed APIs, syntax and type checking, and lots of representation in
  model training data.
- **Move composition into code:** one execution can loop, hold state, sequence
  dependent steps, and parallelize independent ones.

### An entire API behind search and execute · 03:01
- Cloudflare's API surface is about 2,600 endpoints. Exposed as individual tools,
  that is roughly 1.2 million tokens in the first call.
- Matt Carey's implementation exposes just two tools, both taking a string of code.
  - `search` runs against the full OpenAPI document to find operations.
  - `execute` runs against callable API functions to perform them.
- The OpenAPI document stays on the server; the search program picks what matters
  rather than inserting everything into context.
- This separates **discovering capabilities** from **using capabilities**.
- Reported result: down to about 1,000 tokens, around a 99.9% reduction in
  tool-surface overhead (not total task tokens).

### Compose the task, then watch execution · 04:21
- Example: a site is being DDoSed at 3 a.m. and the customer wants offending IPs
  found and blocked.
- Pai estimates that would take about eight MCP round trips. Composed JavaScript
  does it in one execution near the API.
- The live demo is smaller: list the account's Workers. Give it read-only access
  and approve the interaction.
- The run searches for a listing endpoint, generates code, and starts executing.
- It then hits JavaScript errors and appears to try pagination. Pai admits the
  stage run did not cleanly finish.
- The honest lesson: a smaller tool interface does not remove mistakes in
  generated programs or result handling.

### Give everyone access to a small script · 07:08
- Optimizing an MCP server is only the first application.
- A programmer facing "categorize and rename 200 photos" writes a throwaway
  script. A nontechnical user must ask for help or buy an app whose workflow
  almost fits.
- The application's fixed interface decides which combinations are easy.
- A model can turn "rename these files by date and location" into runnable code.
- This shifts the boundary between what programmers can customize and what
  everyone else has to accept from buttons and forms.

### Play the game already on the canvas · 09:03
- Kenton Varda, creator of Cloudflare Workers, built a personal coding
  environment and asked it for a tldraw-style drawing canvas.
- He drew a Tic-Tac-Toe grid and an X, then asked the model to play.
- The model started generating a whole Tic-Tac-Toe app. He stopped it and told it
  to inspect the existing state instead.
- The canvas state was just an array of strokes (each a list of points). The model
  recognized the board, saw the X, and appended a circle in the center.
- **There was no Tic-Tac-Toe code.** The change was a single new stroke in the same
  representation.
- Pai notes the game was lost, and that reasoning traces suggested the model let
  Kenton win (his interpretation).

### The harness: act inside the system · 11:10
- Pai says the model "started inhabiting the state machine": acting within the
  existing app rather than building another one.
- The useful abstraction is the **harness**: it generates code, provides a safe
  execution space, and exposes the capabilities the code may use.
- "Sandbox" is overloaded. This is not Cloudflare's container/VM Sandbox SDK.
- Start with an environment that can *only* execute code: no APIs, no fetches.
  Then grant capabilities explicitly.
- Cloudflare uses **Dynamic Workers** and V8 isolates: fast startup and years of
  security hardening. The host controls what is exposed.
- **Default: no outgoing fetches, only exposed APIs.** Network isolation must be
  configured; it is not automatic.
- You also need full observability to recover and explain what a program did.
  V8 is one choice; WebAssembly or a custom interpreter could work too.

### From one execution to personal software · 14:17
- A one-off API run can extend into long-running workflows that live for days or
  years and carry state.
- That enables interfaces tailored to each user.
- E-commerce is the illustration: as a store grows, its UI becomes generic.
  Personalization today often just changes a button color.
- The system already holds context: preferences, cart, orders, frustrations. That
  context can become actions in a generated interface.
  - Return shoes and find something similar under $100.
  - Investigate an overdue order and offer resolution steps.
- The harness could also run close to the user (even on a phone) to stitch
  together several services task by task.

### Design for the code-generating client · 17:06
- Your customers are still humans, but the immediate clients are agents
  generating code.
- Treat agent experience like developer experience:
  - **Discovery:** registries and search help agents find operations.
  - **Contracts:** types and syntax feedback make valid calls easier.
  - **Documentation:** Markdown makes behavior readable as text.
  - **Recovery:** errors should say what to do next.
- **Capability-based security** is the principle Pai most wants builders to keep.
  JavaScript is incidental; Python, WASM, or Lisp could play the same role.
- Recurring runtime needs: events, sandboxing, explicit capabilities,
  embeddability, fast startup, ephemeral execution.
- UI programmers may do well here, because they have spent time close to users.

## Resources

- [Code Mode: give agents an entire API in 1,000 tokens](https://blog.cloudflare.com/code-mode-mcp/)
- [Sandboxing AI agents, 100x faster (Dynamic Workers)](https://blog.cloudflare.com/dynamic-workers/)
- [Pi agent harness](https://github.com/earendil-works/pi)
