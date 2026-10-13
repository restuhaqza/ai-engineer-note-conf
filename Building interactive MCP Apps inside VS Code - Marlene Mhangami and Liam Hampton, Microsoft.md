# Building interactive MCP Apps inside VS Code

| Field | Details |
| --- | --- |
| **Speaker** | Marlene Mhangami, Liam Hampton |
| **Affiliation** | Microsoft |
| **Type** | Talk |
| **Date** | 2026-06-06 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=_xIwFcnHqp4) |
| **Tags** | `apis-mcp-and-protocols` `enterprise` `rag-context-and-search` |

## Summary

Marlene Mhangami and Liam Hampton explain how MCP Apps turn text-only tool responses into interactive interfaces rendered right inside the chat. MCP (Model Context Protocol) is an open standard, created by Anthropic, for how applications give context to language models. MCP Apps go further: a server tool can return a rich UI that the chat host renders for you.

The pair walk through the moving parts. A tool produces data; a resource supplies the HTML interface; a link connects the two. The host (VS Code in this talk) fetches and renders that HTML inside a sandboxed iframe, so the app stays interactive without getting the editor's full authority.

Hampton then builds a live example: a Go profiler. A local MCP server runs a Go program, profiles it with `pprof`, and returns the data; a React frontend draws it as a flame graph inside Copilot chat. The goal is to cut the back-and-forth of asking the model "is this good?" and just show the data.

## Key Takeaways

- **MCP Apps put interfaces in the chat.** Server tools can return interactive components, not just text.
- **Know the three roles.** Host (the app, e.g. VS Code), client (the connection, e.g. Copilot), server (exposes capabilities).
- **A server is executable software.** Installing one is a trust decision; prefer curated listings over random downloads.
- **Three pieces make an app.** A tool (data), a resource (HTML UI), and the link that tells the host which UI shows that data.
- **The host renders; the model chooses the tool.** The host fetches and renders the HTML, which the model need not do.
- **Clicking can beat prompting.** Buttons and controls spare the user from translating a visual intent back into text.
- **Sandboxed iframes contain the risk.** The app is kept from reaching VS Code settings or external systems.
- **Visibility controls who can call a tool.** Model only, model and app, or app only.

## Detailed Notes

### What would it take to put an interface inside chat? · 00:15
- Mhangami is a senior developer advocate at Microsoft and GitHub; Hampton works on developer-tools advocacy for VS Code and GitHub Copilot.
- MCP is an open protocol, created by Anthropic, that standardizes how applications provide context to models. Its capabilities include tools, prompts, and resources.
- Three roles:
- **Host:** the application that wants server capabilities, such as VS Code.
- **Client:** maintains a one-to-one connection with a server; here, GitHub Copilot.
- **Server:** a lightweight program exposing capabilities through MCP.
- You can build a server or use an existing one. In VS Code's Extensions view, searching `@mcp` lists available servers.
- A server is executable software, so choosing one is a trust decision. They recommend the VS Code or GitHub listings, since an arbitrary server from the internet can contain malicious code.

### From diagram text to an editable diagram · 03:10
- Ask a chat to draw a diagram explaining MCP, and early responses were often text: ASCII art or a README covered in emojis.
- The real limitation was the interaction experience. A text diagram leaves the user without something they can directly manipulate.
- With the Excalidraw MCP App server, the same request returns an actual diagram. The user can move elements and edit text without leaving the conversation.
- MCP Apps let server tools return rich interactive components that render directly in chat, improving both usefulness and looks.

### The host turns a resource into an interface · 05:15
- Walk through a simple request like "Show me analytics":
- The user sends a prompt.
- The agent picks a tool exposed by an MCP server.
- The server returns results plus a UI resource reference pointing to HTML.
- The host — VS Code, not the Copilot client — fetches that HTML and renders it inside a sandboxed iframe.
- The user interacts with the app, which can request more data and update its display.
- The key division: the tool produces data, a resource supplies the interface, and the host connects them. The model selects the tool; the host renders the UI.
- In the current architecture, the tool definition declares a UI resource URI so the host can discover (and potentially preload) the interface before the tool finishes.

