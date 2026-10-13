# Claws Out: Securing and Building with OpenClaw

| Field | Details |
| --- | --- |
| **Speaker** | Nick Taylor |
| **Affiliation** | Pomerium |
| **Type** | Talk |
| **Date** | 2026-07-11 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=xg1zNlzw7Jk) |
| **Tags** | `safety-and-governance` `apis-mcp-and-protocols` `developer-workflows-and-testing` |

## Summary

Nick Taylor, a developer advocate at Pomerium, describes a friction he hit with OpenClaw. Even after putting an identity-aware proxy in front of it, he still had to paste a WebSocket token and re-pair devices. So he opened an issue and contributed trusted-proxy authentication, which lets an explicitly trusted proxy vouch for the connecting user instead.

The change separates two decisions: which proxy is trusted, and which headers carry identity. Configuration selects trusted-proxy mode, lists the trusted proxy addresses, and names the user header plus required headers — a JWT-bearing header (a signed identity token) in his setup. The practical result was no separate WebSocket token, no repeated pairing, and a smaller attack surface.

He then shows a live development loop. He edits an MCP app's files (MCP is a standard way for agents to call tools and apps) through his own OpenClaw instance ("McClaw") over Discord, while a public, gated URL exposes the app inside ChatGPT. Because MCP Apps render React widgets served by Vite, a button can even post a message back into the host conversation. He closes with a caution: exposing services and granting agent credentials still need deliberate limits.

## Key Takeaways

- **Trusted-proxy auth removes duplicate login work.** An identity-aware proxy combines an identity provider, a policy engine, and a reverse proxy.
- **Separate the two decisions:** which proxy is trusted, and which headers carry identity.
- **Trusting a header is not validating a signature.** Assuming a header came from an authorised proxy is not the same as checking a JWT yourself.
- **Test the first-use path.** His bug slipped through because he tested on an already-paired local device.
- **Control plane security is not agent permission.** His agent still pushed a PR (pull request) early because the GitHub CLI had full access.
- **MCP Apps add UI to MCP.** React widgets served by Vite can hot-reload into ChatGPT.
- **Gate every service.** Protecting the gateway does not automatically protect the MCP endpoint the agent starts.
- **`sendMessage` hands control back.** A widget can post a user prompt that the host assistant continues.

## Detailed Notes

### The friction: authenticating behind a proxy · 00:15
- Taylor works at Pomerium and is a long-time open-source contributor (GitHub Star, Microsoft MVP, AWS Community Builder).
- He secured OpenClaw with an identity-aware proxy, yet still had to add a token for the WebSocket connection.
- An **identity-aware proxy** has three parts: an identity provider (who you are), a policy engine (whether you may enter), and a reverse proxy (the gate). Google Cloud's IAP is his example.
- The token sat in a query string — reasonable for local use, awkward on a phone.
- Device pairing added another step, sometimes forcing him back to another device to complete it.
- He filed issue #1560; another user running Caddy agreed, maintainer Peter defined the feature criteria, and Taylor built it.

### Make the trust boundary explicit · 03:59
- The config separates **which proxy is trusted** from **which headers carry identity**.
- `gateway.auth.mode` is set to `trusted-proxy`.
- `gateway.trustedProxies` lists one or more addresses for the proxy that gates access to the gateway.
- `gateway.auth.trustedProxy` holds the identity-header settings: `userHeader`, `requiredHeaders`, and optional `allowUsers`.
- His example uses an email header and a JWT-assertion header; he omits `allowUsers` because Pomerium's policies already decide who may enter.
- Key distinction: a header that arrived from a trusted proxy is an assumption; checking a JWT signature yourself is a separate operation.
- Clients must not be able to bypass the proxy and impersonate that trusted source, so the proxy must control the forwarded headers.
- Practical result: no separate WebSocket token and no repeated pairing. Setup is available through onboarding or the terminal UI.

### The bug was a first-use path · 05:27
- His first version missed a pairing bug because he tested locally on a device that was already paired.
- Another user (Anthony) reported it, and Sid fixed it — **existing session state hid a broken first-use path**.
- The lesson for authentication changes: a familiar browser does not exercise the same conditions as a new device arriving without prior approval.
- The repository moved fast: he started at issue #1560 and PR #1710, went on vacation, and returned to numbering from about 1,500 to nearly 16,000.
- His original PR was closed as stale, and merging the eventual contribution required substantial rebasing.

