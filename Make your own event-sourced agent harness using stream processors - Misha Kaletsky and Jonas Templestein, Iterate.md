# Make your own event-sourced agent harness using stream processors

| Field | Details |
| --- | --- |
| **Speaker** | Misha Kaletsky, Jonas Templestein |
| **Affiliation** | Iterate |
| **Type** | Workshop |
| **Date** | 2026-05-14 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=vi-2nasppAg) |
| **Tags** | `infrastructure-and-deployment` `agent-engineering` `developer-workflows-and-testing` `coding-and-developer-tools` |

## Summary

Jonas Templestein and Misha Kaletsky (Iterate) present an experimental way to build agent harnesses: make everything an event in an append-only log, and let small programs called stream processors derive state from that log and occasionally act on it. The starting problem is subtle — an agent already keeps a history, yet some side effects show up only in a trace, which makes debugging harder than it should be.

Their primitive is a hosted event store (events.iterate.com) that behaves like a filesystem of stream paths. You append events with a type and optional payload; the server assigns an offset and a timestamp. You can subscribe for updates, and a stream processor reduces new events into state and runs side effects in a separate hook. One persisted stream can feed many derived views, and a reducer is just a synchronous function, so processors can be reused and can run on different machines and in different languages.

The highlight demo: appending a JavaScript snippet to a stream installs behavior in it. Jonas posts a "dynamic worker configured" event containing a reducer and an after-append hook, then sends "ping" and gets "pong" from a stream that started with no behavior. The workshop was improvised and only partly worked live — ping/pong and the code-as-event demo succeeded, while the full LLM agent build was abandoned after tooling problems and an API quota error. The underlying ideas still came across.

## Key Takeaways

- Event sourcing here means making every occurrence an event, so the stream is the place to reconstruct what happened and decide what's next.
- The primitive is an append-only, hosted event store with filesystem-like paths; the server assigns increasing offsets (a definite position in the stream).
- A processor reduces events into state with a synchronous, side-effect-free function, then performs side effects in a separate after-append hook.
- Separating reduce from side effects matters for recovery: replay rebuilds knowledge, then the effect hook acts on the final state instead of replaying every past action.
- Reducers are reusable: one persisted stream can feed many derived views (the UI, a circuit breaker), and processors can run locally, as hosted services, or as cloud "dynamic workers."
- You can deploy behavior by appending code: a configuration event carrying JavaScript gave a stream ping/pong behavior.
- External plugins can contribute proactively through the event stream — a route the presenter sees as broader than MCP for hooking into the agent loop.
- There is no third-party "before" hook; plugins join an eventually consistent loop with a bounded wait (for example, up to 200 ms) so a slow check can't block the agent.
- The public demo has no authentication, and making computation reachable raises the open problem of who is allowed to trigger it.

## Detailed Notes

### What happened outside the agent's log? · 00:16
- The premise: agents already keep a log, yet some side effects appear only in an OpenTelemetry (OTel — a standard for traces) trace. The fix is to make every occurrence an event.
- Jonas wants extensibility: useful additions that both people and agents can write, and that compose without all running in one process on one machine.
- An agent should get a URL as soon as it exists, so a Slack webhook or a web-form submission is just ordinary HTTP input.
- Distribution follows: one processor on a server, another on a laptop, one in Rust and one in TypeScript, coordinating through events rather than shared memory.
- That freedom admits race conditions and feedback loops (two plugins replying forever), so those failure modes become part of the design.
- Fair warning: the workshop deployment has no authentication, so anyone can inspect streams; don't put secrets in payloads, and rotate exposed credentials.

### Append Hello World, then watch it arrive · 05:18
- The primitive is a hosted event store at events.iterate.com; its web UI shows raw events as YAML under hierarchical paths, like files.
- Paths start with "/" and URL-encoded slashes are tolerated. A project slug (also passable as an HTTP header) selects a separate namespace.
- An appended event returns an envelope: a required `type`, an optional `payload`, the stream path from the URL, a server-assigned `offset` starting at 1, and a creation time.
- The server is the serialization point: it hands out increasing offsets so a processor can point to a definite position instead of guessing order from contents.
- A second terminal can watch the path over server-sent events: `curl -N` avoids buffering, and `?live=true` keeps the connection open for later appends.
- The API is inspired by Durable Streams but does not implement that spec exactly. Piping through `sed` and `jq .` makes raw streams easier to read.
- If a responder were appending replies, the transport would already support the response loop an agent will later automate.

### Errors and controls are events too · 10:34
- The ingestion interface is intentionally tolerant so third-party webhooks are easy to accept; Jonas's premise is that an LLM can usually interpret a raw webhook without a custom connector.
- When a submitted object lacks the required `type`, the server records an invalid-event error in the stream — the error is itself an event.
- Server-generated types use documentation URLs as identifiers; callers can still use plain strings.
- An idempotency key prevents the same submission from appending twice.
- A circuit breaker pauses a stream at roughly more than 100 events per second; a pause event can also be added explicitly (it required a `reason`).
- While paused, further writes are rejected — and that rejection does not append an error, because errors would let a runaway writer keep growing a paused log. An explicit resume event restarts processing.
- Other configuration-by-event behaviors: a JSONata transformer that rewrites selected events, scheduling (a heartbeat every five seconds, or a future time, cancellable), and subscriptions (pull via SSE, or push to a server or Slack endpoint).

