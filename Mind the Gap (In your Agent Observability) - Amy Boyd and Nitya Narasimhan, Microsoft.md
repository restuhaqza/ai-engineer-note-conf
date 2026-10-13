# Mind the Gap (In your Agent Observability)

| Field | Details |
| --- | --- |
| **Speaker** | Amy Boyd, Nitya Narasimhan |
| **Affiliation** | Microsoft |
| **Type** | Workshop |
| **Date** | 2026-05-14 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=iOXM3zE-2dk) |
| **Tags** | `rag-context-and-search` `observability-and-reliability` `agent-engineering` `safety-and-governance` `developer-workflows-and-testing` |

## Summary

Amy Boyd and Nitya Narasimhan (Microsoft) run a hands-on workshop on closing the observability gap in AI agents. Their title comes from the London Underground warning: a new train (the agent) may fit one platform (the requirements) and leave a dangerous gap at another. The lesson is that better technology doesn't automatically fit the application, so teams must evaluate expected versus actual behavior early and continuously.

The workshop works through Microsoft Foundry on a fictional travel scenario (Contoso Travel: flights, hotels, rental cars). It covers three recurring activities — evaluate, monitor, and optimize — and shows how tracing records what an agent actually does, including which tools it calls and which messages it sends. Foundry builds on OpenTelemetry so instrumentation can span different agent implementations.

The presenters then move from the portal into code: creating an agent, adding tools, composing specialists into a workflow, exporting traces to Azure Monitor, running quality and agentic evaluations, and red-teaming the guardrails. A final path uses a coding agent and a Foundry "Observe" skill to generate evaluation data, find failures, optimize prompts, and keep the best version rather than the newest.

## Key Takeaways

- "Mind the gap" is the distance between what you expect an agent to do and what it actually does; evaluate early and often.
- Three activities form the loop: evaluate (quality and safety), monitor (as requirements and customers change), and optimize (act on the evidence).
- Tracing records the agent's real execution (tool calls, messages, steps); Foundry uses OpenTelemetry so different agent implementations can share one control plane.
- Evaluate the decisions inside a workflow, not just the final answer: intent resolution, tool selection, the call itself, and task adherence.
- Reproducibility first: fork the right workshop branch, launch a Codespace from the dev container, and expect a small Azure bill for deployed resources.
- Creating an agent doesn't connect tracing by itself — you must link Application Insights, or the Traces tab will flag it.
- Trace-linked evaluations shorten detection-to-diagnosis: a score says something changed, and the execution path shows what.
- Red teaming actively generates manipulated prompts (leetspeak, multi-turn Crescendo attacks) to test whether guardrails can be crossed.
- Let a coding agent establish a baseline, fix failures, and version each candidate — a newer prompt isn't automatically better; the demo returned to version five.

## Detailed Notes

### Mind the gap: the analogy and the loop · 01:06
- The title comes from London's "mind the gap" (New York says "watch the gap"): a new train can fit one platform perfectly and leave a gap at another.
- Treat the train as the agent and the platform as its requirements; improving the technology doesn't automatically make its behavior fit the application.
- The signs are also guardrails and repeated reminders — agents need safety mechanisms and ongoing monitoring, not just evaluations.
- The presenters' framing question: once observability produces data, what should a developer actually change?
- Three activities turn the problem into a loop: evaluate (performance, quality, safety), monitor (as requirements, customers, and environments change), and optimize (act on the evidence and debug failures).
- Scope grows from a first reliable agent, to debugging production, to managing fleets of many multi-agent systems.
- Amy Boyd leads Microsoft's Foundry Developer Relations team; Nitya Narasimhan specializes in observability.

### Evaluate the decisions inside the workflow · 07:33
- Tracing records what the agent does: the tools it calls, the messages it sends, and the steps a request passes through.
- Foundry builds on OpenTelemetry (OTel — a standard for tracing), so instrumentation can span agent implementations and feed a shared control plane.
- Evaluation adds judgments to that record; built-in evaluators cover quality, risk and safety, and agent-specific behavior, and custom evaluators cover anything else.
- A weather-agent example shows several checkpoints: interpret the request (intent as local weather), choose/select the expected tool, execute the call, and answer the user (task completion and instruction-following).
- For a nondeterministic agent, expected tool selection is something to measure across runs, and task-adherence criteria may need refinement over time.
- Evaluating intermediate decisions makes an imperfect answer actionable: you can see which stage to change.
- Continuous evaluations can run on code changes; scheduled ones look for drift; red teaming probes security. Foundry connects agent-level views to Azure Monitor, and a fleet view brings together agents hosted in different places.
- The open-source PyRIT framework supports proactive identification of generative-AI risks, alongside platform scan options.

