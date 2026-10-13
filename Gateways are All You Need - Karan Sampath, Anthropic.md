# Gateways are All You Need

| Field | Details |
| --- | --- |
| **Speaker** | Karan Sampath |
| **Affiliation** | Anthropic |
| **Type** | Talk |
| **Date** | 2026-04-27 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=CD6R4Wf3jnY) |
| **Tags** | `apis-mcp-and-protocols` `agent-engineering` `coding-and-developer-tools` |

## Summary

Karan Sampath, a product engineer at Anthropic, looks at why enterprises struggle
to adopt MCP (Model Context Protocol — an open standard for connecting agents to
tools and data). Finding an MCP server is no longer the problem: the official
registry holds thousands of them. The gap is everything that comes *after*
discovery — observability, access control, security, and deployment.

Companies now build servers easily, partly thanks to coding agents. But those
servers then can't get deployed, or can't get the access their tools need. Security
teams face a growing approval queue with too little visibility, so the whole
company ends up with only a handful of usable tools. Sampath frames these three
concerns — observability, access control, and security — as a three-headed problem
that APIs solved long ago but MCP deployments have not made routine.

His answer is a shared **MCP gateway**. Rather than approving each server as a
separate project, security teams approve one trusted platform (a **root of trust**)
within which teams build their own servers. The gateway provides shared
authentication, role-based access control, routing, secure connectivity, an
internal registry, and developer tooling. Teams then focus only on their domain
logic — a legal team, for example, decides what contract review and redlining
should do, not how callers are identified or how requests scale.

The larger goal is architectural: separate the **agent harness** (the runtime that
runs an agent) from where enterprise data lives. New agent surfaces can come and go
while the gateway stays put. Sampath's closing advice: invest in common
infrastructure, let teams build their own servers, and use the gateway to establish
a trust boundary.

## Key Takeaways

- **Discovery is solved; operations are not.** A registry helps you find a server but doesn't provide auth, access control, or observability.
- **Permissions must go deeper than the server.** You often need to distinguish tools and actions within one server, not just grant access to the whole thing.
- **Approve a platform, not every server.** A shared root of trust lets teams iterate without repeated security reviews.
- **A gateway centralizes the shared plumbing:** authentication, role-based access, routing, tunneling, an internal registry, and tooling.
- **Agents need real identities.** As agents take on roles, they'll need explicit identities and scopes, expressed in one access-control surface.
- **Observability should improve tools, not just count calls.** Usage data should reveal which tools important workflows depend on.
- **Separate the harness from the data layer.** Agent deployment choices shouldn't force changes to how enterprise data and servers are organized.
- **Credential handling must respect boundaries.** Don't blindly forward tokens, per MCP security guidance.

## Detailed Notes

### Finding a server is not enough · 00:15
- MCP is an open standard (originating at Anthropic) with an official registry that has grown to thousands of servers.
- Enterprises still struggle with what they consider table stakes: **observability**, **access control**, and **security**.
- Observability questions: who uses these tools, how they are used, and which parts are failing?
- Access control example: the whole company may need to *view* an observability server, but only some people should change systems or dashboards.
- That means **permissions must distinguish tools and actions inside a server**, not merely grant access to it.
- Security has two sides: is the server safe and free of exfiltration risk, and can an untrusted remote client reach private enterprise data?
- Sampath's diagnosis: enterprises solved these in the API era, but their MCP deployments haven't made them routine.

### When server creation outruns deployment · 03:20
- A registry is useful but incomplete for an enterprise — it doesn't supply authentication, access control, observability, or credential management.
- Coding agents like Claude Code make it easy for individual teams to build servers, including tool definitions and access-control structures.
- The stall happens next: the server can't be deployed, or its tools can't get the access they need.
- Security teams face a growing approval queue without enough visibility to decide efficiently.
- Leadership then sees "ineffective agents," even though the integrations were already built.
- The bottleneck is organizational as much as technical: reviewing every server as a separate project overloads security and limits builders.
- Restricting the toolset restricts what *every* agent can do, so fixing these frictions matters beyond one integration.

### Approve the platform, then decentralize the tools · 05:34
- Once teams can build servers, the scarce resource becomes a **trusted path to deployment**.
- The key idea: security teams should **bless one platform** within which teams can build — a **root of trust**.
- Sampath reports this expanded MCP adoption in both internal and external work, since a useful server is reusable by many agents (a qualitative account, not a measured growth curve).
- A **gateway** sits between MCP clients and a potentially large set of MCP servers.
- It supplies shared **authentication, authorization, observability, secure connectivity, and hosting/deployment** support.
- Example: a legal team worries only about contract review, redlining, and escalation — not about who accesses the server, how it scales, or how to add agents.
- Crucially, the domain team can now build the server itself instead of handing every change to a separate technical team.

