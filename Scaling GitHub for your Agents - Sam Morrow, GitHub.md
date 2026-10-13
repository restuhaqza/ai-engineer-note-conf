# Scaling GitHub for your Agents

| Field | Details |
| --- | --- |
| **Speaker** | Sam Morrow |
| **Affiliation** | GitHub |
| **Type** | Talk |
| **Date** | 2026-04-27 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=0n3MKk7r60w) |
| **Tags** | `enterprise` `evals` `safety-and-governance` `rag-context-and-search` `apis-mcp-and-protocols` |

## Summary

Sam Morrow leads development of GitHub's MCP server (MCP is a standard way for AI agents to call tools and read data). GitHub open-sourced its local server in April 2025, and it quickly became one of the platform's most-starred repositories. Contributions poured in, filling gaps in coverage — and then something went wrong.

After roughly a month of new features, agents were getting *worse* at using GitHub and burning through their context windows (the limited working memory a model holds) faster. The tool count had climbed past 100. Morrow's core point: stuffing more tools into context does not make a better agent, because all those definitions compete for attention before any real work starts.

The talk is a walk through GitHub's fixes. They made tools configurable, but most users kept the default settings, so the options never reached them. They shrank the *default* tool load by about half, trimmed the data tools return, wrapped multi-step operations into single intent-driven calls, and let the caller's permissions decide which tools appear. Along the way they hit security limits: easy setup leads to plaintext, long-lived tokens, and prompt injection (hostile instructions hidden in content the agent reads) remains unsolved.

Morrow frames all of this as work-in-progress on an evolving product, with numbers that were already shifting as he spoke. He closes with a small invitation: wrap a server's read-only tools in a CLI and watch how an agent copes.

## Key Takeaways

- **More tools can mean a worse agent.** Definitions loaded into context compete for attention before useful work begins.
- **Defaults beat configuration.** Most users never edit the JSON, so improving defaults reaches far more people than adding options.
- **Cut context twice.** Reduce what loads up front (GitHub reported ~49% less initial load) and what comes back per call (over 75% fewer output tokens in one example).
- **Tools should express intent, not mirror API endpoints.** One call can hide five internal API calls and save round trips and context.
- **Evaluate tools against each other.** A good description earns the right calls *and* avoids the wrong ones — it should not win every selection.
- **Secure auth is hard to make easy.** Plaintext, overprivileged, long-lived tokens are common; OAuth 2.1 with PKCE raises the bar.
- **Let authorization prune the tool list.** Personal tokens filter by scope, OAuth can request extra scopes on demand, and server tokens should hide user-specific tools.
- **Human review still matters.** Experiments like MCP Apps let a person edit an AI-drafted issue before it becomes their public contribution.

## Detailed Notes

### More tools made agents worse · 00:33
- GitHub's MCP server launched publicly in April 2025 and was the week's most-starred repo; the team had just reached its first anniversary.
- A surge of outside contributions filled coverage gaps across repos, issues, pull requests, Actions, and Projects.
- After about a month, agents were becoming less effective and exhausting context sooner; the tool count peaked above 100.
- Morrow points to LangChain's February research, which matched the problems his team was seeing.
- His distinction: it is *more tools pushed directly into context* that hurts, not more capability in the platform.
- The hard constraint: users still needed their own subset of GitHub, so tools could not simply be deleted.

### Configuration loses to defaults · 03:11
- First fix: **toolsets** that group related product tools so users can pick what they want.
- Also added dynamic discovery (agents enable groups in chunks) and, never released, a RAG-style semantic search for finding tools.
- All of it required editing a little JSON — and most people never did, so context bloat stayed.
- Several MCP spec proposals for grouping tools had been rejected, leaving server-specific modes to fill gaps in client support.
- Read-only mode mapped to the MCP `readOnlyHint` annotation and was used by roughly 17% of users, but no client surfaced it as a filter; some gateways now do.

### Cut context on both sides · 05:06
- Because defaults need no setup, GitHub studied real remote-server usage and focused the initial tool surface on common workflows.
- Morrow reports about a **49% reduction** in initial context load from that usage-driven change.
- Grouping create/read/update/delete operations cut it further, landing near **40 tools** by default — still expandable or shrinkable.
- Tool definitions are only half the story; every returned result also consumes context.
- Tailoring the fields returned by `list_pull_requests` removed **more than 75%** of its output tokens.
- These numbers describe an evolving implementation, not fixed costs; the server can differ a lot from a user's last experience.

