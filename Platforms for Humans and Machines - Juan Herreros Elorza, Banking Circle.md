# Platforms for Humans and Machines

| Field | Details |
| --- | --- |
| **Speaker** | Juan Herreros Elorza |
| **Affiliation** | Banking Circle |
| **Type** | Talk |
| **Date** | 2026-04-08 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=cCRO3ChaYhM) |
| **Tags** | `evals` `apis-mcp-and-protocols` `finance` `safety-and-governance` |

## Summary

Juan Herreros Elorza leads the cloud-native technology team at Banking Circle, a
cross-border payments provider. He argues that internal developer platforms should
serve AI agents as well as they serve human developers. A coding agent cannot walk
to another floor to ask an infrastructure colleague to please create a database —
so the human handoffs, copied pipelines, and waiting that already frustrate
developers become hard limits on what agents can do.

His illustration is Banking Circle's platform, **Atlas**, which splits into compute
(Kubernetes), infrastructure (storage, databases, secrets), messaging, and
observability. The talk is a set of design principles for making such a platform
usable by machines, each backed by a story of a developer who has code ready but
cannot get it running.

The recommendations: make operations self-service and API-based, move feedback into
the agent's local working loop, expose observability as data (not dashboards),
place documentation where agents can find it, and let contributions grow without
handing off ownership. Then measure whether any of it actually helped.

His closing advice is organizational: interest in AI is high right now, so teams
can use it to finally fund the API-first, local-friendly, well-documented platform
work they have wanted for years.

## Key Takeaways

- **Agents expose existing platform weaknesses.** Problems that were merely
  annoying for people become blockers for machines.
- **Self-service means finishing the whole operation without a specific person.**
  Automation that still needs a human referral is not self-service.
- **Start from a well-defined API.** A CLI or an MCP (a standard way for agents to
  call tools) interface can wrap it for discoverability.
- **Fail as early as possible.** Local validation beats pushing a change and
  waiting minutes for a pipeline to fail.
- **Observability must be machine-readable.** Expose logs, metrics, and traces via
  API, CLI, or MCP — agents do not read dashboards.
- **Documentation placement matters.** Put docs next to code for small repos and in
  a central, discoverable place for larger platforms.
- **Instructions guide behavior; policies enforce boundaries.** Welcome
  contributions but keep security and compliance in systems that enforce them.
- **Measure the outcome, not the artifact.** An API or doc portal existing is not
  proof it improved anything.

## Detailed Notes

### How finished code becomes a running system · 00:09
- Banking Circle is a cross-border payments provider offering accounts and
  liquidity management; it reports processing over €1 trillion a year and serving
  over 700 regulated financial institutions.
- More than 250 people build technical systems there, spanning client APIs, core
  banking, internal tools, data science, data engineering, and clearing-scheme
  integrations.
- The platform team exists to abstract infrastructure and cloud complexity so other
  teams can focus on their systems.
- **Atlas** is organized into capabilities: **compute** (Kubernetes),
  **infrastructure** (blob storage, databases, secrets), **messaging**, and
  **observability**.
- Providing these capabilities is only part of the job; the developer also needs a
  usable route through them.

### The application is ready; the deployment is not · 02:33
- A new developer writes their first application; the code is ready, but where and
  how should it run?
- A teammate points to an existing pipeline and says to copy it. The developer
  adjusts it, runs it, and it fails — with an error that has nothing to do with the
  application they wrote.
- The next suggestion is to contact someone in infrastructure, who recognizes a
  recurring error and fixes it.
- The deployment then succeeds, only to reveal that a database or blob storage is
  also needed. That request waits behind other work — "I'll help you next week."
- The developer ends up juggling copied configuration, unexplained failures,
  personal referrals, and a queue; documentation is supplemented by knowing whom to
  ask.
- A human can absorb this by walking to another floor. A coding agent generally
  cannot, so the same gaps become hard limits — and the costly handoffs were
  already a problem before agents.

### Make the whole operation self-service · 07:03
- **Self-service means completing an operation without depending on a particular
  person** — the same route available to a developer and to an agent.
- Automation alone is not enough: if a task needs building blocks from five places,
  assembled and triggered elsewhere, the caller still has to discover the process.
- Self-service can be a message or a button, but a **well-defined API** is a good
  foundation for agents; a CLI or a **Model Context Protocol (MCP)** tool interface
  can wrap it.