### From subscription to processor: reducers and effects · 15:57
- Existing harnesses are already event-driven; the proposed difference is a smaller API boundary where even a few letters of a streaming response are ordinary events.
- The hosted service is storage, closer to a database than to a model's response pipe; Jonas compares the programmable feel to Convex.
- The wire interface is language-independent (with docs and an OpenAPI spec); TypeScript is just the workshop's client choice.
- Quick start: `pnpm install`, call `createEventsClient` with the base URL and a path prefix, subscribe with `live: true`, ignore non-`ping` events, and `client.append` a `pong` for each relevant input.
- A processor reduces events into state with a synchronous `reduce(state, event)` function and performs side effects in `afterAppend`.
- Recovery is the reason for the split: after a laptop sleeps and 100 events pile up, replaying the reducer shouldn't fire 100 stale LLM requests. The runtime catches up to the log's end, then the effect hook decides what the reconstructed state requires.
- The live TypeScript run was shaky (a syntax error and editor trouble); participants confirmed ping/pong worked, and the more complex processor failed and was replaced by a code walkthrough.

### Derive the LLM request from state · 38:30
- Feature request: when someone adds agent input, make an LLM request. A UI shortcut creates an `agent-input-added` event with a string payload.
- OpenAI's `responses.create` needs a model, instructions, and input history, so those become the processor's initial state fields.
- The walkthrough initializes empty history, a system prompt, and GPT-5.4; a model-change event could later update the model through a one-line reducer branch.
- A Zod schema describes the string input event; output returned by OpenAI (even a small streaming fragment) is stored as an event, so the log keeps more detail than the finished conversation view needs.
- A small reducer turns an input string into a Responses API history item with `role: "user"` and the string in `content`; making the request stays a separate side effect.
- Response handling was unfinished in the live demo: streaming produces many events then an end event, but the completion branch and its exact event identifier were not settled.
- The full agent build was abandoned (an API quota error); the ping/pong baseline kept working.

### One persisted stream, many derived views · 46:50
- An audience question separates two ideas: every streaming chunk is persisted as an event, while a client can materialize its own derived state.
- Reducers can run in many places and build whatever state each consumer needs; persisting raw facts doesn't require every view to become a persisted event.
- Concrete example from production: the circuit breaker processor starts with `paused: false` and a null reason, keeps the last 100 event timestamps in state, and trips when that window fits inside about one second.
- The UI itself is a processor: it projects interesting events into readable feed items, which are derived state, not the underlying events.
- Because a reducer is a plain synchronous function, another processor can import and reuse it (for example, a Pi harness's reducer) and build new behavior on the same event types.
- Jonas extends the idea to Pi, Claude, and OpenCode: their events could enter the shared log and be interpreted or extended by other processors.

### Deploy behavior by appending JavaScript · 50:28
- Posting to an unused path implicitly creates a stream.
- The UI offers a "Dynamic Worker Configured" preset whose `script` string holds a reducer and an after-append hook; appending it installs a ping-pong worker.
- Sending `ping` then produced `pong` — a stream with no prior behavior gained it from a JavaScript event. This was the successful live code-as-event demo.
- Processor files can be bundled into an event rather than typed into the UI; Jonas reports having run an AI agent this way, though the full AI version wasn't shown.
- Credentials stay outside the log: the OpenAI key lives in environment variables and is substituted into an outgoing request header.
- He describes a basic agent as roughly 40 lines that can be appended to make a stream an agent; directly entered scripts can't use separately installed npm packages unless bundled.
- Each worker config has a URL-safe slug, and a later event can replace it — giving an agent a path to self-extension by appending different JavaScript under the same identifier.
- A proposed next layer: an "unbundled worker" event carrying a `package.json` and a script, which a separate bundler processor turns into the executable config.

### Plugins, delivery, and who can write · 56:30
- Wrong JavaScript: the current evaluator runs JavaScript without TypeScript checking, and execution failures should surface as error events. A proposed compile step could emit diagnostics that an LLM reads to append corrected code. The settled interface showed an error card ("Failed to fetch remote state.") but no completed repair loop.
- Execution options: a local process subscribes via SSE; a hosted HTTP service receives pushed notifications; a dynamic worker gets its code from a configuration event (this implementation uses Cloudflare Dynamic Workers).
- Remote plugins can contribute proactively: instead of the model calling every tool, the stream notifies a service (say, a prompt-injection checker) that a request is being prepared.
- Jonas proposes waiting up to about 200 ms before an LLM request for such contributions; his MCP comparison concerns proactive hooking into the agent loop, not all server-initiated interaction.
- A third-party processor has no before-append hook — only built-in processors can stop an event from entering the stream, which is why pause can't be implemented by third parties. He prefers this boundary, citing OpenClaw caching, performance, and cost problems he attributes to before hooks.
- The alternative is an eventually consistent loop with a bounded contribution window: once the wait expires, the request proceeds even if a plugin hasn't answered. Good for optional context (say, an indexed Notion knowledge base), but it doesn't guarantee a safety check finishes in time.
- Delivery ownership differs: client-driven replay (Durable Streams, Kafka) has clients track their last offset, while managed subscriptions (Google Cloud and AWS pub/sub) track delivery for subscribers — different responsibilities, not interchangeable guarantees.
- The public-demo risk: anyone who can append to a child path tied to a local model could consume that machine's resources. The answer is authentication plus per-event client provenance (who submitted it, what they could do, why they were authorized); access-policy changes could themselves be events. Proposed default: creator-only writes until an explicit "make public" event. All future work.

## Resources

- **AI Engineer workshop repository** (repository) — https://github.com/iterate/ai-engineer-workshop
- **Workshop walkthrough** (documentation) — the repository's `workshop.md`
- **Durable Streams** (repository) — https://github.com/durable-streams/durable-streams
- **Dynamic Workers getting started** (documentation) — https://developers.cloudflare.com/dynamic-workers/getting-started/
- **JetStream pull consumers** (documentation) — https://docs.nats.io/learn/jetstream/pull-consumers
