# Why Your AI UX Is Broken (and It's Not the Model's Fault)

| Field | Details |
| --- | --- |
| **Speaker** | Mike Christensen |
| **Affiliation** | Ably |
| **Type** | Talk |
| **Date** | 2026-05-17 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=YNJvm7t3yq8) |
| **Tags** | `architecture` `agent-engineering` |

## Summary

Most AI chat apps stream responses over a single, direct HTTP connection using
server-sent events (**SSE** — a one-way stream from server to browser). It is the
default in popular frameworks and it is easy to get working, but it assumes one
client talks to one agent over one connection.

Mike Christensen, a staff engineer at Ably (a real-time messaging platform), argues
that this assumption limits what AI products can do. When the connection drops, the
response is lost. A second tab cannot see work that is in progress. And because SSE
has no upstream channel, a "Stop" button is ambiguous — closing the connection looks
the same as a network failure.

His fix is a **durable session**: a persistent, shared resource that sits between
clients and agents. Built on publish/subscribe channels, it outlives any single
connection, so clients can reconnect and resume, multiple devices stay in sync, and
any client can steer or cancel an agent while it works. He presents Ably AI
Transport as a drop-in SDK for this pattern.

## Key Takeaways

- **Direct HTTP streaming couples everything to one connection:** one client, one agent, one pipe.
- **SSE is one-way**, so a Stop button and resumability conflict — the closed connection is ambiguous.
- **Three capabilities separate a demo from a product:** resilient delivery, continuity across devices, and live control.
- **A durable session decouples agent and client.** The agent publishes activity; clients subscribe and reconnect to the same resource.
- **Resumption is per client.** Retain events, number them, and replay only what each client missed — the shared layer can own this.
- **WebSockets add bidirectional control** but do not by themselves create a shared session.
- **Publish/subscribe gives three useful properties:** independent addressing (a channel name), persistence, and resumption.
- **Sessions support more than chat:** multi-agent progress, concurrent work, and human handoff that keeps the history.

## Detailed Notes

### The default chat architecture · 00:28
- Christensen asks who has built an AI chat app — nearly the whole room.
- The default pattern is **direct HTTP streaming**: the browser opens a persistent connection, the agent calls an LLM, and events stream back.
- Server-sent events (**SSE**) is the default in Vercel's AI SDK and TanStack AI.
- It is easy to get something working, but the paradigm assumes one client → one agent → one connection.
- Ably is a real-time messaging platform; Christensen reports platform scale of 2B+ devices, 30B+ monthly connections, and 2T+ API operations (his reported figures, not AI benchmarks).
- Over a year, Ably spoke with engineering teams at 40+ companies across ten industries shipping agents and assistants to millions of users.

### Three requirements that outlive a request · 03:03
- **Resilient delivery:** walking out of Wi-Fi into 5G, refreshing, or navigating away should not lose the response; clients should resume where they left off.
- **Continuity across surfaces:** a new tab or phone should show the same session, including live activity.
- **Live control:** users should be able to talk to an agent while it works — like watching Claude Code and steering it.
- A request-bound stream makes all three harder: delivery depends on the original connection; a second tab sees nothing; other devices cannot reach the working agent.
- A **durable session** is a persistent, stateful resource between clients and agents.
- The work's lifetime no longer has to match the client connection's lifetime.

### Resume from each client's last event · 06:06
- If a client disconnects mid-response, later events have nowhere to go.
- Resumption needs four things: retain events (for example in Redis), assign sequence numbers, handle a resume request, and replay the missed events in order.
- This bookkeeping is per client — different clients disconnect at different points and need different ranges.
- When the agent manages delivery, it must run this recovery for every reconnecting client.
- With a durable session, the agent writes events without checking who is connected; each browser reconnects and gets its missing events from the session.
- Retention and replay still exist, but they move into the shared delivery layer instead of agent logic.

### A disconnect must not double as a Stop command · 07:43
- Adding a Stop button exposes a conflict: SSE carries events only server → client.
- Closing the connection is ambiguous:
  - Temporary interruption → keep generating and buffer for resumption.
  - User cancellation → stop generation and avoid further token cost.
