# Why MCP and ChatGPT Apps Use Two Iframes

| Field | Details |
| --- | --- |
| **Speaker** | Frédéric Barthelet |
| **Affiliation** | Alpic |
| **Type** | Talk |
| **Date** | 2026-06-15 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=c-2eEv2ou7Y) |
| **Tags** | `safety-and-governance` `apis-mcp-and-protocols` |

## Summary

ChatGPT apps and MCP apps can show an interactive interface right inside a conversation — a small HTML page with its own JavaScript and CSS. The hard part is not drawing that UI. It is letting one company's app run inside another company's product without giving the app access to the host's private data.

Frédéric Barthelet, CTO and co-founder of the MCP hosting company Alpic, walks through the browser-security problem behind this. He explains MCP (a standard way for agents to call tools and resources) and the app extensions that let a tool ship an HTML view. Then he inspects ChatGPT's page and finds the view is not rendered in one iframe but in two iframes nested inside each other.

The rest of the talk explains why. Injecting the app's HTML directly (via `srcdoc`) gives the app the host's origin, which either blocks its scripts or exposes the host's stored data. Sandboxing the iframe fixes that, but removes the localStorage, cookies, and similar storage the app may need. Hosting each app at its own URL works, but forces the host to allowlist every app's domain.

Barthelet's answer is the double iframe: a small reusable loader runs on an app-specific subdomain and writes the app's HTML into an inner frame. That keeps the host's origin separate from the app's while giving each app its own storage. He closes with the work this creates for app developers — declaring every domain the view touches — and shows Alpic's open-source Skybridge tooling that flags missing domains before submission.

## Key Takeaways

- **Two goals combine in these apps:** discovery (finding an app in a store or chat) and interactive UI (returning an interface, not just text).
- **The core problem is origin isolation.** An app must render inside the host without inheriting the host's cookies or storage.
- **The UI unit is a "view":** a plain HTML document tied to a specific tool through metadata.
- **Injecting HTML directly fails.** `srcdoc` shares the host's origin and security policy, so the app's scripts are blocked — and loosening the policy lets the app read host data.
- **Sandboxing over-corrects.** It removes host access but also kills localStorage, IndexedDB, and cookies, which need a real origin.
- **The double iframe gives each app its own origin.** A shared loader runs on an app-specific subdomain; the app HTML lives inside it.
- **Apps must declare their domains.** External APIs, scripts, images, and nested frames all have to be listed in metadata, or the browser's policy will block them.
- **Missing domains are a common launch failure.** Alpic reports app-store rejections and production breakage caused by undeclared policy domains.

## Detailed Notes

### Why an app needs a place to run · 00:16
- MCP and ChatGPT apps do two things: make a service **discoverable** (in a store, or pulled into a relevant chat) and add an **interactive UI** where the agent used to return only text.
- The view may come from an MCP server or be generated on the fly; either way the host has to render it safely.
- He traces the idea from early MCP UI work to OpenAI's Apps SDK preview (October 2025), then to the shared **MCP Apps extension** that multiple clients can support.
- Alpic builds MCP hosting, so its apps need to run inside someone else's product — exactly this problem.

### From a tool call to an HTML view · 02:02
- A **view** is a snippet of HTML that can include JavaScript and CSS. It is always shown as the result of a tool call.
- A tool can carry metadata saying "display my results with this view." A supporting host then uses that view when the tool runs.
- Lifecycle: during `tools/list` the host discovers which tools advertise views, fetches the relevant HTML (early for caching, or on demand), then creates the frame and feeds in the tool's data.
- Preloading is allowed too — the view can exist and initialize while the tool is still running, and inputs and results can arrive as notifications.

### The unexpected second iframe · 03:31
- While inspecting ChatGPT's page, Barthelet found an iframe nested inside another iframe; the same pattern appears in Claude.
- Before any third-party UI appears, the host already has a security policy: **Content Security Policy (CSP)** — rules, sent as response headers, that tell the browser what a page may load and run.
- Two directives matter most here: `script-src` (which scripts may execute) and `frame-src` (which sources may load in nested frames).
- ChatGPT's `script-src` uses a per-request nonce (a one-time token), so an app's own bundled JavaScript has no authorization and will not run.

