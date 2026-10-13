# From Chaos to Choreography: Multi-Agent Orchestration Patterns That Actually Work

| Field | Details |
| --- | --- |
| **Speaker** | Sandipan Bhaumik |
| **Affiliation** | Databricks |
| **Type** | Talk |
| **Date** | 2026-04-08 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=2czYyrTzILg) |
| **Tags** | `agent-engineering` `observability-and-reliability` `infrastructure-and-deployment` `architecture` |

## Summary

Going from one AI agent to five is not like adding features — it is building a distributed system. Sandipan Bhaumik, who has spent years scaling data systems and two years deploying multi-agent AI, argues that the failures come from missing distribution-system thinking, not from weak models.

He tells a production war story: a credit-decisioning pipeline where, three days after adding four agents, 20% of decisions had wrong risk ratings. The cause was a shared cache that was never invalidated — one agent read a stale credit score 500 ms after another wrote a new one. The race condition lived in the architecture, not in any prompt.

He then lays out the patterns that fix this: choose who coordinates (decentralized **choreography** vs. centralized **orchestration**), pass immutable versioned state instead of overwriting shared records, enforce data contracts at each handoff, wrap calls in **circuit breakers**, and use **Saga compensation** to undo completed work when a later step fails.

He finishes with a production blueprint on Databricks — LangGraph as orchestrator, Unity Catalog functions as agents, Delta Lake for append-only state, MLflow for tracing — and a reminder that this unglamorous infrastructure is what makes systems reliable.

## Key Takeaways

- **More agents = a distributed system.** Coordination, consistency, and partial-failure problems appear that model quality alone cannot fix.
- **Connections grow fast.** Five agents already create ten possible pairwise connections; each one is a potential race or sync bug.
- **Pick a coordination pattern deliberately.** Choreography (events) for simple, autonomous, fast-changing workflows; orchestration (a coordinator) for complex, stable ones.
- **Don't do shared mutable state.** Pass immutable, versioned snapshots so agents never overwrite each other.
- **Validate at the boundary.** A data contract rejects bad input immediately rather than three agents downstream.
- **Wrap every agent call in a circuit breaker.** Fail fast when a dependency keeps failing, then probe for recovery.
- **Plan compensation, not just retries.** When a later step fails, Saga compensation reverses successfully completed steps in order.
- **Design for failure before production.** The value is a system that behaves predictably even when its agents do not.

## Detailed Notes

### One agent becomes a distributed system · 01:31
- One agent (LLM + prompts + maybe retrieval and tool calls) demos beautifully.
- Then product asks for five more. Now A produces data B needs, C waits on both A and B, D changes state B is reading, and E crashes mid-workflow.
- Bhaumik's point: "This is no longer an AI problem. This is a distributed system problem."
- Most engineers, he notes, did not sign up to be distributed systems engineers.
- Complexities that arise: coordination, consistency, and partial failure — none solved by a better model or prompt.

### A real failure: stale credit scores · 02:34
- A financial-services credit-decisioning system started with one credit-score agent that ran two weeks with zero issues.
- Four more agents were added: income verification, risk assessment, fraud detection, and final approval.
- Within three days, **20% of decisions had incorrect risk ratings** — customers who should have been flagged were approved. Diagnosis took two days.
- Cause: the credit-score agent wrote 750 to PostgreSQL, but 500 ms later the risk agent read 680 from a **cache that was not invalidated** for the same customer.
- The write succeeded; the read was stale. The defect lived between the agents and the database — several agents sharing a cache with no coordinated invalidation.
- Fixing prompts would not have helped. The lesson: "bad architecture, not bad AI," kills multi-agent projects.

### Choreography vs orchestration · 04:34
- The number of pairwise connections for `n` agents is `n(n-1)/2` — 0 for one agent, 1 for two, 10 for five. (Bhaumik describes growth as exponential and cites a 25-fold increase; the connection formula itself is quadratic.)
- **Choreography** is event-driven. A research agent publishes a "research completed" event; an analysis agent subscribes, works, and publishes "analysis ready"; a report agent consumes that. No central caller — agents are loosely coupled, and you can add subscribers without touching a central workflow.
- The cost moves to operations: when a report never appears, which agent failed to publish or consume? Choreography demands strong end-to-end tracing and delivery guarantees. Choosing it just because it "feels more agentic" leads to months of firefighting.
- **Orchestration** puts a coordinator in charge. It calls A and waits, runs B and C in parallel, waits for both, then calls D with their combined outputs. Agents never call each other; the orchestrator owns the execution graph, state, retries, and step logs.
- Bhaumik names LangGraph wired into the Databricks AI Agent Framework as one implementation, but says any workflow engine with DAGs and retries fits. He uses orchestration almost exclusively in financial services, where a failed decision must be reconstructed (which agent, what order, what data).
- Decision framework: simple workflow + high autonomy → choreography; complex workflow + low autonomy → orchestration; complex + high autonomy → choreography with Saga compensation.

