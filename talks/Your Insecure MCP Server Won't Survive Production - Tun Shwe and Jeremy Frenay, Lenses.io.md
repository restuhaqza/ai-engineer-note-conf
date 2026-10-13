# Your Insecure MCP Server Won't Survive Production

| Field | Details |
| --- | --- |
| **Speaker** | Tun Shwe, Jeremy Frenay |
| **Affiliation** | Lenses.io |
| **Type** | Talk |
| **Date** | 2026-04-08 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=BurJvbqFr4c) |
| **Tags** | `safety-and-governance` `rag-context-and-search` `evals` `apis-mcp-and-protocols` |

## Summary

Many teams now connect agents to internal systems through MCP (Model Context Protocol, a standard way for agents to call tools). Tun Shwe and Jeremy Frenay from Lenses.io argue that most MCP servers are insecure for a simple reason: they are designed badly, and bad design and bad security make each other worse.

Their core claim is that you must get the interface right before writing any auth code. Expose few tools, constrain inputs at the schema level, write precise tool descriptions, return only the data the task needs, and scope permissions to individual tools and resources. No amount of OAuth fixes an interface that hands an agent a delete button it never needed.

They then walk from local development to production. Local MCP runs over standard I/O as a single-user process with no network exposure. Going remote with Streamable HTTP is a sudden "security cliff": now you need OAuth, token lifecycle management, TLS, CORS, and rate limiting all at once.

Frenay explains the OAuth path step by step — dynamic client registration, PKCE, and token exchange — and the newer CIMD approach, where a client's identity is a URL it controls. He ends on governance: role-based access at the tool level, masking sensitive data, and tracing the whole request end to end.

## Key Takeaways

- **Agents read your tool descriptions and choose operations.** Design the interface for that.
- **Poor interface design and poor security compound each other.**
- **Expose one coarse-grained tool per outcome**, not many fine-grained operations.
- **Constrain inputs** with top-level primitives and enums; avoid freeform nested payloads.
- **Tool descriptions are an attack surface.** Tool poisoning hides instructions the UI may not show.
- **Return the minimum data.** Unfiltered PII in context is one injection away from leaking.
- **Local standard I/O needs no auth; remote HTTP is a security cliff** — OAuth, tokens, TLS, CORS, rate limits, all at once.
- **Long-lived, unscoped API keys still dominate.** Short-lived scoped tokens via OAuth 2.1 are the direction.

## Detailed Notes

### What should an agent be allowed to see? · 00:08
- Lenses.io builds a streaming-data layer around Apache Kafka that gives agents real-time context.
- Their open-source MCP server applies those field learnings.
- Starting point from FastMCP creator Jeremiah Lowin: agents deserve interfaces designed for their own use cases.
- Shwe's extension: poor interface design and poor security reinforce each other.

### Discovery, iteration, and context as attack surfaces · 02:24
- **Discovery:** a developer scans API docs once; an agent enumerates every tool and reads every description on connect.
- More tools means more tokens and more text that could carry hidden, poisoned instructions.
- **Iteration:** an agent retry resends the whole conversation history, including sensitive results from earlier tool calls.
- **Context:** a limited window (about 200k tokens in the example) that can fill with PII and internal details.
- OWASP's MCP Top 10 lists context injection and oversharing at number 10.
- Answer: curate — expose the fewest tools and the least information. Less is more.

### Five rules for secure MCP design · 04:43
- Good MCP design and good MCP security are the same discipline; OAuth cannot rescue a bad interface.
- **1. Shrink the surface by design.** One coarse tool per outcome (check an order), not separate delete-user tools.
- **2. Constrain inputs at the schema.** Prefer top-level primitives and enums; reject freeform nested payloads.
- Use a typing library (for example, Pydantic) to stop unconstrained strings reaching shells or query engines.
- **3. Treat documentation as defense.** Clear, complete descriptions crowd out a poisoned neighboring server's influence.
- **4. Return only what the task needs.** Don't dump PII, credentials, or system details into context.
- **5. Minimize the blast radius.** Scope permissions per tool/resource, not per session; use resources for read-only access.
- Note that MCP annotations like `readOnlyHint` are hints, not authorization — enforce read-only yourself.

