# AI Didn’t Kill the Web, It Moved in!

| Field | Details |
| --- | --- |
| **Speaker** | Olivier Leplus, Yohan Lasorsa |
| **Affiliation** | Amazon Web Services (AWS) · Microsoft |
| **Type** | Talk |
| **Date** | 2026-04-10 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=XZ0boOjtbNo) |
| **Tags** | `agent-engineering` `apis-mcp-and-protocols` `coding-and-developer-tools` `developer-workflows-and-testing` |

## Summary

Yohan Lasorsa (Microsoft) and Olivier Leplus (AWS) argue that AI did not replace the web — it moved in at every stage of the web-development lifecycle. Using a shared e-commerce demo called Seine, they show AI helping with coding, debugging, performance tuning, and even running inside the browser.

First, a coding agent implements a GitHub issue. What makes it work is skills: small text files (`SKILL.md`) that give the agent specific know-how, loaded only when relevant. The agent grabs the issue with the GitHub CLI, writes the feature, records a browser video with Playwright, exposes a tunnel, and pings the developer's phone via Telegram — but leaves the issue open until a human confirms.

Then they flip the vantage point. Chrome DevTools MCP lets an external agent drive the browser, and AI built into DevTools can explain a CORS error or a performance trace in place, then write a CSS fix back to disk.

Next, browser-native AI APIs run a model locally on the user's machine: the Summarizer API condenses reviews, the Proofreader API returns corrected text with edit positions, and the Prompt API turns an uploaded photo into structured review fields.

Finally, they make the site usable by agents: `llms.txt` gives agents a map of the docs, and the experimental WebMCP lets a page expose its own actions (like add-to-cart) as structured tools.

## Key Takeaways

- **AI now touches every web-dev stage:** coding, debugging, performance, the browser itself, and the live app.
- **Skills are small text files** that give a coding agent task-specific expertise, loaded only when needed.
- **`AGENTS.md` encodes repeatable workflows** such as recording a video, opening a tunnel, and notifying a phone.
- **Chrome DevTools MCP exposes browser tools** (clicks, console, network, Lighthouse) to coding agents.
- **AI inside DevTools** can explain errors and traces, and can persist a CSS fix back to the source file.
- **Browser-native AI APIs** (Summarizer, Proofreader, Prompt) run a local model, avoiding a remote call per request.
- **`llms.txt` / `llms-full.txt`** give agents a structured, up-to-date map of your documentation.
- **WebMCP is experimental:** it lets a site expose its own actions as tools instead of making agents guess the UI.

## Detailed Notes

### AI across the web development lifecycle · 00:01
- Over the last six months, better models plus the integrations around them changed what web developers can do.
- AI now appears in development, debugging, performance tuning, and inside the browser.
- It comes full circle as agents increasingly use web apps alongside humans — so apps must adapt.

### Coding agents, skills, and the Seine demo · 01:41
- Demo app: Seine, an example e-commerce site with product descriptions and customer reviews.
- Prompt: "look at open issues and implement the first one." The agent uses the GitHub CLI without being told.
- Skills live in `.agents/skills`; each `SKILL.md` has a name, a description, and instructions or command examples.
- The description decides when a skill enters the agent's context. Skills cover the GitHub CLI, frontend design, and Playwright.
- Custom skills create a public tunnel to the dev machine and send the URL via Telegram.
- `AGENTS.md` defines the workflow: implement, record a video, run the dev server, open a tunnel, notify, and keep the issue open.
- The Telegram step first failed (missing token); telling it to use the existing `.env` and retry fixed delivery, no rebuild needed.

### Debugging with Chrome DevTools MCP · 07:29
- Chrome DevTools MCP is an MCP server (a toolbox an agent can call) exposing Chrome's browser controls.
- Tools include navigation, clicks, form entry, console/network inspection, Lighthouse audits, screenshots, and viewport resizing.
- Asked to "test the main page," the agent reads `package.json`, starts the server, opens Chrome, and captures screenshots.
- For performance it runs fast, 3G, and 2G traces, then reports LCP, CLS, critical-path latency, and render-blocking savings.
- Findings included an oversized headphone image and suggestions to improve CSS/JS and preload JSON — recommendations, not applied fixes.