### Start Contoso Travel in a reproducible environment · 14:23
- The scenario: at fictional Contoso, build a travel app that helps users find flights, hotels, and rental cars.
- Narasimhan frames the live session as a cooking show — she shows prepared results from a repository she calls almost a four-hour workshop.
- An agent needs instructions (its job), tools (so it can do that job), and a suitable model. The model is the knowledge, the agent is the experience, and tools extend both.
- Model selection is itself a search problem: she cites more than two million models on Hugging Face and more than 11,000 in Azure's catalog.
- The new app has no historical evaluation dataset, so the first challenge is getting a prototype and useful test data before settling every model choice.
- Two distinct obligations: protect the company's reputation in normal use (evaluation, like a building inspector) and withstand users who deliberately manipulate the agent (safeguarding, like inviting someone to break in).
- Setup: fork the repo with main-only unchecked so the `2026-04-aie-europe` branch is included, then launch a Codespace from the branch. The dev container defines dependencies; a free Azure account works, but deployed resources still incur charges (Narasimhan reports about $10 over two days).
- Two paths: the SDK path (recommended for newcomers) builds step by step; the coding-agent path starts from a minimal endpoint and uses a skill. Both use the same travel scenario.

### Create the agent, then connect its telemetry · 23:22
- In the portal: choose Start building, create a project, pick a subscription and resource group (East US 2 in the demo), and create a uniquely named Contoso Travel agent.
- The portal provisions GPT-4.1 automatically and looks for capacity; the playground exposes instructions, tools, and voice mode. Bing web search lets the assistant consult travel sources.
- With no specific instructions, the first agent offers generic help; the presenters then switch to a prepared version.
- Key gotcha: creating an agent does not connect tracing. The Traces tab flags the missing Application Insights; linking it through the administration interface sets the telemetry destination.
- Once connected, conversations and traces expose tokens, estimated cost, and evaluations.
- The prepared agent adds behavioral rules (use web search, show comparisons, present sourced specifics, stay focused), plus a display name, description, and starter interactions so users can see what it does.
- After a travel query, the response footer shows the model and token usage — useful when a router picks models per request.
- Task adherence scored low in the demo because the agent asked for more information instead of answering; even a minimal proof of concept had a concrete target.

### Build the same capabilities in code · 35:39
- A repository script uses the Azure CLI to collect configuration and generate `.env` (kept offscreen because it holds keys); the AI Toolkit extension brings Foundry-like features into VS Code.
- The notebook outline adds one bounded capability at a time: validate connectivity, create a basic agent, add tools, compose a workflow, configure tracing, run evaluations, and red-team.
- A smoke test with fake data confirms the environment can reach the service; Azure CLI login supplies credentials, and the Foundry client creates an agent with travel instructions and the deployed model.
- A first Responses API request asks about planning a trip to Paris; a following multi-turn test checks the conversation continues. At this stage custom tracing isn't active, so portal inspection shows the agent's traces but no evaluations.
- Next, register function tools for car, flight, and hotel searches; test single requests (a flight search) and combinations (a hotel plus a rental car).
- A further lab splits responsibilities into specialist agents, and a declarative YAML definition composes them into a Foundry Workflow that invokes each specialist and combines their responses.
- The resulting trace shows the path through the workflow, including which specialist underperformed and where tokens went — so you can replace or optimize one component without touching every agent.
- On cost: try a cheaper model, rerun evaluations immediately, and check for an accuracy regression; the trace also flags expensive or slow operations (for example, a web search that could be cached or data-backed). Agent versions can be selected and rolled back — that restores a configuration, not external side effects like completed bookings.

