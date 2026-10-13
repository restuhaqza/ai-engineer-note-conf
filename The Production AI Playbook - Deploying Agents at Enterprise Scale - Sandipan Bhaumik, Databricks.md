# The Production AI Playbook: Deploying Agents at Enterprise Scale

| Field | Details |
| --- | --- |
| **Speaker** | Sandipan Bhaumik |
| **Affiliation** | Databricks |
| **Type** | Talk |
| **Date** | 2026-06-18 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=ObTPqBGsEbA) |
| **Tags** | `evals` `safety-and-governance` `infrastructure-and-deployment` `observability-and-reliability` `finance` |

## Summary

Most enterprise AI projects start in the wrong place: choosing a model before deciding how to tell whether the application works. Sandipan Bhaumik, a Data and AI technical lead at Databricks (previously five years as an AWS Principal Architect), has watched this pattern repeatedly. A team picks GPT or Claude, builds features against clean demo data, wins leadership sign-off, and then real customers get answers that do not match the demo. The money bought a demonstration, not a system.

He attributes the failure to three gaps — observability, evaluation, and governance — and turns them into a five-pillar framework: evaluation, observability, data foundation, orchestration, and governance. His rule is to think about all five before building, then build them gradually. The most important is evaluation, which he calls the specification: it defines success in numbers, such as how many simple queries the bot should deflect from human agents.

A retail-banking chatbot case study makes it concrete. The bank had about 20,000 calls a month, roughly 60% of them simple questions. An earlier proof of concept spent about 85K over six months and failed. The rebuild deferred model selection until week seven of an eight-week POC — instead building an evaluation dataset from 200 real human-support cases, setting targets (about 85% accuracy and 60% deflection), and putting tracing in place first. After launch it reported 87% accuracy, 62% deflection, a 65% drop in response time, and a customer-satisfaction score of 4.4/5.

The most instructive moment came after launch. The bank changed interest-rate policies and told customers by email and app, but the chatbot gave stale answers. Negative feedback surfaced the drop, and traces led to the root cause: the new policy document's embeddings had not reached the vector database. The fix was at the data layer, and it was only visible because the measurement and tracing systems already existed. Bhaumik closes with a production incident playbook and small, practical first steps.

## Key Takeaways

- **Demos fail because teams pick a model before defining success.** Evaluation should come before code, features, and model debates.
- **Five pillars:** evaluation, observability, data foundation, orchestration, governance. Consider all of them upfront, then build gradually.
- **Evaluation is the specification.** Define success in numbers tied to a business outcome (like deflect), not a vague sense of accuracy.
- **Evaluate in three layers:** deterministic checks (formats, entities, PII), semantic checks (LLM judges for safety, relevance, groundedness), and behavioral checks (tool calls, loops, retries).
- **Tracing is a must-have.** In many regulated settings you cannot onboard AI without it, and it also supports debugging and dispute resolution.
- **The data foundation is the biggest job.** Bhaumik says it takes about 60% of his project time; data built for forgiving humans gets consumed confidently by agents.
- **Treat prompts as code and models as swappable.** Version prompts with reasons, and test model upgrades on your own evaluation dataset.
- **Plan for incidents.** Detect → diagnose → contain → fix → preserve, and connect the playbook to your existing ITSM (IT service management) alerting.

## Detailed Notes

### Why demos fail in production · 00:15
- The recurring pattern: leadership pressure to "do something with AI," so every conversation starts with "should we use GPT or Claude?"
- The team picks a model, builds features against predictable data and limited scenarios, and it looks great. Leadership signs off and it ships.
- Weeks later, real customer questions produce answers unlike the demo. Recovery is hard, and the effort never scales — the investment bought a demonstration.
- Bhaumik reduces the problem to three gaps: **observability** (cannot trace decisions), **evaluation** (no defined business outcome measured continuously), and **governance** (unclear who owns failures and the data).
- He developed the framework from these gaps and reports applying it across multiple customer organizations.

