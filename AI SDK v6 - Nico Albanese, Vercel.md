# AI SDK v6

| Field | Details |
| --- | --- |
| **Speaker** | Nico Albanese |
| **Affiliation** | Vercel |
| **Type** | Workshop |
| **Date** | 2026-05-12 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=wflNENRSUb4) |
| **Tags** | `rag-context-and-search` `apis-mcp-and-protocols` `agent-engineering` `coding-and-developer-tools` `safety-and-governance` |

## Summary

This is a recorded hands-on workshop where Vercel's Nico Albanese builds a streaming TypeScript agent from a fresh Next.js app, using the AI SDK v6 packages. The path is deliberately small at first: define an agent, expose it through a route, render it in the browser, then add tools one at a time.

The core idea is that the agent should be a reusable object, not a pile of prompt and tool code buried inside an API route. He calls this a `ToolLoopAgent` (an agent that can call tools repeatedly). He frames agent building around three building blocks: a runtime that manages the tool loop and context, the tools themselves, and a "computer" — a filesystem where the agent can run code and keep state.

The most interesting part is the persistent filesystem. A workspace gives the agent a plan to follow, memory to accumulate, and a place to save reusable scripts, so a later request can reuse work instead of starting empty. He shows the live debugging this requires: storing a memory is easy, but getting the agent to save the right things means fixing both the instructions and the files already on disk.

He closes with a larger background coding-agent system he runs on the same patterns, using durable workflow steps and sub-agents to keep the main conversation small.

## Key Takeaways

- **Reusable definition:** in v6 the agent's model, instructions, and tools live in one file (for example `lib/agent.ts`) that any server or app can import.
- **Three build blocks:** runtime (tool loop + context), tools, and a computer/sandbox for state and code execution.
- **Three tool types:** custom, provider-defined, and provider-executed — they differ by who defines the interface and who runs it.
- **Typed by default:** the agent definition is the source of truth for UI types; tool inputs and outputs flow to the client through one inferred message type.
- **`prepareStep` reshapes** what the model sees at each step. Pruning saves context but can invalidate cached input, hurting reuse.
- **Named/persistent sandboxes** keep filesystem state across requests even though the process itself is ephemeral.
- **"Memory is a file":** deterministic code injects a `memories.md` into the instructions, while the agent decides what new facts to write.
- **A single Bash tool** is enough: the agent can write and later reuse its own Python scripts, so its workspace builds up capabilities over time.

## Detailed Notes

### Get the demo project running locally · 00:00
- The app is already deployed on Vercel; the first task is running that project locally.
- Clone the repo, then install the Vercel CLI (`npm i -g vercel`) and check the active account with `vercel whoami`.
- Run `vercel link` to connect the local folder to the project, then pull the environment variables.
- The pulled **OIDC token** (a short-lived identity token) authenticates both AI Gateway inference and Vercel Sandbox access.
- Two easy-to-confuse setup failures: the wrong account (`vercel logout` then `vercel login`) and the wrong `vercel` executable when npm and pnpm both have one installed.

### Define a reusable agent and wire it to the UI · 07:31
- Install the dependencies: `ai` (core SDK), `@ai-sdk/react` (React bindings), and `zod` (schema definitions).
- Version 6 keeps the older primitives (`generateText`, `streamText`, `generateObject`, `streamObject`) but pushes structured output into the text functions too.
- A `ToolLoopAgent` packages the repeated tool-calling behavior. The smallest definition only needs a model.
- v6 ships a **global provider** that defaults to AI Gateway, so a plain model string works. You can override the provider if you want.
- The workshop uses **GPT-5.4 mini** as a cheap, fast model for experimenting.
- The HTTP route (`app/api/chat/route.ts`) calls `createAgentUIStreamResponse` and returns the agent's UI stream. The agent owns behavior; the route owns delivery.
- The client uses `useChat` from `@ai-sdk/react`. A message holds `parts`, which the page renders by part type.