### What belongs inside the gateway · 08:28
- There's no single mandatory design; Sampath's component list is illustrative, not exhaustive.
- **Auth Handler** — authenticate callers.
- **RBAC Engine** — apply role-based access control (permissions tied to roles).
- **Proxy Router** — route requests to internal servers.
- **Tunnel Handler** — provide secured connectivity.
- **Subregistry** — catalog internal MCP servers.
- **Tooling** — help builders create and integrate servers (for example, a gateway CLI).
- The routing boundary matters: clients see only the gateway, and internal servers treat the gateway as their only trusted endpoint.
- The gateway CLI lets teams (and their coding agents) create a server, wire it to the shared services, and focus on business logic.
- Low maintenance is an aspiration, not an automatic property of adding a gateway; the durable benefit is one place to improve shared infrastructure.

### Centralize identity, policy, and tool feedback · 10:00
- The gateway connects to the enterprise's existing identity provider (**IdP**) and can support delegated identity for users and agents.
- Sampath expects agents to need more explicit identities and scopes as their roles develop.
- Result: one access-control surface for all agents and MCP servers, with policies scoped to teams or individual users.
- A small TypeScript example shows central policy: `legal-reader` may `review-contract` but not `redline-contract`; `legal-reviewer` may do all three.
- These are **permission decisions**, not executed operations; roles must come from authenticated identity, and the gateway enforces the decision before routing.
- Central policy means access can change without embedding a separate permission system in every workflow.
- **Observability** should be more than call counts: it should show which servers are active and which tools important workflows depend on, guiding where to invest.

### Add clients and iterate within shared standards · 11:26
- With a gateway, new client surfaces (Sampath names Claude.ai, Claude Code, and Claude Cowork) connect to the same shared layer.
- Contrast: an awkward estate of 40 servers configured unevenly for different clients. A common entry point removes repeated integration work.
- Sensitive data is the next benefit: internal servers may expose years of valuable work, so a gateway concentrates investment in encrypted connectivity and a defined trust boundary.
- Encryption protects transport but doesn't stop an authorized or compromised client from misusing data it receives.
- Fast iteration is the practical payoff: the legal team can revise its workflow without resubmitting the same infrastructure for security review each time.
- Shared infrastructure also encodes enterprise standards — common primitives reflecting expected procedures, permitted tools, and prohibited behavior.
- **Credential flexibility:** a server might start accepting one auth type but need user-, team-, company-, or service-account credentials later; the gateway can help select and substitute them.
- This must not become blind token forwarding — MCP security guidance forbids token passthrough.
- Scaling illustration (hypothetical): 40 servers serving populations growing from tens to hundreds of thousands of agents; the gateway is a common place to accept and distribute requests.
- Routes to the infrastructure: open-source, providers, or in-house — the common requirement is the trusted platform boundary, not a particular vendor.

### Keep the agent harness separate from the data layer · 15:11
- The longer-term purpose: **separate the agent harness from where enterprise data lives**.
- As agents multiply and new execution surfaces appear, their deployment choices shouldn't force changes to the organization's data structure or server organization.
- Conversely, the data layer shouldn't be designed around one particular harness; a stable gateway interface lets both sides evolve.
- Example: both **Claude Managed Agents** (hosted execution) and an internal **Claude Agent SDK** application (enterprise-operated) can connect through the same gateway to enterprise MCP servers.
- The enterprise can decide which agents run inside and which run outside without rebuilding the gateway layer.
- Shared connectivity doesn't mean identical client auth behavior — in the current Agent SDK, the application handles interactive OAuth itself.
- The shared infrastructure should let teams build their own servers, not reproduce the platform plumbing.
- Summary of the three main points: invest in common infrastructure, use gateways for secured connections and a root of trust, and move toward separating the agent harness from the data layer.

## Resources

- [Introducing the MCP Registry](https://blog.modelcontextprotocol.io/posts/2025-09-08-mcp-registry-preview/) — official registry announcement for server discovery.
- [MCP Security Best Practices](https://modelcontextprotocol.io/docs/2025-11-25/tutorials/security/security_best_practices) — guidance for proxies and servers, including consent and token passthrough.
- [Introducing Claude Managed Agents](https://claude.com/blog/claude-managed-agents) — Anthropic's hosted agent execution announcement.
- "Decoupling the brain from the hands" — Anthropic's explanation of how Managed Agents separates the harness from the data.
- [Connect Claude Agent SDK to MCP tools](https://code.claude.com/docs/en/agent-sdk/mcp) — configuration examples for MCP servers, permissions, and OAuth.