### Add debugging context and inspect failed evals · 51:11
- As an agent changes, more failure causes become plausible, so custom trace attributes record the context a developer will need.
- The tracing notebook configures console output and enables generative-AI tracing plus message-content capture (exact setting names depend on the workshop's package versions).
- Capture carefully: prompts, responses, and tool arguments can hold sensitive data, so capture only what's needed and apply redaction, access controls, and retention limits.
- The mechanism is an OTel span with developer-defined attributes — for example, recording the destination and agent role for a Paris-trip request without logging the whole prompt.
- Exporting telemetry adds a level: basic traces show what happened to the agent, while Azure Monitor can place AI telemetry beside other service telemetry (such as an Azure AI Search index) to reveal whether a slowdown comes from a dependency.
- The evaluation notebook runs quality, safety, and agentic checks individually or in batches: prepare a dataset, define evaluators and criteria, map fields to inputs, run, poll until done, then analyze locally or in the portal.
- Coherence and fluency passed in the deliberately simple dataset. A groundedness failure with the wrong year (2024 where 2025 was expected) opens into a concrete explanation — more useful than a failed status alone.

### Red-team the guardrails · 58:17
- Ordinary evaluations judge the examples given to them; a red-team scan actively generates manipulated prompts. A second AI gets risk categories and attack strategies, attacks the target, and reports where protections fail.
- The reversed-string illustration: a direct request for bank-robbery instructions should be refused, but reversed text may look meaningless to a guardrail while the model reconstructs the prohibited request. The point is the difference in interpretation across layers.
- For tool-using agents the risk extends to actions; a prohibited-action taxonomy can flag behaviors like entering passwords or leaking data, and the red-team agent generates prompts that try to induce them.
- An audience member proposes the stricter policy — allow nothing except explicitly authorized actions at authorized times — which still needs adversarial testing (say, a persuasive story built around a grandmother).
- The completed example selected violence, sensitive-data leakage, and task adherence as risk categories, using leetspeak (numbers substituted for letters) as the attack strategy; Narasimhan reports it passed all tested cases.
- Crescendo is a multi-turn technique that escalates gradually rather than relying on one transformed prompt — she compares it to a frog in slowly warming water. Longer scans can reveal weaknesses missed by simple tests, but they don't prove every vulnerability is found.

### Let a coding agent optimize, and keep the best version · 1:04:07
- The second path automates much of the notebook work with the Observe skill, a sub-skill in Microsoft Foundry's agent-development tooling, presented as an early preview.
- In GitHub Copilot Chat (using Claude as the coding model), she asks it to start an observability loop for the portal agent, starting from just instructions, a model, and an endpoint — no evaluation suite.
- The skill inspects instructions, code, and metadata, builds a cache, generates evaluation data, and turns the work into tasks: retrieve agent details, query the evaluator catalog, run an initial batch evaluation, and recover from a misspelled agent name.
- The narrated baseline: relevance and intent resolution looked good, task adherence had two failures, and one indirect safety test passed; the explanations pinpoint what to fix.
- Authorizing optimization, a prompt optimizer tightens ambiguous wording, publishes a new version, and reruns the same evaluators. The first revision fixed one of the two failures.
- Pushing further produced regressions rather than steady improvement; the iteration history became the useful artifact. By version 10, the demo identified version five as best and recommended returning to it.
- Takeaway: preserve the baseline, version each candidate, compare results, and keep the ability to choose an earlier candidate — newer isn't inherently better.
- The skill can also change the model (Narasimhan cites slow web search as a reason to investigate alternatives), and the installed `SKILL.md` exposes the checks and best practices. Domain knowledge still matters: a developer can direct the next experiment.
- It can produce a history and recommendations for future work; she mentions asking a create skill to convert a prompt agent into a hosted LangGraph implementation as a proposed next step.
- Assistance spans layers: Ask AI in Foundry reasons over project state (inspect a trace ID, check model access and quota), while an observability agent above Application Insights supports natural-language investigation where you'd otherwise write KQL (Kusto Query Language — Azure's log query language).
- The loop continues because a working agent can drift from its requirements. For further learning she points to Model Mondays and a Foundry skills session, and highlights AI Toolkit for bringing portal and SDK capabilities into VS Code.

## Resources

- **AIE Europe observability workshop** (repository) — https://github.com/Azure-Samples/microsoft-foundry-e2e-agent-observability-workshop
- **Foundry Observe skill source** (repository) — https://github.com/microsoft/azure-skills/blob/main/skills/microsoft-foundry/foundry-agent/observe/observe.md
- **PyRIT** (repository) — https://github.com/Azure/PyRIT
- **The Crescendo Multi-Turn LLM Jailbreak Attack** (paper) — https://arxiv.org/abs/2404.01833
- **Configure Foundry agent tracing** (documentation) — Application Insights tracing setup and troubleshooting. *(update)*
- **Understand Codespaces billing** (documentation) — who pays, including organization sponsorship. *(related)*
- **Model Mondays: Foundry agents in VS Code** (video). *(related)*