### Five pillars to design before coding · 04:52
- The five responsibilities — **evaluation, observability, data foundation, orchestration, governance** — should all be considered before starting, then built gradually. The order is guidance, not a rigid schedule.
- **Evaluation:** what counts as success, and how do we measure it continuously?
- **Observability:** which operations produced this response? He describes tracing as an onboarding requirement in many regulated organizations he sees, including in Europe (not a blanket legal rule for every AI application).
- **Data foundation:** two parts — *question data* (what the agent needs to answer, from training or APIs) and *tracking data* (what the system did, which needs its own collection, storage, and serving strategy).
- **Orchestration:** one agent is simple; five bring coordination, waiting on responses, and the need for explicit patterns.
- **Governance:** accountability, data security, and protection against prompt injection that could cause harm or reputational damage.

### Evaluation is the specification · 07:34
- "Answer accurately" is not a spec. A banking chatbot needs numbers: acceptable accuracy, tolerable false positives, and how many simple queries the AI should handle without a human.
- **Deflection** ties the chatbot to a business goal — reducing human workload for questions it can safely resolve — and needs a target plus tracking.
- Build the dataset with domain experts: capture the answers human agents actually give, including ambiguous questions, gray areas, and edge cases.
- Automate the comparison: send a question, capture the AI response, score it against the expected answer, then extend that same pipeline to live production responses.
- Three evaluation layers: (1) **deterministic and classic ML** — regex for formats, plus models for entities, intent, and PII (personal information); (2) **semantic** — LLM judges scoring safety, relevance, and groundedness; (3) **behavioral** — tool choice, loops, duplicate calls, and retries.
- The account-balance example: the answer is correct, but the agent makes three database calls to get it. At thousands of daily queries, repeated work is a real cost that answer-only evaluation misses.

### Tracing: an overdraft-fee dispute · 12:25
- A customer disputes an overdraft fee and asks the chatbot to waive it. A simplified trace shows the steps: classify intent (with confidence and duration), retrieve the account through an API to the customer database, retrieve the overdraft policy from a RAG (retrieval-augmented generation) vector database, reason over account and policy, run final guardrails, then respond.
- The value: when the customer disputes the answer, the team can see exactly which account data and policy documents the agent used. Without that, support is reduced to making an uninformed concession.
- Traces also reveal duplicate calls, turning an unexplained cost or delay into a specific operation that can be corrected.
- Online monitoring connects this to a recovery policy: his example allows at most three retries, then reports the failure or hands it to a human. The application must enforce that control path; automatic trace scoring alone does not stop a tool call.
- This distinction matters for background LLM judges, which can score a trace asynchronously while the application keeps running.

### The data foundation · 15:05
- He estimates data foundation takes about **60% of his time** on typical projects. Enterprise data was organized for humans, who notice a suspicious report and ask someone to fix it; an agent will consume the same error and confidently pass it into a customer response.
- Question data and tracking data need separate plans. Tracking data serves auditors, regulators, online monitoring, and LLM judges, so trace schemas, collection, and serving cannot stay incidental logging once many teams depend on them.
- Databricks layering: **Apache Spark**, **MLflow**, and **Delta Lake** over cloud storage on Google Cloud, AWS, or Azure. Delta Lake adds database-like behavior and incremental management over stored files.
- **Unity Catalog** adds centralized permissions, sharing, discovery, ownership, and metadata. Table and column descriptions plus PII tags give AI usable context when it queries governed data.
- Above the foundation sit Mosaic AI for model and application work, data warehousing, BI (business intelligence) reporting, and Genie for natural-language SQL.
- The same foundation collects traces from mixed frameworks (for example LangChain) and multiple clouds, giving operational teams one shared source of evidence — health dashboards, Genie text-to-SQL, and custom Databricks apps. Agent Bricks and MLflow support automated judges and drift monitoring.

### Orchestration and governance · 20:05
- **Orchestrator-worker:** one central orchestrator routes work to specialist agents. Every request passes through it, so its logs are a natural place to diagnose failures.
- **Choreography:** autonomous agents subscribe to events on a message bus. Independent work runs in parallel (his mortgage example: one agent inspects customer details, another approval details), which can cut latency by avoiding orchestrator round trips.
- **Human-in-the-loop:** a threshold crossing or low confidence brings a person into the workflow to inspect the agent's work and decide what happens next.
- The pattern is not the whole architecture — shared state, fault tolerance, and scaling still need explicit treatment (he covers these in his companion talk "From Chaos to Choreography").
- Governance goes beyond data permissions: audit trails for actions, user connections, and requests, plus pre-validation of personal information. He reports detecting **47 PII breaches** during the customer's testing phase.
- **Treat prompts as code** under change management — a prompt edit needs review and release discipline, not just a Git commit. Model upgrades need the same care: a provider's public benchmark results do not tell you whether a new model works on *your* data, so compare candidates on your own evaluation dataset and keep the ability to switch.
- He presents **Agent Bricks** as Databricks' effort to make the five pillars easier to implement together.