- The same signal cannot reliably mean both.
- Christensen cites the AI SDK's abort/resume warning as documented evidence that abort and resumable streams conflict in that SDK (not a universal rule).
- One option is to keep SSE downstream and add a separate upstream control path.
- **WebSockets** are his proposed alternative — bidirectional, so explicit control messages can arrive while output still streams.
- Changing transport opens steering, but by itself it does not create a shared session.

### Reach the working agent from another device · 09:13
- With WebSockets but no shared session, the tab that started the work still owns the connection, so a second tab has no visibility.
- Bidirectionality solves the direction of communication, not its scope.
- Flight-booking example: ask for next Tuesday, switch to the phone, and now need Wednesday.
- The phone must first see the ongoing work, then send the correction to the agent handling it.
- A private connection belonging to the original device supplies neither ability automatically.
- With a durable session, all clients hold connections to the same resource and continuously observe activity.
- The agent also observes the session, so the phone has a route to send the correction (which does not mean a flight is already booked).

### Separate orchestration from progress delivery · 11:11
- Multi-agent pattern: a user asks an orchestrator, which delegates subtasks to specialized agents.
- To show progress through one response, the orchestrator would also have to collect and relay every sub-agent update.
- That relay job is separate from deciding what work to delegate and collecting final results.
- In Christensen's view, the orchestrator wants the sub-agents' results, not every granular update.
- With the session as the shared destination, each agent writes its own updates directly.
- The client subscribes to one session and sees activity from all agents and from other clients.
- Adding a specialist no longer requires routing its progress through a central agent just to reach the interface.

### Build the session on persistent pub/sub · 12:54
- The pattern resembles **publish/subscribe**: participants communicate through a shared resource rather than directly; Ably calls that resource a channel.
- Three channel properties support a durable session:
  - **Independent addressing** — a channel name identifies the session.
  - **Persistence** — messages outlive connections, devices, and agent instances.
  - **Resumption** — a reconnecting client receives events from where it left off.
- Recovery has boundaries; the talk does not specify retention periods or recovery windows.
- **Ably AI Transport** is presented as a drop-in SDK that packages the durable-session pattern on channels; its design accepts many event stream formats, with codecs for frameworks.
- It adds: response materialization (chunks → complete response), automatic resumption, multiplexing of concurrent activity, and fan-out plus bidirectional control.
- Supporting UX includes push notifications for finished background work and shared/subscribable data objects.

### Demo: surviving interruption and handing off to a human · 15:34
- Demo: an electronics-shop support chat; the agent makes a client-side tool call (user location) and a server-side call (nearby stores).
- The conversation appears in multiple tabs with a shared view; refreshing keeps everything in sync.
- Forcing a network disconnect and reconnect, delivery continues — attributed to the client consuming and subscribing to the channel, with no extra recovery logic in the agent.
- A second tab starts a product-research sub-agent and shows granular progress; the first tab cancels it, so control belongs to session participants, not just the initiating browser.
- Direct publication: the specialist writes its own events without a centralized orchestrator relaying them; two agents run at once and stay synchronized.
- Final handoff: an unhappy customer asks for a human; a support-agent view opens on the same session, reads the full AI history, and replies directly — no reconstruction needed.

## Resources

- "Why production AI needs a session layer, not just a stream" — Christensen's companion article (related).
- [AI SDK: Abort breaks resumable streams](https://ai-sdk.dev/docs/troubleshooting/abort-breaks-resumable-streams) — documents the resumption/cancel conflict (mentioned).
- [TanStack AI connection adapters](https://tanstack.com/ai/latest/docs/chat/connection-adapters) — current transport options and examples (update).
- [Ably connection state and recovery](https://ably.com/docs/connect/states) — reconnection, recovery windows, and page-refresh handling (related).
- [Ably AI Transport](https://ably.com/docs/ai-transport) — SDK documentation for shared agent sessions (mentioned).
- Ably Server-Sent Events — shows how SSE subscribers receive channel events and resume delivery (related).
