# The agent-ready web: expose actions with WebMCP

| Field | Details |
| --- | --- |
| **Speaker** | Tara Agyemang |
| **Affiliation** | Google |
| **Type** | Talk |
| **Date** | 2026-06-11 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=ghJmWQCIHRM) |
| **Tags** | `creative-and-generative-media` `rag-context-and-search` `apis-mcp-and-protocols` `infrastructure-and-deployment` `agent-engineering` |

## Summary

Tara Agyemang, a developer relations engineer on Google's Chrome team, opens with
an everyday annoyance: asking an AI agent to buy two concert tickets. To do that
today, the agent has to parse the whole DOM (the page's element tree), read the
accessibility tree, take a screenshot, guess where to click, then click. It is
slow, brittle, and full of tokens — and a late-loading ad can shift the page and
invalidate the click entirely.

Her answer is **WebMCP** (Web Model Context Protocol), a proposed web standard.
Instead of making the agent infer which controls do what, a site exposes its
capabilities as structured tools — a menu of actions the agent can call. She
compares it to USB-C: one common interface through which agents can discover what
a site supports. She is careful to frame it as complementary to MCP, not a
replacement.

Crucially, WebMCP builds on plain web fundamentals. Semantic HTML, accessibility,
fast loads, and good Core Web Vitals (Google's page-experience metrics) make a site
better for people and give agents a cleaner surface to work from. The standard does
not remove the need for those foundations.

She demonstrates two implementation paths — declarative (annotate an HTML form)
and imperative (register custom tools in JavaScript) — and ends with a Gemini-driven
ticket purchase across three tool calls. The whole feature is early preview and
still changing.

## Key Takeaways

- **Screen scraping is the problem.** Agents infer actions through DOM parsing, accessibility trees, and screenshots, which is slow, token-heavy, and fragile.
- **WebMCP exposes an action contract.** A site publishes structured tools so the agent calls a named action instead of guessing at clicks.
- **Fundamentals come first.** Semantic HTML, accessibility, performance, and Core Web Vitals are prerequisites, not optional extras.
- **Tools follow page state.** In the maze demo, available tools change as the page changes — agents do not work from one fixed global list.
- **WebMCP complements MCP.** MCP connects agents to server-side applications; WebMCP exposes actions inside the user's open browser session.
- **Two APIs, one mental model.** Use the declarative API for standard forms; use the imperative API for complex or multi-step flows.
- **Keep the UI in sync, and keep the user in charge.** Tool calls should update the visible page, and final checkout should return to the human.
- **It is early preview.** The API was changing week to week; expect the code shown to drift.

## Detailed Notes

### Two tickets, too much inference · 00:40
- The demo site, BeatDrop, sells concert tickets; the user asks Gemini (running in Chrome) for two Afrobeats Festival tickets.
- The agent may parse the full DOM, inspect the accessibility tree, and screenshot elements the other representations did not resolve.
- It then locates the target, works out where to click, and clicks.
- Agyemang calls the process brittle and token-heavy without measuring the token cost.
- A single late-loading ad can push content down and invalidate the calculated coordinates.

### Make the page usable before exposing capabilities · 03:03
- The first improvements are ordinary web engineering: semantic HTML, robust accessibility, fast loads, attention to Core Web Vitals, clear user journeys.
- A site that works well for people, she argues, is already partway to being agent-ready.
- **WebMCP** is a proposed web standard that describes a site's capabilities as structured tools.
- The USB-C analogy: a common interface through which agents discover what a site supports.
- She reports better performance and reliability but gives no numeric benchmark or measured token reduction.

### Discovering and calling tools in Maze Escape · 04:43
- Maze Escape, built by Chrome DevRel, is controlled entirely through AI tools — clicking the UI cannot navigate it.
- The **Model Context Tool Inspector** (a Chrome extension side panel) lists the tools a page exposes and allows prompting or direct invocation.
- At the start, only one tool exists: start the maze game. The agent calls it, reads the result, and composes a reply.
- Starting the game changes the available tools: movement (north/south/east/west), inspecting location and open directions, and pick up/drop/use items.
- A request to move "down, then right" becomes calls to the movement tool with south, then east; shorthand like `R` is read as right.
- "Complete the maze" lets the agent loop over tools, but a vague prompt can send it backward; naming the exit's location helps.
- The demo stops early because traversal can take a while.

### Shared browsing and the MCP boundary · 09:26
- Most sites are not agent-only. A user may browse normally, delegate a sequence, then take control back.
- WebMCP supports that shared workflow: the agent acts in the browser where the user already is.
- MCP and WebMCP are complementary — she describes MCP as server-side application access and WebMCP as actions within the user's session.
- Qualifier: an MCP server is a program that can run locally or remotely; WebMCP borrows the tool concept, not the full MCP wire protocol.
- Her JavaScript/Java comparison is about inspiration, not compatibility.
- WebMCP tools live in the browser and require the relevant page to be open, so the page's current context is central.
- Use cases: booking a flight, filtering products, filling complex medical or financial forms, invoking hidden controls.

### Two APIs: declarative and imperative · 12:35
- **Declarative API:** start with a standard HTML form, add `toolname` and `tooldescription`, and the browser generates a JSON schema whose parameters come from the form fields.
- `agentInvoked` is a Boolean on `SubmitEvent` that flags an agent-triggered submission — it is not an attribute recording who filled every field.
- Use the declarative path when a normal form already describes the action.
- **Imperative API:** `registerTool` takes a name, description, a hand-written input schema, and an `execute` function.
- The description matters because the agent uses it to decide *when* to call the tool, not just to label it.
- `execute` runs ordinary JavaScript — in the `addTodoItem` example it validates and trims input, builds DOM nodes, and returns an outcome to the agent.
- Returning a result matters: changing the DOM alone leaves feedback implicit.
- Agyemang expects the imperative approach to be more common, since many apps are more complex than a standard form.

### A ticket purchase through three tool calls · 15:17
- Back on BeatDrop, tools sit alongside featured events, the full listing, and individual concert pages.
- She switches to Gemini 3.1, noting it works better for this demo, and asks for two VIP Summer Vibes Festival tickets.
- `searchConcerts` finds the concert by name and returns its ID.
- `openConcertPage` takes that ID and opens the festival page.
- The new page exposes `purchaseTicket`, called with quantity 2 and the VIP section; the demo shows a £356 notification (a demo outcome, not a real payment).
- The UI stays synchronized: the page opens, VIP is selected, the quantity updates.
- For a real purchase she recommends handing the final checkout step back to the user; preparing and authorizing payment should stay distinguishable.

### Try the experimental workflow · 17:44
- WebMCP is in early preview, and the API was changing fast enough that the shown code might differ the next week.
- Setup at the time of the talk: use a separate Chrome Canary, enable the WebMCP testing flag, then install the Model Context Tool Inspector.
- Later public guidance differs: an origin trial beginning with Chrome 149, and a newer Inspector requiring a recent Chrome build.
- The Inspector warns it lacks production security boundaries, so use it only on trusted sites.
- Recommended entry points are the early-preview announcement and the WebMCP tools repository.
- The repo holds the Inspector, the maze source, roughly six or seven demos, and an evals CLI for testing a site's tools.
- The next step is to exercise tools against real user goals, then report friction and bugs so the API can improve.

### Make site actions explicit · 20:29
- Agents are already using the web; reconstructing actions from screenshots and page structure need not be the only available interface.
- WebMCP's ambition is for a website to act as a high-performance API for agents while staying a good experience for people.
- The invitation is to make capabilities explicit, try them with an agent, and keep the person central as actions become easier to delegate.

## Resources

- [WebMCP early-preview announcement](https://developer.chrome.com/blog/webmcp-epp) — program signup, initial docs, and best practices.
- [WebMCP setup and overview](https://developer.chrome.com/docs/ai/webmcp) — current setup, origin-trial info, and limitations.
- [Declarative API guide](https://developer.chrome.com/docs/ai/webmcp/declarative-api) — expose HTML forms as tools.
- [Imperative API guide](https://developer.chrome.com/docs/ai/webmcp/imperative-api) — register JavaScript tools and manage execution.
- [Model Context Tool Inspector](https://chromewebstore.google.com/detail/webmcp-model-context-tool/gbpdfapgefenggkahomfgkhfehlcenpd) — Chrome extension to inspect and invoke a page's tools.
- [WebMCP tools and demos](https://github.com/GoogleChromeLabs/webmcp-tools) — the Inspector, demo apps, and an evals CLI.
- [WebMCP proposal](https://github.com/webmachinelearning/webmcp) — design rationale and ongoing discussion.