### Crossing the deployment security cliff · 08:11
- Local standard I/O: the host talks directly to a local process; single user, no network endpoint, no auth needed.
- Streamable HTTP makes MCP a shared service: many clients, horizontal scaling, centralized governance.
- Remote adds OAuth, token lifecycle, TLS, CORS, and rate limiting — you need them all at once, with no gradual ramp.
- A cited Stacklok benchmark: 20 of 22 requests failed at 20 concurrent connections on standard I/O.

### What API-key plumbing leaves unresolved · 10:10
- Frenay counts more than ten specifications across core OAuth flows, discovery/metadata, and token lifecycle.
- Typical setup: an API key in client config, passed via environment variable to the local server and on to the upstream API.
- Such keys are long-lived, rarely rotated, and not scoped to specific actions.
- Moving to HTTP just changes where the key travels; its lifecycle stays the same.
- Two problematic upstream patterns: pass the token through (no real validation), or map many callers to one shared key.
- Frenay estimates more than 50% of MCP servers still use these long-lived, unscoped credentials.

### From client registration to OAuth sign-in · 13:47
- Traditional OAuth assumes known clients; MCP lets any client discover any server at runtime, so pre-registration breaks down.
- **Dynamic Client Registration (DCR)** lets a client register itself and receive a fresh client ID.
- Discovery: request `/mcp` → `401` → follow `WWW-Authenticate` → read resource metadata → find the authorization server.
- Then `POST /register` to obtain a client ID.
- Authorization uses **PKCE** (Proof Key for Code Exchange): the client keeps a code verifier and sends a derived challenge.
- No session yet → redirect to an identity provider for login → consent screen → authorization code → `/token` → access token (a JWT here).
- The MCP server validates the token's audience and scopes, then exchanges it for an upstream token under RFC 8693.

### Two ways to identify a client: DCR and CIMD · 18:55
- DCR problems: a new registration per connection (not portable), an open `/register` endpoint, and self-asserted identity.
- A malicious client can claim to be Claude; a registration ID proves nothing about who it is.
- **CIMD (Client ID Metadata Documents):** the owner publishes a metadata document at a public URL, and that URL becomes the client ID.
- The flow drops the `/register` step and goes straight to `/authorize` with a URL-valued client ID.
- The authorization server fetches the document, binds allowed redirect URIs, and can allow or deny clients.
- Since November 2025 the spec says implementations SHOULD support CIMD and MAY support DCR — preferred, not mandatory.
- CIMD still needs SSRF defenses and does not fix localhost callback impersonation.

### Govern and trace the operation after authorization · 22:40
- Auth is not the finish line: use role-based access control at the individual tool and resource level.
- Deciding "may this caller read this order" matters more than "is there a session."
- Mask PII (email, phone, national insurance numbers) before the agent sees it; omit unnecessary fields entirely.
- Log which agent called which tool, with which parameters, and what came back (the EU AI Act is cited as motivation).
- Trace the full request: client call, validation, tool execution, data retrieval, and response.
- If you cannot reconstruct what an agent did end to end, you cannot govern it.

## Resources

- [Lenses MCP Server](https://github.com/lensesio/lenses-mcp)
- [OWASP MCP Top 10](https://owasp.org/www-project-mcp-top-10/)
- [MCP authorization specification (November 2025)](https://modelcontextprotocol.io/specification/2025-11-25/basic/authorization)
- [MCP Security Best Practices](https://modelcontextprotocol.io/docs/2025-11-25/tutorials/security/security_best_practices)
- [OAuth 2.0 Token Exchange — RFC 8693](https://www.rfc-editor.org/rfc/rfc8693.html)
- [MCP server performance: Transport protocol matters (Stacklok)](https://stacklok.com/blog/mcp-server-performance-transport-protocol-matters/)