### Where clicking beats another prompt · 06:34
- In data exploration, an initial question might reveal a trend in a bar chart. The next step is usually narrower: inspect specific values or switch charts.
- Repeatedly describing those actions in text forces the user to translate a visual intention back into language. Controls let them take the next step directly.
- Commerce is a related example, aimed more at general chat products than VS Code. Instead of returning links that send the buyer to a browser, a shopping assistant can keep selection and checkout inside chat.
- Examples span several kinds of interface:
- **Shopify:** preserves a merchant's branding in rendered components, with a full checkout inside chat.
- **Excalidraw:** turns architecture diagrams into interactive visualizations.
- **Figma:** generates components on the fly, though no rendered example is shown.
- The shared goal is to keep the task's useful interaction surface next to the conversation.

### Build a tool, a resource, and the link · 09:46
- Hampton starts from a skill in the mcp-apps repository, edits it, and runs it through GitHub Copilot CLI to generate several apps.
- His demo repository includes the FlameGraph example plus a Markdown viewer, a flight-status app, and a color picker. The aim is to stay within one working context: VS Code.
- Three essential pieces:
- **Tool:** a server operation the model can select.
- **Resource:** the bundled HTML interface.
- **Link:** the association telling the host which UI shows which data.
- The frontend can be React, Vue, vanilla JavaScript, or Svelte.
- The skill supplies setup and run instructions, code examples, handlers, and tool-visibility guidance.
- Visibility decides who invokes a tool: **model only** (the model selects it in conversation), **model and app** (both can), or **app only** (the interface calls it without exposing it to the model).
- That choice belongs in the design, because an interactive control and an agent may need different operations.

### Turn a Go workload into profiling data · 11:31
- The Flamegraph Profiler app profiles a Go program containing a bubble sort and a Fibonacci workload.
- Bubble sort repeatedly compares and swaps neighboring values; the recursive Fibonacci function repeatedly evaluates smaller inputs. Both give recognizable functions whose costs can be inspected.
- Hampton sets a five-second profiling window using Go's `pprof`. Five seconds is the collection window, not a reported speedup.
- The MCP server runs on localhost via a TypeScript implementation. The important operation prepares the Go program, runs it, profiles it, and returns the data.
- The linked frontend is a React app called Flame App. It uses hooks to receive the tool input and results, then draws the profile as a flame graph.
- That is the practical value of the tool/resource split: profiling stays server work, while the frontend makes the data easier to read.

### Inspect the result inside Copilot chat · 13:06
- With the server installed and running, Hampton opens GitHub Copilot. It exposes one tool, "Profile the App."
- He asks Copilot to profile the application; Copilot recognizes the installed Flame Graph Profiler and selects the tool, loading the MCP App.
- The interface appears inside an iframe in the chat window, showing top functions and a profiling summary. Its size can be adjusted to fit.
- Instead of repeatedly asking the model whether the profile is good or bad, the user can read the function data directly.
- His claim is about removing that conversational back-and-forth: the relevant data is already visible.

### Keep the interface useful and contained · 14:21
- The finished demo follows the analytics path: the request reaches the model, the model selects the tool, the server returns JSON, and the host recognizes the linked UI and renders it in chat.
- The model does not have to turn every part of the profile into prose before the user can explore it.
- Hampton explains the iframe with a hamster analogy: loose in a room, a hamster may chew things up; in an enclosure, its activity stays contained.
- The goal is to keep the embedded app from having unrestricted access to VS Code settings or external systems.
- An iframe is part of that boundary, not a total ban on external access: host permissions and content security policy govern permitted resources, and tool calls are mediated by the host.
- The interface can stay interactive without receiving the editor's full authority.

## Resources

- [Liam Hampton's MCP Apps examples](https://github.com/liamchampton/mcp-apps) — example apps with local setup instructions and VS Code configuration.
- [Flamegraph Profiler MCP App](https://github.com/liamchampton/mcp-apps/tree/main/flamegraph-profiler-mcp) — the Go profiling example with an interactive flame graph.
- [Official MCP Apps SDK and examples](https://github.com/modelcontextprotocol/ext-apps) — specification, SDK, framework examples, and skills.
- [MCP Apps architecture and security](https://modelcontextprotocol.io/extensions/apps/overview) — current documentation for UI resource declarations and host containment.
- [Excalidraw MCP App](https://github.com/excalidraw/excalidraw-mcp) — MCP App server for streamed, interactive Excalidraw diagrams.