### Why srcdoc and sandbox both fail · 05:29
- An iframe can receive a document two ways: `src` (the browser loads a URL) or `srcdoc` (the caller supplies the HTML directly). Since the MCP server already exposes HTML, `srcdoc` looks like the shortest path.
- The catch: a `srcdoc` frame shares the parent's origin *and* its CSP, so the app's scripts are blocked by the nonce rule.
- Loosening the CSP makes a worse problem: now the app's scripts run with the host's origin and can read the host's `localStorage` and cookies and send them to the app's backend.
- Restoring CSP and sandboxing the frame gives it an **opaque origin** (a "null" origin). That blocks host access, but it also disables `localStorage`, IndexedDB, and cookies, since those need a real origin to store under.
- Adding `allow-same-origin` hands back a real origin — but for a `srcdoc` frame that origin is the parent's, so you are back to square one. The HTML spec warns that same-origin plus allowed scripts let the child simply remove the sandbox and reload.

### A separate URL shifts the problem · 09:45
- Alternative: expose the HTML at a normal endpoint (for example `/view`) and let the browser load it with `src`. That gives a separate origin.
- But now the host's `frame-src` must allowlist that domain — and at store scale that means every app's domain, updated whenever a new app arrives.
- A host-owned content domain (his example is `openai-usercontent.com`) makes the allowlist tractable: allow the domain once, then use an app-specific subdomain as a routing key.
- The cost: the host now runs infrastructure that serves third-party code it did not write, on its own domain — heavy for smaller MCP hosts that lack OpenAI- or Anthropic-scale resources.

### One reusable loader, separate origins per app · 12:03
- The double iframe serves one small, reusable **loader** in an outer frame on a domain separate from the chat host. The loader takes the view HTML and writes it into an inner frame with `srcdoc`.
- The inner document can share the outer frame's origin without ever sharing ChatGPT's origin. The MCP Apps spec recommends this architecture.
- Serving the same loader from one shared origin would let apps collide in origin-based storage. Instead, serve the *same bytes* from different app subdomains — so app `ABC123` cannot read app `ABC456`'s `localStorage`.
- Layer map: the chat host keeps its own origin and policy; the outer iframe loads the common wrapper on an app-specific origin; the inner `srcdoc` frame renders the app HTML inside that boundary.
- The app gets its own CSP too, applied through a meta tag in the outer frame using declarations from the MCP metadata.
- He notes he runs this in production, and that Facebook's App Marketplace used the same pattern years earlier. Cookies need correct scoping — subdomains alone do not guarantee isolation.

### Declare every domain the view uses · 14:11
- The host can build the app's policy only if the app lists its dependencies in MCP Apps metadata: connection domains (for `connect-src`), resource domains (scripts, images), and frame domains (nested frames). `baseUriDomains` is in the set too.
- These declarations are policy inputs, not permission to load anything.
- He compares CSP debugging to getting CORS wrong years ago — it looks like a broken app until you find the missing permission.
- Practical advice: test against the policy the deployed view will actually face. He notes ChatGPT developer mode used to drop CSP entirely, hiding missing domains until production, and says a new option aims to improve that workflow.

### CSP Inspector: catch a missing domain early · 15:55
- **Skybridge** is Alpic's open-source layer on top of the official Apps SDK: end-to-end type safety between the MCP server and its views, polyfills for host-specific Claude/ChatGPT APIs, and local development tools.
- The featured demo feature is the **CSP Inspector**. The sample app is a Magic 8 Ball with 25 preset answers; a tool generates an answer and a view shows the question and response.
- Running the Skybridge server opens a browser dev environment: a tool list on the left, live view rendering, and the inspector comparing declared domains against domains the view actually calls.
- Repair loop shown: inspector is green → he adds a fetch to an IP-location API → the inspector flags the new domain as missing → he adds it to metadata and reloads → green again.
- Why it matters: he has seen app-store rejections and production failures from missing CSP domains. The inspector surfaces the exact missing domain while you edit, rather than after launch.

## Resources

- [Skybridge](https://github.com/alpic-ai/skybridge) — Alpic's React and TypeScript framework for MCP apps
- [MCP Apps overview](https://modelcontextprotocol.io/extensions/apps/overview) — how tools advertise interactive HTML views
- [MCP Apps specification (January 2026)](https://github.com/modelcontextprotocol/ext-apps/blob/main/specification/2026-01-26/apps.mdx) — sandbox proxies and CSP details
- [HTML iframe sandbox semantics](https://html.spec.whatwg.org/dev/iframe-embed-object.html) — the standard's note on escaping the sandbox
- [Apps SDK launch announcement](https://openai.com/index/introducing-apps-in-chatgpt/) — OpenAI's October 2025 preview
