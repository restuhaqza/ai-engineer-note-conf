# Building an ACP-Compatible Agent Live

| Field | Details |
| --- | --- |
| **Speaker** | Bennet Fenner |
| **Affiliation** | Zed |
| **Type** | Talk |
| **Date** | 2026-07-08 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=HsxQICTLF84) |
| **Tags** | `apis-mcp-and-protocols` |

## Summary

Bennet Fenner, a developer at the Rust-based code editor Zed, starts with a problem every editor now faces: coding agents like Claude Code, Codex, and Gemini CLI each speak their own language. Wiring up each one separately means solving the same communication job over and over. His answer is the Agent Client Protocol (ACP), an open, JSON-RPC-based way for agents and editors to talk through one shared interface — much like MCP for tools or LSP for language servers.

The rest of the talk is a live build. Starting from a tiny TypeScript coding agent with just two tools (read a file, edit a file), Fenner adds ACP support one piece at a time. He adds version negotiation and sessions, fixes a prompt that finishes with no visible answer, then layers on streamed text, visible tool calls, editor-aware file reads, and diff display. The stated goal is ten minutes; it ends up closer to fifteen.

The value is a concrete look at the "agent side" of ACP: how an ordinary loop — call the model, run tools, repeat — gets connected to an editor without rewriting its logic. Fenner is candid about rough edges (some duplicated output he never debugs) and warns viewers not to use the AI-generated demo code in production.

## Key Takeaways

- **ACP (Agent Client Protocol)** is an open, JSON-RPC-based protocol between coding agents and clients (editors), similar in spirit to MCP and LSP.
- Agents can join through an **adapter** that translates their native interface, or by implementing ACP directly — OpenCode and Cursor ship ACP modes, and about forty clients supported it at talk time.
- ACP does not replace the agent's **model-and-tool loop**; it connects that loop to a client.
- A **session** is one editor thread. `session/new` returns an ID that later prompts use to reach the same conversation and history.
- Handling a prompt request is not the same as **publishing the response**: without session updates, the editor shows nothing even though the model is producing output.
- Model text streams as `agent_message_chunk` **session updates**, while tool work uses `tool_call` first, then `tool_call_update` with the same ID.
- Read and write are **separate capabilities**. When the client offers filesystem access, the agent can read the editor's buffer — including unsaved changes.
- The demo connects over **standard I/O**; the agent sends diff text (old and new) and the client computes the visual diff.

## Detailed Notes

### Why an agent-editor protocol · 00:15
- Zed is an AI code editor written in Rust. The rise of terminal coding agents (Claude Code, Codex, Gemini CLI) raised a question: how do you let users bring their preferred agent and still get one consistent interface?
- Supporting each agent with a bespoke integration would mean solving the same communication problem repeatedly.
- **ACP** gives agents and clients a shared interface. It is open source and JSON-RPC based, playing a role analogous to MCP (a standard way for agents to call tools) or LSP (a standard for editor-to-language-server links).
- Fenner estimates up to forty clients supported ACP at the time; JetBrains and Obsidian are among them. OpenClaw acts as both a client and an agent.

### Start with the model loop · 02:03
- The starting code is a minimal TypeScript agent with two tools: read a file, and edit an existing file by swapping old text for new text.
- Model APIs are stateless, so the agent keeps the conversation and resends the accumulated history with each request.
- The loop runs: append the prompt and call the model → an `end_turn` stop reason means the turn is done → if a tool is requested, run it locally → add the result and call the model again.
- ACP does not need to change this loop. It needs to connect the loop to a client.

### Add sessions and cancellation · 04:11
- `initialize` returns the protocol version the agent supports. Initialization negotiates a version: return the client's requested version if supported, otherwise the agent's own latest.
- Both sides can advertise capabilities. The demo keeps them minimal and skips an interactive auth flow because the API key comes from an environment variable.
- A **session** maps to one editor thread. Creating one generates a random ID, builds an agent instance with the client-supplied working directory, stores it in an internal map, and returns the ID.
- A prompt request carries the session ID plus content blocks (text is all this agent handles). The adapter looks up the session, filters to text, and calls the existing prompt function.
- Cancellation is wired through to the session's agent as a convenience.

### Debugging: a prompt with no visible answer · 06:51
- Zed needs no special code — its config names an ACP-compatible agent, launched with Node and the path to the compiled JavaScript file.
- After rebuilding and prompting, Zed shows a wait indicator and then finishes. Nothing appears.
- The in-editor **ACP debug view** shows the traffic: `session/new`, a session ID returned, a prompt, and a stop reason — plus red adapter errors.
- The lesson: the model is producing output behind the scenes, but the agent never forwards it. Handling the prompt request is not the same as publishing the answer.

### Stream text as session updates · 08:16
- To surface output, the agent is handed the ACP connection and the session ID.
- The Anthropic SDK exposes text events; each chunk becomes a **session update** — a notification tied to the conversation, not the final response to the prompt request.
- The update type is `agent_message_chunk`; its content holds the new text, and its enclosing message identifies the session.
- After a rebuild, a greeting streams into Zed chunk by chunk. The client can render notifications as they arrive rather than waiting for the stop reason.

### Publish tool calls, then update them · 10:21
- Text alone does not show what the agent is doing. Asking it to read a file yields a description, but the file-read operation itself stays invisible.
- The agent first emits a `tool_call` update carrying an identifier, a title, metadata, an `in_progress` status, and file locations. Those fields let the client pick an icon and tie the action to a file.
- Afterwards, `tool_call_update` messages reuse the same identifier to change status and return result content.
- Ordering matters: the first notification creates the object the client displays; later ones modify it instead of introducing unrelated operations.

### Editor-aware reads and visible diffs · 12:18
- The read handler still uses direct filesystem access. ACP offers another path: when the client advertises filesystem capability, the agent can ask the client to read the file instead.
- The reason is an everyday editor state — **unsaved changes**. A buffer may hold code that is not on disk yet, so reading through the editor returns what the user actually sees.
- Read and write are separate capabilities. The agent still returns text to the model; only the component supplying that text changes.
- The edit handler gets the same treatment. The agent sends diff content (file path plus old and new text), and Zed renders the diff. Duplicated output appears and is left undiagnosed.

### Let the agent build its next tool, then wrap up · 14:59
- With reads and edits working, the agent can modify itself. Fenner feeds it a prepared prompt to add a terminal tool, including pointers to the relevant ACP APIs.
- Terminal support follows the same capability boundary: a client can advertise that it creates and manages terminals, giving the editor an interactive view of the running operation.
- The generated tool builds and runs. Asking the agent to wait five seconds and then list the directory shows a running terminal and its output.
- In Q&A, Fenner confirms the client computes the diff from the agent's old and new text. The connection is local, over **standard I/O**; he says remote-transport work is underway (tentatively by JetBrains) but is not shown here.
- He frames the conversion as roughly fifteen minutes of work and warns the agent-generated demo code should not be used in production. At recording time the companion repository was still empty.

## Resources

- [Agent Client Protocol – introduction](https://agentclientprotocol.com/get-started/introduction)
- [ACP TypeScript SDK](https://agentclientprotocol.com/libraries/typescript)
- [Initialization and capability negotiation](https://agentclientprotocol.com/protocol/v1/initialization)
- [Tool updates and diff content](https://agentclientprotocol.com/protocol/v1/tool-calls)
- [Client-mediated file access](https://agentclientprotocol.com/protocol/v1/file-system)
- [OpenClaw ACP bridge and client](https://docs.openclaw.ai/cli/acp)
- [Cursor CLI ACP mode](https://prod.cursor.com/docs/cli/acp)