### McClaw, Discord, and a workspace without SSH · 06:51
- His own instance, **McClaw**, sits on his desk in Montreal; he mostly talks to it through Discord, with WhatsApp also configured.
- He left Telegram after his CEO raised encryption concerns. The precise point: Telegram Cloud Chats use client-server encryption, while Secret Chats are end-to-end, so ordinary chats are not simply plaintext. Discord here is a workflow preference, not a claim of stronger confidentiality.
- McClaw helped implement the trusted-proxy contribution — and also pushed a PR early, because Taylor had given the GitHub CLI full access. He returned it to draft.
- Securing an agent's control plane does not decide what the agent may do with credentials once it is running.
- After the feature merged, he built **Clawspace** so he could browse, read, and edit workspace files without opening an SSH session.

### A live workspace inside ChatGPT · 08:43
- The demo starts from a template for an MCP app — an MCP server that also returns UI.
- The AI Engineer website exposes developer entry points including `llms.txt` and an MCP server, which becomes the demo subject.
- Sequence: create the application in ChatGPT, configure OAuth (a standard sign-in handoff), register the MCP server, and connect.
- The server exposes two tools: `echo` and `searchSpeakers`.
- Invoking "Echo hello" returns an interactive widget (**MCP Apps** add this UI extension to MCP); it shows the text and offers action buttons.
- Taylor asks McClaw to change the echoed message to "AIE EU". Underneath, **React** renders the widget, **Vite** serves it, and hot module reloading carries the workspace edit into the running interface.
- Two boundaries make it work: trusted-proxy auth governs OpenClaw's gateway, and a separate Pomerium gate protects a public URL for the MCP service.

### From speaker records to a conversational button · 12:01
- In a new ChatGPT chat he selects `@AIE` and invokes `searchSpeakers`.
- He asks McClaw to use the conference's `speakers.json` as the speaker source and match the Echo widget's visual style, so the directory can filter and show presenters.
- The agent pauses noticeably to finish — a reminder that you never know exactly when agentic work ends.
- He restates the loop: he types in Discord, OpenClaw edits workspace files, and those files power the publicly reachable, gated app. He also uses Claude and Codex; this is one workflow among several.
- When done, the interface supports filtering, selecting a speaker, and viewing more information.
- Next feature: a **More** button using `sendMessage`. Instead of making a model call inside the widget, it sends a user message into the chat that hosts the app.
- A short TypeScript handler calls `app.sendMessage` with a user-role text prompt; the connected app instance comes from the button's click handler.
- Clicking it puts ChatGPT into its thinking state and appends a new prompt asking for more about a talk — the widget contributes the message, and ChatGPT continues the conversation.

### Expose the service, keep the boundary · 15:15
- Pomerium is his choice; Caddy with OAuth is an alternative. He briefly mentions NGINX, then narrows his deprecation remark to the Kubernetes Ingress NGINX controller (maintenance ending March 2026), not NGINX generally.
- The architectural requirement: make a local service reachable by an external host while keeping an access gate.
- That changes where development can happen — he now routinely builds through chat on his phone, and Clawspace is his example of a tool made for one person.
- The freedom depends on limits: he warns about accidental public exposure and destructive agent actions, including reports of deleted email.
- Trusted-proxy auth makes controlled access less cumbersome; it does not replace careful decisions about which services are exposed and which credentials an agent receives.

## Resources

- [The trusted-proxy authentication contribution](https://github.com/openclaw/openclaw/pull/15940) — the merged contribution introducing trusted-proxy auth (mentioned).
- [Configure trusted-proxy authentication](https://docs.openclaw.ai/gateway/trusted-proxy-auth) — current configuration and security requirements (update).
- [Clawspace](https://github.com/nickytonline/clawspace) — a browser-based OpenClaw workspace file explorer and editor (mentioned).
- [MCP app TypeScript template](https://github.com/pomerium/mcp-app-typescript-template) — starter with an MCP server, React widgets, and an Echo tool (related).
- [MCP Apps overview](https://modelcontextprotocol.io/extensions/apps/overview) — the official entry point for the MCP Apps extension (related).
- [Telegram encryption FAQ](https://telegram.org/faq) — the distinction between encrypted Cloud Chats and end-to-end Secret Chats (related).
- [Ingress NGINX retirement announcement](https://kubernetes.io/blog/2025/11/11/ingress-nginx-retirement/) — the community Ingress NGINX controller's maintenance end (related).
