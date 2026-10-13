# What Breaks When You Build AI Under Sovereignty Constraints

| Field | Details |
| --- | --- |
| **Speaker** | Bilge Yücel |
| **Affiliation** | deepset GmbH |
| **Type** | Talk |
| **Date** | 2026-05-19 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=x2bH0RKPgdc) |
| **Tags** | `infrastructure-and-deployment` `observability-and-reliability` `architecture` `apis-mcp-and-protocols` |

## Summary

Bilge Yücel, a senior developer relations engineer at deepset (the company behind the Haystack framework), treats "sovereign AI" as an engineering problem rather than a legal one. She defines it as having explicit control over data flow, model choice, infrastructure, observability, and operations — then breaks that into four pillars: data, model, infrastructure, and operational sovereignty.

The talk walks through what actually breaks when a working AI system is retrofitted for sovereignty. Replacing a frontier API with a self-hosted model means rewriting API logic, revising prompts and re-evaluating performance. Moving data across borders can leave the team running several databases and instances, and moving off managed infrastructure hands back Kubernetes and GPU-connection work.

Yücel then shows how Haystack's orchestration helps: consistent component interfaces, explicit typed data flow, and serializable pipeline definitions that can be version-controlled. She sketches a sovereign agent architecture with input and output guardrails, bounded tool access through MCP, human confirmation for consequential actions, and tracing — and closes with three questions to test whether a team can operate independently.

## Key Takeaways

- **Sovereignty is a spectrum, not a binary.** Finance or healthcare may need an air-gapped setup; a startup may only need to know its level of vendor lock-in.
- **Storage location is not the whole data path.** Sending documents to an embedding API hosted abroad moves their processing out of a trusted boundary.
- **Access permissions count too.** A colleague seeing data they should not, even inside the same jurisdiction, still breaches the intended boundary.
- **Swappability is an architectural property.** If switching models means rewriting application logic, the organization is not really free to choose.
- **Retrofitting breaks more than the model call.** API logic, prompts, multiple databases and search routing all change.
- **Orchestration reduces integration edits, not every migration task.** It cannot supply missing GPU capacity.
- **Guardrails and bounded tools add control.** Input and output guardrails classify and check, while `tool_names` limits which tools an agent can reach.
- **The closing test is capability, not paperwork.** Can you swap models, keep reproducible run logs, and respond to an incident without a vendor?

## Detailed Notes

### Sovereignty as an engineering question · 00:24
- Yücel introduces deepset, the company behind the open-source Haystack orchestration framework, and names Airbus, Bosch, Siemens, the European Commission and German ministries as customers.
- The standard policy definition is an organization's ability to design, deploy and operate AI on its own terms.
- For engineers, she translates that into explicit control over data flow, model choice, infrastructure, observability and operations.
- Her four **pillars**: data (where stored and processed), models (who controls them and their training origin), infrastructure (where compute runs), and operations (traceability, updates, incident response).
- The pillars overlap, but each exposes a different dependency — keeping storage local says nothing about where inference runs.

### Follow the data, then locate the compute · 02:22
- **Data sovereignty:** a European database is not enough if documents go to an embedding API hosted in Virginia; processing has crossed the boundary.
- Access permissions belong in the same analysis: a user seeing data they should not violates the boundary even when everything runs in the approved region.
- **Infrastructure sovereignty:** ingestion, RAG pipelines, agents and their tools all run somewhere.
- She describes a spectrum from maximum control to maximum convenience: air-gapped, private VPC, sovereign cloud, and SaaS.
- Provider jurisdiction is a separate axis from hosting: a US-headquartered provider raises a concern even when data stays in Europe.
- The slide's safety labels are simplifications; deployment isolation does not replace the documentation and oversight obligations that AI Act guidance describes.

### Make model choice usable and operations accountable · 04:29
- **Model sovereignty** starts with freedom to choose and switch models. A one-provider system inherits that provider's outages and price changes.
- Supporting alternatives in principle is not enough: if switching requires extensive application changes, the choice cannot be exercised easily.
- **Training-data provenance** is a separate concern; teams often cannot tell what trained a model. A European provider may have an advantage, but headquarters does not prove the corpus origin.
- **Operational sovereignty** extends into production: monitoring and evaluating behaviour, inspecting inputs and outputs, and managing model and application updates in a controlled, auditable way.
- In high-stakes HR or finance settings, human oversight is part of the operating model.
- The required degree of control varies by domain, so the practical task is to identify the control your system gives you and the dependencies it retains.