### Instructions, and three ways to add tools · 16:59
- Adding `instructions` (a system prompt) changes behavior with one line and no route or client edits — the cowboy example proves the point.
- Web search becomes the first tool, added through the OpenAI provider even though inference still runs through the gateway.
- The three tool categories differ by who defines the interface and who executes it:
  - **Custom:** the developer writes the description, input schema, and execute function.
  - **Provider-defined:** the provider designs the interface for its models, but your app still runs the operation (for example Anthropic's Bash or computer-use tools).
  - **Provider-executed:** the provider both designs and runs it, returning results into the message state (for example hosted web search).
- Provider-executed tools save code but tie the capability to one provider.
- You can pass options to a tool — like a user location — and that extra context travels as a tool parameter.

### Make tool activity visible and fully typed · 23:18
- Tool parts are named `tool-` plus the tool name, so `webSearch` shows up as `tool-webSearch`.
- Adding a render case makes the activity visible, but the first attempt left the input typed as `unknown`.
- The fix is to derive a message type from the agent definition with `InferAgentUIMessage`, then use it in the route and in `useChat<MyAgentUIMessage>()`.
- Types then flow from the server-side tool definition into the client renderer, giving typed tool inputs and outputs.
- The payoff is an inline pending state, so the user can see the agent working instead of waiting for the final prose.

### Give the agent a persistent computer · 27:43
- Motivation: Vercel's internal **d0** analytics agent sits in Slack and reaches backend data and Salesforce.
- With many tools it could still return answers that were partly hallucinated.
- Adding a filesystem gave each session a scratchpad: the objective at the top, an initial plan beneath it, and steps to tick off.
- Instructions required the agent to consult and follow the plan, repeatedly bringing the objective back into focus.
- Reported results include better follow-through and an inspectable record of the work; Albanese also reports roughly 90% fewer support tickets (no measurement period given).

### Control what the model sees at each step · 33:45
- Remote sandboxes are ephemeral. **Named sandboxes** (in beta) let code reference a stable name; sessions are its running instances.
- On inactivity Vercel snapshots the filesystem, and the next request restores it — so the filesystem persists without an always-on process.
- On resending history: `useChat` sends the whole message list by default. A production app can send only the newest message and rebuild the rest on the server.
- `convertToModelMessages` strips UI-only fields (IDs, timestamps); because you own the messages, you can also transform them.
- `prepareStep` runs before each model step and can change that step's inputs — for example, after step 20 keep only the last five messages (a sliding window).
- Pruning context trades against cache reuse: changing earlier input invalidates cached input. He personally now leans on sub-agents and handoff (a fresh thread primed with relevant context) instead of aggressive pruning, since LLM summaries are lossy.

### Pass in a sandbox, add Bash, and keep memory · 45:06
- Install `@vercel/sandbox@beta`; a small helper returns an existing named sandbox or creates one.
- `callOptionsSchema` (a Zod schema) declares call-time inputs the agent needs, here an actual `Sandbox`. The route's `options` then becomes type-checked.
- **Runtime context** (like React context) carries objects and functions down to tools; `prepareCall` runs once per run and copies the sandbox into that context.
- The Bash tool has the usual three parts — description, input schema, execute — and reads the sandbox from the execute function's second argument, keeping the tool independent of any one agent.
- A four-step wiring checklist gets the first `ls -la` working: register the tool, create/get the sandbox in the route, pass it through call options, and add a `tool-bash` render case.
- Memory design: deterministic code reads `memories.md` and injects it into the instructions; the agent decides what new facts to save.
- Live debugging shows both sides matter — it saved a greeting as a fact, then stale memories kept feeding the bad behavior back until those files were deleted.

### Let the workspace build itself · 1:02:58
- The agent can turn repeatable tasks into Python scripts and record short descriptions in `memories.md`, so later runs can find and reuse them.
- Demo: asked for London's weather, the agent wrote and ran Python and stored the capability within one turn.
- Asking for SF weather reused the script, but first resolved "SF" to Quebec; after a fix it returned 12 °C — a reminder that running code successfully is not the same as reading the request correctly.
- Closing system: AI SDK + AI Gateway + Workflow for durable execution, where each LLM step maps to a retried workflow step.
- Reported figures for that system: 23 Vercel users, about 3.8 billion tokens in one to two months, a 90–91% cache-read ratio, and close to 350 PRs.
- The sub-agent demo used about 30,000 tokens and returned roughly 500, keeping the main thread near 7,000 tokens.

## Resources

- **Open Agents** (repository) — an open-source template for deploying your own cloud agents.
- [Amp: Handoff, Please](https://ampcode.com/news/ask-to-handoff) — how asking an agent to hand off starts a fresh thread with relevant context.
- [We Removed 80% of Our Agents' Tools](https://vercel.com/blog/we-removed-80-percent-of-our-agents-tools) — Vercel's account of simplifying its internal d0 analytics agent.
- **How Vercel runs internal agents** (article) — how d0 combines Slack, permission-aware data access, Sandbox, and AI Gateway.
