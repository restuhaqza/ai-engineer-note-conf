# What if the network was the sandbox?

| Field | Details |
| --- | --- |
| **Speaker** | Remy Guercio |
| **Affiliation** | Tailscale |
| **Type** | Expo session |
| **Date** | 2026-06-01 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=BM2JX9hqsVQ) |
| **Tags** | `observability-and-reliability` `coding-and-developer-tools` `apis-mcp-and-protocols` `safety-and-governance` |

## Summary

Remy Guercio of Tailscale poses a question about how we sandbox AI agents. A sandbox is usually described as a box — a VM or a container — but he argues the box is only half of it. The other half is permissions, and today those permissions usually live inside the box. An agent gets an API key or a logged-in account, so it holds the credential that grants its own access. He asks whether the network could carry that boundary instead.

Tailscale builds an identity layer on top of WireGuard connections. Every connection in a private network (a "tailnet") can carry a user, a group or a workload tag, so a receiving service knows who is calling before acting. Their gateway, Aperture, applies this to LLMs: it holds the provider keys, while a sandbox connects with a placeholder key and Aperture decides what it may do and records every request. In a live demo, Guercio inspects his own Claude Code traffic and a PR-review bot's tool calls, then shows shared budgets and webhooks.

The talk is candid about the edges. Model, provider and quota permissions are implemented; finer per-tool rules are still coming. Observing an operation is not the same as understanding everything it might do, especially when an agent writes and executes code. But for many teams, simply seeing what agents actually use — often Bash — is already valuable.

## Key Takeaways

- **A sandbox needs both a boundary and permissions.** Without permissions it isolates something without giving it useful access.
- **Credentials inside the sandbox undermine the isolation.** An agent that holds its own key can reuse, share or leak it.
- **Network identity is an alternative.** Tailscale attaches user, group and workload-tag identity to every connection.
- **Keep provider keys at the gateway.** Aperture stores upstream credentials; sandboxes use a dummy `-` key pointed at the gateway's URL.
- **Revocation is immediate.** Traffic has to pass through the gateway, so access can be cut off without the agent's cooperation.
- **Budgets and webhooks become shared controls.** Cost limits can span providers, and tool-call events can be forwarded from one central place.
- **You can build this yourself.** The open-source `tsnet` library lets a Go service join the network and resolve who is calling.
- **Visibility is not full understanding.** Agents that generate and run code can obscure their effects; per-tool permissions and command guardrails are still maturing.

## Detailed Notes

### A sandbox needs a boundary and permissions · 00:43
- Guercio strips a sandbox to two parts: a **boundary** (inside vs outside) and a set of **permissions** deciding what can cross it. Without permissions, he says, it is a sandbox with no toys.
- The familiar debate is VMs versus containers, and whether the agent belongs inside or outside the box; he sets that aside to look at how agents get access today.
- Two usual routes: an **API key** (full price, and the key does not really express an authorization policy) or an **OAuth/OIDC login** (access tied to an account, but the session still lives with the agent).
- He separates authentication (who you are) from authorization (what you may do). Handing over a key blurs the two, and a long-running agent may try to reuse or manipulate even a synthetic key.
- Concrete image: an OpenClaw instance running elsewhere with a logged-in account — the agent is boxed, but the thing granting access is inside the box too. Hence his question: can the network hold the permission boundary?

### The network carries identity · 03:21
- WireGuard gives network nodes cryptographic keys; Tailscale builds an identity layer on top and connects devices into a private network called a **tailnet**.
- Participants can be containers, GPU servers, laptops or phones — the useful abstraction is an identified connection, not a machine type.
- Identity can describe a person or a workload: a logged-in laptop maps to its user; SCIM-synced membership maps to groups; a GitHub Actions runner can carry a tag identifying it as a specific PR-review bot.
- That data supports two decisions: network policy can refuse an unauthorized connection outright, and a receiving service can read the caller's identity for its own authorization instead of starting from an IP address and demanding a separate key.

### Aperture: keep provider keys at the gateway · 05:38
- **Aperture** is Tailscale's identity-aware LLM gateway. Like other gateways it stores provider credentials and forwards requests; he names Anthropic, OpenAI, Gemini, Vertex and Bedrock as upstreams.
- The difference is that Aperture is itself a tailnet node, so it can identify clients without issuing each one a provider key.
- Walk-through for a PR-review agent: the runner uses GitHub's federated OIDC identity to join the tailnet, receives a workload tag, connects to Aperture, and Aperture's rules decide what that tagged workload may reach.
- The provider key never leaves Aperture, so the sandbox has no upstream secret to leak or carry elsewhere. He frames it as requesting access without holding the credential that fulfills the request.