- Interface definitions give callers ways to discover operations and their options
  without reverse-engineering someone else's pipeline.
- **Schema validation** checks whether a request has the expected structure; it
  does not prove the request will succeed.
- Authentication and authorization govern who can act — the agent might use the
  developer's credentials or its own identity.
- With those boundaries in place, API responses become feedback: the agent acts,
  reads the response, and decides its next step.

### Move feedback into the agent's working loop · 10:02
- In the setup Juan describes, the agent runs on the developer's machine while the
  model runs elsewhere; server-hosted agents are another option.
- The platform should make operations accessible from wherever the agent works —
  here, letting it validate and act locally.
- **Fail early.** Making an agent push to version control and wait minutes for a
  workflow failure delays information it could have had immediately.
- Where checks can run up front, expose them locally through the API or a wrapper,
  so the agent can change, validate, and revise without crossing a remote pipeline
  each time.
- Direction alone is not enough; the task also needs a completion condition.
  Expose **logs, metrics, and traces** through an API, CLI, or MCP server.
- The full loop: read the outcome and its success criteria, perform the operation,
  retrieve state and evidence, then compare evidence with the criteria before
  declaring completion.
- A successful deployment request and a correctly running application are different
  observations — the platform must expose both.

### Put instructions where the agent can find them · 12:29
- Documentation that exists but cannot be found does little to reduce a dependency
  on colleagues. Placement should match scope.
- **Small repositories:** keep docs beside the code they explain, so an agent
  discovers both together.
- **Larger systems and shared platforms:** provide a central place for an agent to
  discover available documentation.
- **Retrieval:** serve relevant pieces through an API, so the agent doesn't load a
  whole HTML page to find the useful part.
- Agent-specific instructions add working conventions: `AGENTS.md`, `CLAUDE.md`, or
  `copilot-instructions.md`, depending on the agent.
- A repository can specify how to build, test, deploy, and verify — the important
  content is the procedure and its verification step, not a description of what the
  repo contains.
- **Skills** capture how to carry out a recurring kind of task (for example, how to
  interact with a platform). Repository instructions set the context; skills
  explain a repeated operation.

### Welcome contributions without handing off responsibility · 15:17
- An internal platform serves developers who can also improve it; Juan reports
  greater willingness to contribute as agents lower the barrier.
- That can add features users need, but maintenance responsibility stays with the
  owning team; security, compliance, and standards must still hold.
- Two complementary mechanisms: **enforced policies** prevent outcomes that must
  not occur, while `AGENTS.md` and skills explain conventions and the intended
  contribution process.
- The principle: instructions guide behavior, policies enforce boundaries. Don't
  make platform safety depend on an agent following every instruction.

### Measure the outcome the platform was meant to improve · 17:10
- Once developers and agents can use the platform, ask whether it helped; choose
  the intended outcome and compare before and after.
- An API, a doc portal, or an agent workflow is an implementation choice — its
  existence does not prove improvement.
- For delivery, Juan points to **DORA** metrics (change frequency, mean time to
  recovery, lead time, and others). Note the current DORA guide uses five metrics,
  so define carefully which recovery time you mean.
- Other lenses: **reliability** (application errors and traffic — especially in
  fintech), **platform support** (support-request counts, which good self-service
  should reduce but confusing self-service can raise), and **developer experience**
  (via a framework like SPACE, which considers satisfaction alongside other
  dimensions).
- Juan supplies no measured before-and-after numbers; the requirement is to decide
  what the change should accomplish, then measure it.

### Use interest in AI to fund the missing foundations · 19:25
- Self-service and APIs give the agent a route to act; local feedback,
  documentation, and observability let it complete and verify the task.
- Contributions can extend the platform, and measurement tests whether the work
  helps its users.
- The last obstacle may be organizational, not technical: API-first interfaces,
  local workflows, and proper documentation may have lost priority even when their
  value was understood.
- Interest in AI — from executives to individual contributors — creates a reason to
  revisit those decisions and fund foundations that were missing all along.
- The work that lets an agent finish a deployment also removes the referrals,
  unexplained failures, and waiting that blocked the developer.

## Resources

- Companion slides for "Platforms for Humans and Machines".
- DORA software delivery metrics.
- The SPACE of Developer Productivity paper.
- AGENTS.md conventions.
- Agent Skills specification.
- MCP tools specification.
- OpenClaw repository.
