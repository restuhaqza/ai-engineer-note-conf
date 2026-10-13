# Why, and how you need to sandbox AI-Generated Code?

| Field | Details |
| --- | --- |
| **Speaker** | Harshil Agrawal |
| **Affiliation** | Cloudflare |
| **Type** | Talk |
| **Date** | 2026-04-08 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=AHtGAgQ0Q_Q) |
| **Tags** | `coding-and-developer-tools` `rag-context-and-search` |

## Summary

More and more apps let an LLM write code that then actually runs. Harshil Agrawal, a developer educator at Cloudflare, argues we should treat that code the way we would treat a random snippet from the internet: as untrusted input. A model is a text generator with no loyalty or intent, so it can be right, subtly wrong, or steered by an attacker.

He walks through three failure routes. First, plain bugs: bad imports, recursion without a base case, infinite loops. Second, an over-helpful model that reads API keys and secrets while "just trying to help." Third, prompt injection, where hostile text hidden in a document or page turns the model into an attack vector. All three get dangerous for the same reason: the generated code inherits the app's filesystem, network, database, and credentials.

The fix is old and proven: sandboxing, guided by capability-based security. Start from no permissions and grant only what the task needs. Agrawal builds two demos to compare two tools: fast V8 isolates (separate JavaScript runtimes) for small generated functions, and full Linux containers for work that needs files, packages, and servers.

He closes with an eight-point checklist that works for any sandbox: block the network, grant narrow capabilities, isolate each user, cap resources, keep secrets outside, clean up, log everything, and validate code before running it.

## Key Takeaways

- **AI-generated code is untrusted code.** The LLM has no intent; it just predicts text that looks like code.
- **Three ways it hurts:** ordinary bugs, secret-leaking "helpfulness," and prompt injection.
- **Capability-based security = deny by default, allow explicitly.** Don't try to blocklist every bad thing.
- **Use V8 isolates** for fast, small generated functions such as tool calls and data transforms.
- **Use Linux containers** when the code needs a filesystem, package installs, or a long-running server.
- **Block outbound network by default.** Route any needed call through a service you control.
- **One user, one sandbox.** Never share a filesystem between tenants; that's a data-leak path.
- **Keep secrets outside and clean up.** Proxy sensitive calls, and destroy sandboxes with `try/finally`.

## Detailed Notes

### AI-generated code is untrusted code · 00:28
- We went from autocomplete to generated modules to agents that write, run, and revise code without asking.
- Strip away the AI framing: you are running code from a black box with your own credentials.
- Running a random internet snippet via `eval` would be rejected instantly; generating it with an LLM changes nothing.
- The model has no intentions and no loyalty — it produces text shaped like code.

### Three ways generated code causes harm · 03:07
- **Bugs:** a nonexistent import, recursion with no base case, or an infinite loop. Can crash a process or burn compute.
- **Over-helpful:** asked to configure a database, the code inspects the environment and reads API keys and secrets. No thief required.
- **Prompt injection:** direct (a user says "ignore your instructions") or indirect (a document the model reads hides instructions).
- The shared cause is architectural: generated code runs with the app's filesystem, network, database, and keys.

### Capability-based security and choosing a boundary · 06:55
- Sandboxing is decades old: browser tabs, operating-system processes, phone app permissions.
- The principle: don't enumerate what to block; enumerate what to allow. Default deny, then grant the minimum.
- Building analogy: a master key plus 10,000 banned rooms, versus keys to just the three rooms you need.
- Isolation spectrum: `eval` (no isolation) → isolates (constrained runtime) → containers (full Linux).
- Five questions before choosing: secrets, networking, filesystem, tenants, and resources.

### Isolates: a fast, tiny execution surface · 13:08
- Demo 1: an agent generates small repeatable functions ("skills") that need speed, not a filesystem.
- Cloudflare Dynamic Workers create a Worker isolate at runtime; the core call is `loader.load`.
- The isolate has its own memory and globals and cannot reach the parent Worker's environment unless passed in.
- `globalOutbound: null` blocks all outbound requests. The `env` object carries only a narrow database binding and a logger.
- The database binding calls back into trusted code (via RPC), scoped to the user, exposing only a `query` method.
- Networking has three levels: blocked, mediated (through your service), or unrestricted. Start blocked.

### Containers: a full Linux workbench · 21:12
- Demo 2: PromptMotion turns a prompt like "animate this logo" into a running app with a live preview URL.
- That needs `git clone`, `npm install`, a build, a dev server, and a reachable port — impossible in an isolate.
- Path: Worker → Sandbox SDK → Durable Object (lifecycle coordinator) → Linux container.
- Each user gets their own container and filesystem; other users' files are absent, not merely hidden.
- Steps: pick the user's sandbox, clone, install, start the dev server, expose the port, return the URL.

### Isolation, secrets, and cleanup · 26:16
- Rule: **one user, one sandbox.** A shared sandbox means a shared filesystem and a data-leak vector.
- Don't pass API keys as environment variables into the container; any code there can read them.
- Instead, proxy: the sandbox calls your Worker, which adds the real key and forwards the request upstream.
- The key lives in the Worker's environment, never in the sandbox.
- Clean up with `try/finally` (not just `catch`) so failures still destroy the container.
- Set a maximum lifetime; an idle sandbox costs money and stays a live security surface.

### Pick the environment for the step · 31:03
- One question decides it: does this step need a filesystem, processes, or package installs? Yes → container. No → isolate.
- Isolates fit tool calls, snippet interpreters, and data transforms; containers fit builds, tests, and servers.
- Agents can use both: isolates for a fast iteration loop, containers when it's time to build and deploy.
- Trade-offs: containers are heavier, slower to start, and more expensive, but they unlock real capabilities.

### Eight safeguards for any sandbox · 33:16
- Deny network access by default; open only the routes the task needs.
- Grant explicit capabilities; expose the minimum methods and data.
- Isolate each user; never share execution environments across tenants.
- Limit resources with timeouts, memory caps, and CPU limits.
- Keep secrets outside by proxying sensitive operations through your own code.
- Clean up finished environments and set maximum lifetimes.
- Log what ran, when, who triggered it, and what it did.
- Validate before execution: length limits, syntax checks, and dangerous-pattern scans.

## Resources

- [Dynamic Workers documentation](https://developers.cloudflare.com/dynamic-workers/)
- [Cloudflare Sandbox SDK](https://developers.cloudflare.com/sandbox/)
- [Code Mode: the better way to use MCP](https://blog.cloudflare.com/code-mode/)
- [PromptMotion](https://promptmotion.app/)