### Make tools express intent, then test them together · 06:41
- Tool-call success was roughly **above 95%** at the time, but agents still hallucinated or attempted writes they lacked permission for.
- Morrow argues a tool should represent an **agent's intent**, not a single API endpoint.
- If an operation needs five API calls to be robust, the server can do all five behind one tool call, avoiding repeated round trips and intermediate reasoning.
- GitHub began running evals the previous year (details live in a colleague's companion article).
- Design lesson: test descriptions *against each other*. A description that always wins selection is bad, and so is one that never attracts a call.
- Evaluation must check both the right calls and the right non-calls across the competing tool surface.

### Security: make safe auth easy, but injection remains · 08:14
- Morrow sees plaintext access tokens in the wild, stored where agents can reach them — often long-lived and overprivileged.
- He does not treat this as a deliberate user choice: easy config and secure storage are genuinely hard to combine. Clients need system keyrings or encrypted storage, as VS Code uses.
- Remote HTTP (supported from April 2025) let people connect without a local runtime; GitHub's remote server supports OAuth 2.1.
- His team helped add **PKCE** (Proof Key for Code Exchange, which strengthens the client login flow) to GitHub's authorization server.
- GitHub considered and **rejected dynamic client registration**, citing unbounded app growth, rate-limit bucketing, and unreliable app identity.
- Client ID metadata is MCP's emerging direction for easier login; Morrow advocated for it but made no shipping promise.
- Credentials are not the whole problem: he accepts Invariant Labs' GitHub MCP exfiltration demo as a valid attack and points to Simon Willison's "lethal trifecta" framing — data access plus the ability to act is the dangerous combination.
- Risk profiles vary widely, from air-gapped Enterprise Server instances to cloud agents holding broad token access, so there is no single posture to optimize for.

### Let authorization shape the tool list · 12:26
- Authorization can cut both exposure and wasted context without asking users to tune another file.
- **Personal access token:** GitHub MCP filters tools down to the token's scopes automatically.
- **OAuth step-up:** when a needed scope is missing, the server returns a scope challenge instead of failing; the client asks the user to grant it, and the call can continue.
- Morrow built this with VS Code, which already holds a GitHub token and wanted a clean-install path that upscopes on demand.
- **Server tokens** (for example in Actions) have no user, so user-specific tools are inappropriate; hiding them removes predictable failures and dead definitions.

| Credential | Tool behavior |
|---|---|
| Personal access token | Filter tools by existing scopes |
| OAuth | Challenge for extra scopes when needed |
| Server token | Hide tools that require a user identity |

### Per-request servers, review, and scale · 13:47
- GitHub runs a **stateless** remote deployment: serving processes, Redis for sessions, plus a standard observability and debugging stack.
- The twist is building a **fresh server instance per request** and adding only the tools that request's config, permissions, and policies allow.
- This differs from one long-lived stateful MCP process — the SDK object's lifetime is separate from the service handling traffic.
- Morrow reports about **seven million tool calls per week** with no session affinity: requests need not return to the same process.
- Sessions mainly preserve the self-reported client identity, helping GitHub see which clients people use.
- **Insiders Mode** switches on experimental feature flags; one example is MCP Apps, which lets a user edit an AI-drafted issue before posting.
- Morrow values that check because he cares how his issues are received in open source and does not want them dismissed as bot-generated submissions.

### From choosing tools to composing them · 16:30
- Morrow's forecast: automatic server discovery and **compositional** tool use, where one tool's output feeds the next like Bash piping.
- He points to Cloudflare's Code Mode and to tool-search APIs from Anthropic (now in Claude Code) and OpenAI.
- If agents can find capabilities when needed, thousands of tools could become normal without loading them all up front — a prediction that may reverse the earlier "fewer tools" decisions.
- In that future, users may not need to know what MCP is; auth setup and tool selection become largely autonomous.
- Scale snapshot: over 11 million Docker downloads of the stdio server, 126 contributors, 2,300+ issues and pull requests, almost 4,000 forks, nearly 30,000 stars, and traffic approaching eight million calls a week.
- Audience question on piping tool calls: he suggests wrapping a server's read-only tools in a CLI with proper help text and observing how an agent does — a pattern he calls surprisingly effective.

## Resources

- [GitHub MCP Server](https://github.com/github/github-mcp-server) — the official repository, with install and tool-configuration docs.
- [Benchmarking Single Agent Performance](https://www.langchain.com/blog/react-agent-benchmarking) — LangChain's February research on how added tools affect agents.
- [GitHub MCP Exploited](https://invariantlabs.ai/blog/mcp-github-vulnerability) — Invariant Labs' prompt-injection exfiltration demonstration.
- [The lethal trifecta for AI agents](https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/) — Simon Willison on private data plus untrusted content plus the ability to act.
- [Cloudflare Code Mode](https://blog.cloudflare.com/code-mode/) — composing tool calls by generating code instead of calling tools one by one.