### The banking case study · 24:36
- The bank had roughly **20,000 customer calls per month**, and about **60%** were simple questions (balances, overdrafts). It wanted to reduce reliance on human agents for those.
- An earlier proof of concept consumed about **85K over six months** without succeeding (currency unspecified). The team lacked visibility into failures, a reliable success measure, and clear accountability.
- New goal: the AI handles the simple queries making up 60% of volume. In an eight-week POC, they **deferred model selection until week seven**.
- Weeks 1-2: collected **200 cases** from real human-support interactions and defined success criteria — about **85% accuracy**, plus deflection and latency targets (these were goals, not achieved results).
- Then an automated feedback loop: capture question and response, compare with the evaluation dataset and rate, send below-threshold cases to a human, fix the cause (prompt or tool-calling), and add the failure to the dataset so future runs catch it.
- **The evaluation dataset is a living system** — the initial collection is a starting point, not a fixed size or a finished asset.
- Foundation work overlapped: checking database access, security, traceability, distributed storage, and trace collection in week two. This implementation used direct API calls to databases, not MCP.
- Weeks 7-8: model selection became an empirical comparison — run candidates on the same questions, compare with expected answers, and compute accuracy, which shortened the earlier open-ended debate. Production launch followed once observability and evaluation were in place.

### From incident to a living eval system · 29:18
- Six-week post-launch results: **87% accuracy, 62% deflection, a 65% reduction in response time, and CSAT (customer satisfaction) of 4.4/5**. The talk does not give scoring definitions, sample sizes, or the response-time baseline.
- Then a policy change exposed a data bug. The bank changed interest-rate policies and told customers by email and app, but the chatbot gave incorrect follow-up answers. Thumbs-down feedback revealed a drop in satisfaction.
- The measurement system caught the decline, and traces led to the documents used for retrieval. The agent was reading an **outdated policy** — the new document's embeddings had not reached the vector database. They fixed the update path, so the failure was actionable at the data layer instead of an unexplained quality dip.
- **Production incident playbook:** detect (evaluation dashboards) → diagnose (traces) → contain (for example, withdraw a bad prompt version or divert work to humans) → fix (judge reports and evaluation results) → preserve (add the failure as a regression case). For harder recovery he points to saga, compensation, and circuit-breaker patterns from his companion talk.
- The playbook must connect to existing **ITSM** (IT service management) systems, so the right person is alerted — detecting a failure is useless if no one is told.
- **Start small tomorrow:** define success in business terms, collect examples of good answers, and automate a comparison in Python. The essay's `evaluate_balance` example scores answer correctness and repeated tool calls separately, so a correct answer cannot hide wasted work.
- Keep history useful and evaluations affordable: record *why* each prompt changed (and the failure it fixed), not just the text diff; give the growing test library an owner and categories (for example, security); and run a small subset of the evaluation in continuous integration, reserving the full suite for merges to `main`.

## Resources

- [MLflow production tracing and monitoring](https://mlflow.org/docs/latest/genai/tracing/prod-tracing) — running custom LLM judges on traces
- [Delta Lake FAQ](https://docs.delta.io/delta-faq/) — transactional tables, Parquet files, and the transaction log
- [Unity Catalog documentation](https://docs.databricks.com/aws/en/data-governance/unity-catalog/) — governed assets, permissions, and metadata
- [Agent Bricks and governed enterprise data](https://developers.databricks.com/perspectives/databricks-data-intelligence-platform-connects-agent-bricks-with-governed-internal-data) — how Databricks components divide agent deployment
- [From Chaos to Choreography](https://ai.engineer/talks/from-chaos-to-choreography-multi-agent-orchestration-patterns-that-actually-work-sandipan-bhaumi) — the companion talk on multi-agent coordination