### Demo: from your identity to the request body · 07:22
- Live in the dashboard, Aperture recognizes Guercio as a user because his laptop is on the corporate tailnet, and ties token, model and spending metrics to that identity.
- Drilling into a request exposes headers, request body and response body — including the large context Claude Code sends before the user's short instruction.
- He corrects himself mid-demo: the request asked for a ten-word story (a cat on a mat that vanishes), not a greeting. The visible user prompt is only a small part of the payload.
- The session groups two requests — a Haiku summary plus the story request — and he reports the story request cost $0.20 with all that supplied context.

### A PR review bot, seen as a workload · 09:47
- He opens a small PR-review check that runs on every update; its activity is attributed to a dogfood tag over a 30-day view rather than to a person.
- A displayed bot request cost $0.04 and ran three commands at once; drilling in shows the Bash and grep work, an MCP (a standard way for agents to call tools) call to update the code review, and a later update to the review comment.
- The extraction happens at the gateway, outside the container and outside the agent's own harness — so it is visibility into tool calls in routed model traffic, not a record of every local process.
- Because the control lives at the gateway, cutting access off is immediate: the sandbox cannot rescue the request by taking its key to another endpoint, since it never had one.

### A placeholder key and a nine-word story · 11:41
- The client setup is tiny: Claude Code runs in API-key mode with a literal `-` as a placeholder, pointed at Aperture's base URL. The dash only satisfies the client's key field; it does not authorize anything.
- He shows the same arrangement in his real `settings.json` and names Codex and Gemini CLI as other supported clients.
- During a live ten-word-story request, an audience member counts nine words. Guercio names the model as Opus 4.6 and jokes about its one-million-token capacity missing a small constraint; a follow-up prompt fixes the ending.
- He notes the same prompt drifted from stories about robots, to dogs, to cats over about three weeks — a reminder that pipelines depending on exact output structure can break far from where the cause is obvious. The log preserves both the failure and the model's acknowledgment.

### Budgets, hooks, and building your own · 14:07
- Cost controls can span providers: one $1,000 allowance the owner distributes as they wish, rather than separate locked budgets at each provider.
- **Webhooks** let Aperture push tool-call information to a third party. Placing the hook in the gateway means routed interactions trigger it without relying on a cooperative hook inside the agent; he ties this to reliable invocation, not guaranteed delivery.
- A smaller quota (for example $5 a day) shows how the gateway can bound experimentation.
- The underlying primitive is the open-source **tsnet** library, which lets a Go program join a tailnet and read the identity of incoming connections.
- A Go example in the bundle uses `tsnet.Server` and `LocalClient.WhoIs` to return the connecting node and, when present, its user — the building block for identity-aware internal APIs or MCP servers without public exposure or a separate OAuth flow. He notes Aperture was required to use only public building blocks, so in principle others could rebuild it.

### Permissions, trade-offs, and the limits of visibility · 17:04
- On configuration: grants can be set in Aperture's UI (model access, quotas, MCP access, hooks, roles; he describes group selection as forthcoming) or declared in Tailscale's policy file as **application capabilities**, managed through JSON or an API for GitOps workflows.
- Asked why not transparently intercept requests to a provider's default URL, he says it is possible but not chosen: explicitly setting the base URL keeps the gateway visible in the client's configuration, and hidden interception tends to break confusingly as systems change.
- On per-tool permissions, he separates what ships from what is in progress: model, provider and quota permissions exist, while finer tool-specific rules and group handling are still being built. He describes team and individual allowances as a union without specifying its enforcement algorithm.
- For agents that write and execute code rather than call named tools, permissioning gets harder. Observing the LLM layer (rather than only an MCP proxy) captures more, and commands usually appear in the interaction, but obfuscated code can still hide its effects. Guardrails like rejecting `rm -rf /` are future work, and he notes Bash dominates real usage inside Tailscale — so discovery often comes before blocking.

## Resources

- [How Aperture works](https://tailscale.com/docs/aperture/how-aperture-works)
- [Build an identity-aware Go service with tsnet](https://tailscale.com/docs/reference/tsnet-server-api)
- [Tailscale workload identity federation](https://tailscale.com/docs/features/workload-identity-federation)
- [Introducing Claude Opus 4.6](https://www.anthropic.com/news/claude-opus-4-6)