### AI assistance inside DevTools · 16:02
- Reverse the angle: AI embedded in DevTools inspects the current page without copy-pasting errors elsewhere.
- It must be enabled under the DevTools settings' AI-related options; it is off by default.
- On Seine's About page, clicking the AI icon explains a CORS-blocked (cross-origin) request and suggests fixes.
- Selecting a failed network request adds it to the assistant's context; the analysis flags a possibly missing endpoint.
- A recorded performance trace can be analyzed for render-blocking work, though answers vary between runs.
- The assistant can edit the live DOM — for example, turn an `h1` into a gradient. That edit is temporary.
- "Apply to Workspace," with the source folder added, writes the CSS change back to the project instead of leaving it lost.

### Browser-native AI: summarize and proofread · 23:24
- Instead of a remote provider, browser-native AI APIs expose a model running locally on the user's machine.
- Summarizer API setup: check availability, choose shape and length, set languages, and describe the input data.
- A downloaded model (~4 GB here) is shared across websites and can be evicted by Chrome under storage pressure.
- The demo summarizes customer reviews, producing a summary about sound quality and battery life.
- These APIs are experimental and need Chrome flags; on-device internals let you inspect tokens, events, and settings.
- Proofreader API follows the same lifecycle: check support, create a proofreader, attach a download monitor, set the language.
- It returns corrected text plus individual corrections with start and end positions, so a UI can show edits in place.

### Structured reviews from images · 30:17
- The general-purpose Prompt API accepts text and images combined in one `session.prompt` message.
- A JSON Schema (`title` and `description`, both strings) constrains the output so the app can use it directly.
- Uploading a headphone photo produced review text claiming heavy damage and a disappointed recipient.
- That is the model's interpretation, not verified evidence — a fair warning about trusting generated content.
- Inference ran locally, but still depends on the model download, device resources, browser support, and configuration flags.

### Agent-readable docs: llms.txt · 36:02
- Websites may now need to be legible to software that is looking for information before it acts.
- `llms.txt` is proposed by analogy with `robots.txt` and sitemaps: a concise, structured map of useful material.
- An Angular example lists Markdown links to relevant documentation, such as pages about animation.
- `llms-full.txt` gathers the underlying docs into a single text resource.
- Why it matters: a model's training data lags framework releases, so it reaches for outdated patterns or old AngularJS examples.

### WebMCP: expose app actions as tools · 40:14
- Without explicit actions, browser agents inspect screenshots or the DOM and imitate human clicks.
- WebMCP (highly experimental) lets a page expose its own business actions as structured tools.
- It differs from Chrome DevTools MCP: that gives developers inspection tools, while WebMCP gives user-facing operations.
- Seine's Add to Cart already calls `addCart`; Leplus wraps it in a tool with item name and quantity arguments.
- The handler loops once per unit because `addCart` accepts only one item at a time.
- The tool is registered on the page's `navigator`; an extension discovers it. Five water bottles were added through the tool.
- Forms can become tools too: annotate the review form, and its inputs become schema fields (rating 1–5, title, text, photo).
- Filling a form is not submitting it; an autosubmit annotation makes the tool fill and submit in one step.
- The analogy: like responsive design, sites may need to accommodate agentic browsers. No stable standard exists yet.

## Resources

- [Seine demo source](https://github.com/sinedied/web-ai-talk/tree/main/seine)
- [Agent skills used in the Seine demo](https://github.com/sinedied/web-ai-talk/tree/main/seine/.agents/skills)
- [Chrome DevTools MCP](https://github.com/ChromeDevTools/chrome-devtools-mcp)
- [Chrome Summarizer API](https://developer.chrome.com/docs/ai/summarizer-api)
- [Chrome Proofreader API](https://developer.chrome.com/docs/ai/proofreader-api)
- [Chrome Prompt API](https://developer.chrome.com/docs/ai/prompt-api)
- [`llms.txt` proposal](https://llmstxt.org/)
- [WebMCP proposal](https://github.com/webmachinelearning/webmcp)
