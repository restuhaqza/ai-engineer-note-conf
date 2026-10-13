# Your Agent Is an Infinite Canvas

| Field | Details |
| --- | --- |
| **Speaker** | Rachel Lee Nabors (RL Nabors) |
| **Affiliation** | Arize |
| **Type** | Talk |
| **Date** | 2026-05-23 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=LMbeDEQO6QM) |
| **Tags** | `apis-mcp-and-protocols` `agent-engineering` |

## Summary

Rachel Lee Nabors argues that the browser is more than a document reader — it is an "infinite canvas" that can render video, audio, and interactive experiences. Using their revived webcomic archive as a test case, Nabors shows how one server can serve three kinds of clients: a person in a browser, a person talking to an agent, and an agent operating the website on a person's behalf.

The demo walks through three layers of MCP (Model Context Protocol, a standard way for agents to call tools and reach servers). First, MCP tools hosted over HTTP instead of a local process, so users add a URL rather than edit a command. Second, MCP Apps — interactive HTML inside the agent — that replace a wall of chat text with an actual comic reader. Third, WebMCP, which lets a webpage expose its own functions and links to a browser agent directly.

Nabors is candid about rough edges: MCP resources are hard to reach in current clients, apps run in a sandbox with no local storage and blocked external assets, and WebMCP is still an evolving standard. The through-line is that existing web primitives — HTML, CSS, JavaScript, and the Web Speech API — are enough to make agents interactive.

Why it matters: it reframes chat as one interface among many. Rich HTML inside agents, and explicit actions exposed from pages, can reduce the token cost and guesswork of interfaces that agents must otherwise infer.

## Key Takeaways

- **The browser is a canvas, not a document reader.** Video, audio, animation, canvas, Wasm, and CSS are all available to agent interfaces.
- **Hosted MCP over HTTP removes local setup.** A user pastes a connector URL instead of configuring a command and arguments; it fits serverless edge functions.
- **Chat is the low common denominator.** Nabors calls a bare chat box "starfish design" — it sits there while the user does all the discovery work.
- **MCP Apps give tools a rich UI.** A tool can return an inline reader built from HTML/CSS/JS, packaged as a single file, keeping the site's design system.
- **The app sandbox has boundaries.** No local storage, blocked external assets, host-mediated links, and server calls through a proxy mean CSP and CORS checks matter.
- **Not every tool should be visible to the model.** Navigation-only tools can be marked app-only so the model doesn't expose their JSON as the experience.
- **WebMCP lets pages expose real actions.** Instead of screenshotting or parsing the DOM, a browser agent calls a registered tool and gets the next URL.
- **Resources are the right idea but underused.** Servers can expose resources, but clients often make them unusable — Nabors asks harness builders to fix this.

## Detailed Notes

### Meet readers where they are · 00:29
- Nabors has worked on Firefox DevTools at Mozilla, the W3C Web Animations API, Microsoft Edge, and React.dev/ReactNative.dev. They are now Principal Developer Experience Engineer at Arize.
- A friendly aside: prompt changes and LLM migrations can break agent flows, and evals help catch those failures.
- Before tech, Nabors made webcomics for teenage girls in the iVillage network, reaching a reported 400,000 weekly readers — learning web development to reach people where they were.
- Examples of the browser as more than text: an interactive Alice in Wonderland using the Web Animations API, a newsletter called The Agentic Web, and ASSA (Antisocial Social Agent), an MCP app that puts Twitter and Bluesky side by side.