### What breaks when you retrofit a working system · 07:09
- The scenario: a working AI system, and a manager or CIO asks the team to make it sovereign.
- The intuitive first move — replace the frontier API with a self-hosted model — forces API logic to be adapted, prompts revised, and performance re-evaluated.
- Moving private data into the required jurisdiction can leave the team running multiple databases and instances.
- Retrieval then has a new problem: classify the query and route it, or search both databases and combine the results. Either adds behaviour a single-store design did not need.
- Replacing managed infrastructure with on-premises deployment hands back Kubernetes cluster management and makes hardware limits explicit — the application may run on CPUs while inference runs on GPUs, requiring connection and network management between them.
- Adding observability reveals whether the application was understandable at all; an opaque AI layer cannot become auditable just by attaching a logging destination.

### How orchestration exposes dependencies · 09:44
- Haystack offers consistent component interfaces, so a cloud-backed component can be swapped for one connected to self-hosted inference while the surrounding interface stays the same.
- Yücel frames this as reducing integration edits, leaving the team to focus on hardware and connectivity — not as completing the whole migration.
- **Explicit data flow:** typed, declared inputs and outputs let you read a pipeline definition and follow what moves between components.
- Agent execution is less deterministic, but tool calls and tool outputs can still be traced, so a changing path is not an invisible one.
- Pipelines can be serialized to YAML and committed, making application structure inspectable across revisions.
- Open-source components add another form of control: teams can inspect, customize and extend them rather than treating them as a black box.

### Wrap the agent with guardrails · 11:32
- The proposed architecture starts with an **input guardrail** that checks for prompt injection and whether a request fits the application's permitted intent.
- Input classified as unsafe leaves the application path; safe input proceeds to the agent.
- This is a classification control, not proof that every malicious request will be caught.
- The agent combines an LLM, a system prompt and tools; tools can call APIs, search a knowledge base, invoke other agents, or connect through MCP servers.
- An **output guardrail** then checks the response for compliance and sensitive-information leakage before it reaches the user.
- Deployment choices can be mixed by task: public-data work may use hosted proprietary models, while guardrails or knowledge-base tasks use locally hosted ones (she names Mistral, Google, NVIDIA and Jina).
- Component activity and agent spans can feed an LLM observability system, with OpenTelemetry integration for custom setups; tracing must be switched on deliberately.

### Route messages, bound tools, connect inference · 14:36
- The walkthrough starts with a guardrail model configured through `NvidiaChatGenerator`; connecting it to `LLMMessagesRouter` turns classification into pipeline routing, with safe and unsafe paths.
- The router matches model outputs against configured patterns and has an unmatched-output path, so implementations must handle classifications that fit neither route.
- `MCPToolset` connects to a locally hosted MCP server and selects named tools, such as knowledge-base search and PDF-report generation.
- `tool_names` restricts the discovered inventory because a server may expose capabilities this agent should not receive — but it is not a substitute for authorization enforced by the server.
- Tools can also be Python functions, Haystack components, other agents, or whole workflows such as ingestion and RAG pipelines.
- These are placed in a `SearchableToolset`, which uses BM25 to discover relevant tools dynamically instead of putting every definition in the model's context.
- The agent's chat generator supplies inference; for an internal on-premises endpoint, a team can write a custom component, or reuse an existing one if the endpoint is OpenAI API compatible.

### Human confirmation, assembled workflow, and the closing test · 16:19
- **Confirmation strategies** add human-in-the-loop control. In the payment example, submitting a request asks for approval every time, while listing payment requests asks once and is then allowed repeatedly within the cycle.
- The final assembly adds a tracer connected to LLM observability, the input guardrail, the agent and its tools, and the output guardrail.
- The concrete request is to pull outstanding Q3 payment requests and generate a PDF, which the agent does with its tools and saves to a directory.
- Her closing checklist of independence:
  - **Model replacement:** can you swap models without changing application logic?
  - **Auditable execution:** do you have reproducible run logs stored in a compliant way?
  - **Incident response:** can your team respond to an incident without calling a vendor or hyperscaler?

## Resources

- Engineering AI Systems Under Sovereignty Constraints — Yücel's recap of the four sovereignty dimensions.
- Haystack documentation: `LLMMessagesRouter`, `MCPToolset`, `NvidiaChatGenerator`, tracing, and pipeline serialization.
- SearchableToolset in Haystack v2.25.0 — dynamic tool discovery example.
- EU rules on international data transfers — European Commission explanation of safeguards.
- The Purpose and Impact of the CLOUD Act — US Department of Justice explanation of cross-border data disclosure.