### Immutable, versioned state · 11:28
- The naive approach is shared mutable state. If A and B both read a score of 680 and A writes 750 while B writes 720, last-write-wins loses A's update.
- Database protections (row locks, serializable isolation, `SELECT FOR UPDATE`) only help if the application uses them correctly; many teams ship default isolation and a race condition.
- The fix is **immutable state snapshots with versioned handoffs**. A produces version one and seals it; the orchestrator stores it as a new append-only row. B receives that exact version, validates its schema, and inserts version two rather than updating version one. C receives version two.
- If C fails, the workflow can return to a known input without reconstructing a record others have overwritten. History records what each agent received and produced.
- In code, a frozen `AgentState` dataclass holds a `version`, a payload (also frozen), and a `created_by`. A `handoff` function validates, increments the version, and passes immutable input. Note that `frozen=True` blocks attribute reassignment but does not recursively freeze a dict or list.
- Debugging gets easier: if version seven is bad, inspect version six, then five — you can binary-search the history to find where state first diverged.

### Data contracts at the handoff · 15:23
- Immutability preserves an input; a **data contract** decides whether the next agent should accept it.
- The research agent promises findings, a confidence score, sources, and a timestamp; the analysis agent declares its requirements and validates before working.
- In the example it rejects research with confidence below **0.7**. That is an acceptance rule, not proof the confidence score is calibrated.
- The rule belongs at the boundary, before analysis runs — so you find out immediately rather than after a garbage report three agents downstream.
- Registering input/output schemas in Unity Catalog lets an organization govern and version contracts centrally, but the receiving agent must still enforce its contract during execution.

### Circuit breakers · 16:24
- Agents will fail: the LLM times out, the API rate-limits, an agent crashes. A **circuit breaker** stops a repeatedly failing dependency from consuming resources on every later call.
- It is a state machine: **closed** (call normally; success resets failures, threshold opens), **open** (reject immediately), **half-open** (allow one test request after a cooldown; success closes, failure reopens).
- His example opens after five consecutive failures and waits 60 seconds before the half-open probe; a failed probe resets the cooldown.
- Opening the circuit still leaves a business choice: degrade to reduced functionality, use acceptable cached results, or alert a human.
- He recommends a breaker around every agent call, placed at the serving boundary (Model Serving or AI Gateway). Note that documented rate limits and per-request fallbacks do not by themselves implement the failure counter, cooldown, and half-open logic — that breaker behavior needs an explicit implementation.
- Log the open/close transitions in MLflow so an investigator can see when an agent began flaking out.

### Compensation (Saga) rollback · 18:58
- A breaker contains repeated calls but does not undo work already completed. That is what **Saga compensation** is for.
- Each agent exposes `execute` and `compensate`; the orchestrator records which steps succeeded. In the example, research caches findings, analysis writes a draft, then the execution agent fails — recovery deletes the analysis draft, then clears the cached research.
- Order is: execute A (record), execute B (record), execute C (may fail), then compensate B and A in reverse.
- Those two compensations restore the initial state only because the side effects are reversible. In general, recovery is application-specific: an external action may be irreversible, concurrent changes may need to survive, and compensation itself can fail.
- A Saga needs defined recovery actions — it is not a database rollback spanning arbitrary services. The `compensate` methods must encode the real undo operation, not just mark a step failed.

### The production blueprint on Databricks · 20:56
- The orchestrator holds the workflow engine, a state store of versions zero through N, and the observability layer. A returns version one; the orchestrator gives that same snapshot to B and C in parallel, stores their results as versions two and three, and calls D with the combined outputs. All coordination passes through the orchestrator.
- Agents never call each other, giving one place to inspect execution and initiate recovery. (Bhaumik mentions running around the clock across billions of transactions but supplies no measurement for that scale claim.)
- Mapping onto Databricks: LangGraph wired into Mosaic AI Agent Framework manages the graph and call order; agents are Unity Catalog functions (SQL or Python) or registered models; Model/Function Serving exposes them and is where retries, timeouts, rate limits, and call protection are enforced.
- Delta Lake stores customer data and state history. Each agent result becomes a new state row. Note: **append-only is an application policy**, not a property of Delta tables — Delta supports updates, deletes, and merges, so access controls must preserve it.
- Unity Catalog supplies governance (access control, lineage, audit); MLflow tracing gives per-agent execution records and evaluation (LLM judges, call metrics); Agent Bricks sits above as a packaged route to common multi-agent cases.
- Tracing needs instrumentation (e.g. `mlflow.langchain.autolog()`) plus app context linking calls to state versions and breaker transitions. LangGraph checkpoint recovery does **not** automatically undo external side effects — compensation is workflow logic you must implement.
- Final thought: scaling past one agent forces coordination, consistency, and partial-failure choices. Circuit breakers may earn no applause, but preventing a dependency failure from becoming a late-night incident is what people notice over time.

## Resources

- [LangGraph persistence](https://docs.langchain.com/oss/python/langgraph/persistence) — checkpoints and workflow recovery
- [Agent Bricks multi-agent supervisor](https://docs.databricks.com/aws/en/agents/agent-bricks/multi-agent-supervisor) — packaged coordination patterns
- [AI Gateway endpoint configuration](https://docs.databricks.com/aws/en/ai-gateway/configure-ai-gateway-endpoints) — serving-boundary protection
- [Trace LangGraph with MLflow](https://docs.databricks.com/aws/en/mlflow3/genai/tracing/integrations/langgraph) — per-agent tracing
- [Compensating transactions](https://learn.microsoft.com/en-us/azure/architecture/patterns/compensating-transaction) — Saga recovery design
- [Python frozen dataclasses](https://docs.python.org/3/library/dataclasses.html) — immutability reference
- [Delta Lake updates and merges](https://docs.delta.io/delta-update/) — why append-only is a policy, not a default