### One archive, three client relationships · 03:28
- The project is a neglected comic archive from around 2010, by then full of 404s, broken images, and failed infrastructure including a CDN.
- Repairing it became a chance to also make the comics available to agents, while still serving the women who send fan mail.
- Nabors uses their own comics (`rachelthegreat.com`) to avoid copyright issues, and mentions "glazing" artwork to discourage training use.
- The target is **one server serving three client relationships:** human in a browser (visit and navigate the site), human in an agent (ask for comics and use an embedded reader), and agent in a browser (operate the site on a person's behalf).
- Hosting matters: a server on the developer's machine is different from one readers can connect to on the web.
- Nabors predicts MCP Apps could become the next web pages, and tests that idea against a real body of content.

### Transports: stdio vs HTTP · 05:57
- A transport is how the client and server exchange messages.
- **stdio** (standard input/output): the client launches the server as a local process; the user's config supplies a command and arguments, and the process stays alive for the session.
- **HTTP:** the server runs as a web service listening at an endpoint, using POST requests; it fits serverless hosting like Vercel and Cloudflare edge functions.
- The Streamable HTTP spec also defines GET/SSE behavior and optional sessions; POST is just the client-to-server direction.
- Security and privacy design were explicitly out of scope for the demo.
- In Claude, the recorded setup: open connector settings, enter a name and the server URL (the demo used `/MCP`), hit Add, then review the connector's tool permissions.
- The URL removes local launch config, but the buried settings still add friction; Nabors grants always-allow because it is their own familiar server.

### Navigation tools are not the reading experience · 08:57
- The first tools mirror the site's navigation: list comics, storylines, and characters; search transcripts; and find pages featuring a character.
- Storylines keep the arcs, beginnings, and endings navigable; character search answers a personal request like "show every appearance of the cat."
- These tools return structured JSON, except transcript retrieval, which returns Markdown.
- The tools-only demo is verbose: Claude narrates its discoveries at length, and finding a comic is not the same as giving someone a comfortable way to read it.
- Nabors wanted the full transcripts of 533 comics available for theme and character-arc analysis, rather than fetching one comic at a time.
- **MCP resources** are the intended vehicle for that reference context, but in the clients Nabors tried they existed on the server with no usable way to reach them.
- A related annoyance: coding agents are reluctant to call an MCP tool just to add Markdown docs to context; Nabors proposes harnesses that pre-prime context (e.g. fetch React resources when switching to React work).

### MCP apps: a comic reader instead of chat · 11:31
- Nabors compares chat's role to the command line's role in software; a blank chat box demands expertise the user may not have.
- **MCP Apps** provide inline rich interfaces in familiar HTML, CSS, and JavaScript. Nabors bundles them into a single file — closer to DHTML than to a new visual programming model.
- The demo asks Claude to read *Crow Princess*; Claude finds it and calls **Get Page**, which returns an embedded reader.
- The reader reveals creator commentary and reader comments, moves forward and backward, and switches to text mode to show the transcript.
- It retains the website's look because it reuses the site's resources.
- The summary: the conversation finds the work; the reader supplies the controls for actually reading it.

### Sandbox boundary: links, CSP, CORS · 14:33
- Get Page has a description and an input schema (comic ID, storyline ID, page number). Nabors notes the comic ID is redundant and plans to remove it.
- The key addition is UI metadata pointing at the app resource, documented as `_meta.ui.resourceUri`, which names the HTML the host retrieves and renders.
- The reader runs in a sandboxed iframe described as an "island": one HTML file with assets embedded (Base64 where needed), no usable local storage across iframe instances, and server calls made asynchronously via `callServerTool`.
- The economical strategy is to reuse the site's design system — share fonts and CSS, configure resource access, and bundle with Vite single file.
- **Links** need host permission: use the host-mediated `appRef.current.openLink` rather than assuming `window.open` or a plain anchor works.
- **External assets** can be blocked by Content Security Policy (CSP); a missing font or image is a reason to check CSP first.
- CSP and CORS are different checks: `resourceDomains` permits asset origins and `connectDomains` permits connection origins, while the destination server's own CORS policy governs cross-origin access.
- Some tools exist only to support the interface, not to give a useful conversational answer; mark them with `visibility: app` so the model does not surface their JSON as the reading experience.

### WebMCP: let browser agents call page actions · 17:33
- Browser agents often navigate by interpreting screenshots or traversing the DOM — both compute-heavy, spending tokens to discover something as simple as the next URL.
- **WebMCP exposes the page's existing actions directly**, treating each HTML page like a miniature tools server.
- Nabors warns WebMCP is to MCP roughly what JavaScript is to Java: inspired by, but not one-to-one compliance with, the MCP spec — the standards may diverge.
- Two approaches: **declarative** (annotated forms, good for search/forms/dashboards) and **imperative** (registered JavaScript callbacks, good for navigation, API calls, and workflows).
- Because the comic site has few forms beyond search, Nabors mostly uses imperative registration.
- The procedure: feature-detect `modelContext`, then register a name, description, input schema, and an execution callback.
- For next-page navigation, the callback finds `link[rel="next"]` and returns its destination; it does not itself call `location.assign`.
- The demo uses the MCP-B debugging extension, whose sidebar exposes the page's tools and a chat window. A request to read returns the transcript; "go to the next page" calls the exposed function and the browser advances.
- The standard was still developing; later Chrome docs use `document.modelContext.registerTool` and spell the declarative attributes `toolname`/`tooldescription`. Browser support was incomplete at the time.

### The browser can also read aloud · 21:13
- Nabors started exploring the agentic web about two years earlier with a provocative claim that the browser would die; the point now is that the web is evolving, not dying.
- CSS and JavaScript are also the languages of interactive experiences inside agents.
- A closing demo has the reader speak panel descriptions and dialogue, illustrating the continuity between the web and agents.
- The narration uses the browser's **Web Speech API**; a reader just calls `window.speechSynthesis.speak` with a `SpeechSynthesisUtterance`, and a stop control calls `cancel`.
- Nabors calls it a zero-dependency solution, dislikes the voice quality, and suggests ElevenLabs as an alternative.
- Speech is one of many browser capabilities available; animation, audio, canvas, Wasm, and CSS all already exist for making content interactive.
- One caveat: speech voices can use local or remote services, so "no inference required" is not guaranteed by the API alone.

## Resources

- [MCP Webcomic Site Server (companion repo)](https://github.com/nearestnabors/mcp-webcomic-site-server)
- [Rachel the Great (comic archive)](https://rachelthegreat.com/)
- [Nabors interviews Alex Nahas about WebMCP](https://www.arcade.dev/blog/web-mcp-alex-nahas-interview/)
- [MCP Apps overview](https://modelcontextprotocol.io/extensions/apps/overview)
- [CSP and CORS for MCP Apps](https://apps.extensions.modelcontextprotocol.io/api/documents/csp-and-cors.html)
- [MCP transport specification](https://modelcontextprotocol.io/specification/2025-11-25/basic/transports)
- [MCP resource specification](https://modelcontextprotocol.io/specification/2025-03-26/server/resources)
- [Chrome WebMCP imperative API](https://developer.chrome.com/docs/ai/webmcp/imperative-api)
- [Chrome WebMCP declarative API](https://developer.chrome.com/docs/ai/webmcp/declarative-api)
