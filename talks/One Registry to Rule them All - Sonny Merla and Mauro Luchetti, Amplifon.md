# One Registry to Rule them All

| Field | Details |
| --- | --- |
| **Speaker** | Sonny Merla, Mauro Luchetti, Mattia Redaelli |
| **Affiliation** | Amplifon (with Quantyca) |
| **Type** | Talk |
| **Date** | 2026-04-10 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=VXfRt_H-V08) |
| **Tags** | `safety-and-governance` `apis-mcp-and-protocols` `infrastructure-and-deployment` `enterprise` |

## Summary

Amplifon is a hearing-care company operating in 26 countries, with more than 20,000 people and over 10,000 stores. When dozens of teams across three continents began building AI agents, each one wired its own connections, invented its own security model, and deployed its own infrastructure. Sonny Merla, Mauro Luchetti, and Mattia Redaelli describe how the company's AmplifAI program brought order to that sprawl.

Their fix is an enterprise registry system. A shared AI gateway handles model access, authentication, budgets, and auditing. Underneath it sit three linked catalogs: one for MCP servers (Model Context Protocol — a standard way for agents to call tools), one for A2A agents (agent-to-agent, described with standard Agent Cards), and one for business use cases.

The use-case registry is what turns metadata into governance. It connects agents and tools to the business activities they support and records which models each use case consumes. That lineage lets the company trace a model change or outage back to the applications it affects.

The talk also covers delivery. Two GitHub blueprint repositories give teams a standard starting point, and CI/CD publishes both the container image and the service's registry metadata at release time. The main message: make governance a by-product of development and release, not a separate inventory exercise teams must remember later.

## Key Takeaways

- **Centralize governance, not infrastructure.** Amplifon wanted shared guidelines and reusable operations while teams still worked on different infrastructures.
- **One gateway for model access.** A unified endpoint provides authentication (via Microsoft Entra ID), per-use-case budgets, and central auditing.
- **Three registries, linked together.** MCP servers, A2A agents, and business use cases are cataloged and cross-referenced.
- **Enterprise metadata makes MCP servers operable.** Ownership, environment, authentication, cost attribution, and use-case links answer questions a technical description cannot.
- **Publishing is part of deployment.** When an agent ships, its CI/CD pipeline also publishes the Agent Card or `server.json` to the registry.
- **The use-case registry enables impact analysis.** If a shared model or component breaks, the catalog points back to affected use cases.
- **Keep the interface fixed, keep framework choice free.** The A2A blueprint is framework-agnostic; teams can use LangChain, Agno, or another library.
- **Governance should fall out of the release process**, not depend on a manual inventory.

## Detailed Notes

### The problem: dozens of teams building in isolation · 00:03
- Amplifon runs across 26 countries, 20,000+ people, and 10,000+ stores, so AI adoption has to scale responsibly under centrally defined rules.
- The near-term problem is duplication: teams rebuild security, connections, and deployment each time.
- The longer-term problem is visibility: the organization cannot easily see what exists, what depends on it, or who must respond when something changes.

### Governance model and execution plan · 01:17
- AmplifAI launched in January 2025 as a global, cross-functional program.
- A small **control tower** (senior leadership) sets security, legal, and technology guidelines and selects the first use cases.
- A **committee** carries that strategy into countries and corporate functions and prioritizes work more granularly.
- Execution splits into three parts: **governance** (rules and awareness), **platform** (certified infrastructure and practices), and **factory** (teams rolling out reusable solutions to markets).

### Why a catalog and lineage matter · 03:29
- LLMs have short life cycles, but the applications consuming them must stay useful over time.
- When a model changes or fails, Amplifon wants to identify affected use cases quickly.
- Governance needs to know where AI is used, what each use case implements, and which assets it consumes.
- A **catalog** provides the inventory; relationships between its records provide **lineage**.
- The goal is to let developers focus on business logic instead of repeatedly rebuilding security, deployment, and maintenance.

### AI gateway plus three registries · 05:30
- The **AI gateway** is the common entry point: developers point at one unified endpoint to reach all models in the catalog.
- **Entra ID** integration supplies authentication.
- Per-use-case **budgets** (monthly, weekly, or another period) erode as teams consume, so developers can inspect the remaining balance.
- Central auditing, monitoring, and analysis tools attach to the gateway to inspect requests and responses.
- Beneath the gateway, three registries divide the cataloging work: MCP (tools), A2A (agents), and use case (business linkage).

### Making MCP records useful for operations · 08:57
- Amplifon started from the community MCP Registry and extended it with private functionality and enterprise context.
- The internal catalog holds both custom servers built for Amplifon systems and a curated, approved set of public servers.
- Added metadata answers operational questions: **ownership**, **environment** (dev/test/prod), **authentication**, **cost attribution**, and **use-case linkage**.
- Cost attribution connects to the gateway budgets.
- Consumer links turn a server record into an entry point for impact analysis, forming an audit trail of available tooling and its use.

### A2A Agent Cards and the use-case registry · 11:36
- The A2A registry is built on **Agent Cards**: standard descriptions of an agent's identity, endpoint, capabilities, supported modalities, and authentication requirements.
- When an agent deploys through a blueprint, CI/CD automatically publishes its Agent Card, so a release becomes self-documenting.
- The **use-case registry** maps agents and tools to adopted use cases and records what each implements, consumes, and which models it uses.
- A platform walkthrough shows a "Ticket Optimization with AI" record with a linked MCP server and a LangChain test agent, plus an object-lineage view.
- An inspector page can open an MCP server or check an A2A Agent Card for compatibility; metadata wizards generate `server.json` or Agent Card descriptions from a form instead of raw JSON.

### Developer workflow: blueprints and CI/CD · 17:47
- Two GitHub template repositories (one for MCP, one for A2A) provide FastAPI servers, Dockerfiles, package tooling, infrastructure, and boilerplate.
- Authentication and cost tracking are handled inside the blueprints, giving every team a shared operational start.
- A platform-level **Langfuse** integration lets teams trace agents, run evaluations, and inspect performance.
- The A2A blueprint is framework-agnostic, organized around interfaces and ports, so teams keep their preferred agent framework (LangChain, Agno, or another).
- Tagging a branch triggers GitHub Actions to publish the Docker image **and** the service metadata (Agent Card or `server.json`) to the registry.
- At runtime, agents reach MCP or A2A proxies through an **Apigee AI gateway**, which looks up the real backend URL from the catalog; the calling agent then authenticates to the actual server with another header (the talk does not name the header or credential exchange).

### Governance as part of delivery · 21:38
- The result is a catalog of MCP and A2A servers across teams, with traceability across use cases, agents, tools, and models.
- Blueprints give developers a standard start so they can focus on business logic.
- Standardized CI/CD publishes service artifacts and registry metadata in the same delivery workflow.
- The platform is still growing; the durable choice is to make the information needed for governance a product of development and release, rather than a separate exercise.

## Resources

- [MCP Registry](https://github.com/modelcontextprotocol/registry) — the community registry Amplifon extended.
- [A2A Agent Discovery](https://a2a-protocol.org/latest/topics/agent-discovery/) — explains Agent Cards, curated catalogs, and discovery.
- [Langfuse](https://langfuse.com/docs) — open-source tracing, usage analysis, and evaluation.
- [Current Apigee AI gateway capabilities](https://docs.cloud.google.com/apigee/docs/api-platform/get-started/ai-capabilities) — AI traffic and gateway features.
