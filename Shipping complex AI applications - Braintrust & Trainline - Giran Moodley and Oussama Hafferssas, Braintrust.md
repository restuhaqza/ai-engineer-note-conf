# Shipping complex AI applications | Braintrust & Trainline

| Field | Details |
| --- | --- |
| **Speaker** | Giran Moodley, Oussama Hafferssas, Mayank Soni |
| **Affiliation** | Braintrust & Trainline |
| **Type** | Workshop |
| **Date** | 2026-05-01 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=ZdheJTfLu-s) |
| **Tags** | `apis-mcp-and-protocols` `developer-workflows-and-testing` `reasoning-and-models` `observability-and-reliability` `evals` `agent-engineering` |

## Summary

This hands-on workshop brings together Giran Moodley from Braintrust and Trainline
engineers Oussama Hafferssas and Mayank Soni. The problem they start from is simple:
a demo that answers plausibly is not proof that an application behaves across
customer inputs, company policies, and changing dependencies. Without a record of
what changed and a way to test the consequences, every prompt patch just adds
uncertainty. Giran's opening point is that the gap is rarely model intelligence —
it's operational rigor.

Trainline supplies the real-world case. It builds both classic ML (a disruption
predictor he calls a weather app for trains) and agentic applications (a travel
assistant that handles refunds, train changes, and hand-offs to humans). Agentic
systems mix deterministic and probabilistic parts, so Trainline combines software
tests with model evaluations — offline before release, online in production. Cost
and token pressure push them to swap models, and evaluations let them check a
cheaper model holds the same quality.

The hands-on half builds a support-triage app in stages: one model call, then local
tools, then five specialist stages, then tracing, a golden dataset, managed prompts,
and production scoring. The closing exercise takes a real failure — a calm email
hiding a blocked financial workflow — and walks through reproduce, add a test, tighten
the prompt, re-run, and check for regressions. The through-line: make every change
something you can inspect and evaluate.

## Key Takeaways

- **A demo proves one plausible answer; production demands consistent behavior**
  across inputs, policies, and dependencies.
- **Split a monolithic prompt into explicit stages** so a wrong classification, a
  bad reply, or a missing escalation can be diagnosed locally.
- **Trace the whole interaction as a nested tree:** one parent span per conversation,
  child spans per stage, model call, and tool.
- **An evaluation is data + a task + scorers.** Use deterministic checks where the
  rule is clear and LLM judges only for nuanced criteria.
- **Start with a small golden dataset** (the workshop seeds ten cases) and let real
  failures grow it.
- **Move prompts, tools, and scorers into managed infrastructure** for collaboration,
  but keep version control and know which configuration runs.
- **Online scoring on live traffic**, starting with a higher LLM-judge sampling rate
  and dropping to 5–10% once behavior is trusted.
- **Fix a failure by reproducing it first**, then adding a test before changing the
  prompt, and finally re-running the broader set for regressions.

## Detailed Notes

### What makes an AI demo ready to ship? · 00:26
- A few successful demos show an app *can* answer; they say nothing about how it
  behaves across customer inputs, policies, or changing dependencies.
- Giran names a familiar loop: ship a prototype, hit a production failure, patch the
  prompt, wait for the next failure.
- Logs record events, but observability should also expose the behavior and execution
  path that produced them.
- The operational unit is **a change you can inspect and evaluate**. Borrow the
  monolith-to-microservices idea: break one big prompt into explicit stages. The goal
  is progressive coverage of meaningful failure modes, not complete coverage.

### The feedback loop, and Trainline's case · 10:03
- Braintrust presents a shared workflow for instrumenting, evaluating, and scoring AI
  applications; Giran describes its data store, **Brainstore**, as built for changing,
  semi-structured trace data.
- Two ways into the same loop: a new app starts from an evaluation set that expresses
  required behavior; an existing app starts from instrumenting and collecting real
  failures. Either way: identify a failure mode, remediate, ship, and monitor.
- Trainline stitches fragmented rail carriers into one booking app. Its disruption
  predictor is the classic-ML example; the travel assistant (reached from a ticket
  view, handling refunds and train changes) is the agentic one.
- Agentic apps combine deterministic and probabilistic parts, so Trainline mixes
  software quality practices with model evaluations — offline before release, online
  in production.
- Evaluations cover tone of voice and real helpfulness, including ticket-specific
  reasoning; Mayank describes checking a candidate model offline, then confirming the
  effect survives production traffic. Shared access lets non-technical colleagues
  inspect data without asking an engineer for logs.

### Build a support-triage app, then add specialist stages · 25:15
- Environment: TypeScript, Node, and pnpm (with `mise` and `Make` as wrappers);
  participants need a Braintrust account and an OpenAI key.
- The app is a fictitious support-triage system — built for teaching, not production.
  A ticket flows through five stages: context collection, triage, policy review,
  reply writing, and finalization.
- Model calls concentrate in the specialist stages; a severe ticket can trigger a
  tool call for human escalation, and the result is structured for downstream systems.
- Checkpoint 1: a single function, one model call, structured output. It looks
  plausible but can't handle organization-specific exceptions.
- Checkpoint 2: add local **tools** for help-desk articles, account context, and
  escalation. Here they're deterministic code; real setups might use vector search,
  MCP (a standard way for agents to call tools), or a CLI. Each new dependency is
  another place to fail, so traces matter more.
- Checkpoint 3: split into the five **specialist stages**, each with its own prompt
  where needed — the purpose is local diagnosis.
- A live demo changes the ticket to a blocked Pro-to-Enterprise upgrade: triage flags
  it high priority for revenue impact and adds an escalation reason.

### Trace the full interaction as a nested tree · 46:11
- The final answer can't say which stage introduced a mistake or consumed the time,
  so instrumentation must capture the full path: stages, nested functions, model
  calls, and tools.
- Useful fields include latency, cost, input/output tokens, time to first token,
  inputs, outputs, and metadata; metadata makes traces filterable for monitoring.
- The design is language-independent: **one parent span represents an interaction or
  conversation; child spans represent the work inside it.** Recording calls
  separately loses the relationships.
- Setup needs a Braintrust API key (in `.env` or a vault) and an OpenAI key configured
  at organization or project scope.
- In the UI, the **tree** explains dependencies while the **waterfall timeline** shows
  which stage caused delay.

### Define good behavior, then make it repeatable · 56:35
- New applications hit a cold-start problem: no production history to learn from, so
  start by defining release criteria.
- For this agent: the category should be correct, blocking issues shouldn't get low
  severity, escalation should respect policy and SLAs, and output should be valid.
- An evaluation combines **data, a task, and scoring functions**. The golden dataset
  holds important expected behaviors and edge cases; running against them creates a
  repeatable baseline instead of anecdotal confidence.
- Scorer choice:
  | Scorer | Good for | Cost |
  | --- | --- | --- |
  | Deterministic code | category, schema, required fields | no model call |
  | LLM judge | nuanced style or response quality | extra model call |
- Keep simple requirements cheap and explicit; reserve an LLM judge for interpretation.
  `make seed_dataset` uploads ten cases with expected output and metadata, scored by
  deterministic checks plus an LLM-based customer rubric.

### Manage shared prompts without losing change control · 1:05:17
- Local files are fine for one engineer, but collaboration needs a shared reference,
  version history, change authorship, and reproducibility.
- A product manager or domain expert should be able to propose prompt changes without
  asking an engineer for every edit — Giran connects this to his regulated-banking
  experience.
- The managed checkpoint packages **prompts, tools, and scoring functions** and
  uploads them to Braintrust; the app references those managed assets while keeping
  its execution trace. Local mode is the default; managed mode changes the path.
- A **prompt slug** is a stable code reference, and interpolation supplies runtime
  values; versioning keeps the reference and the selected version separate.
- Giran changes the model in the UI, saves a version with a comment, and reruns setup
  to sync. A ticket run then follows the managed model selection with no code edit.
- His explicit warning: **managed editing still needs source-control discipline.**
  Keep version control and a centralized representation, with automation to sync.

### Score incoming traces, with deliberate sampling · 1:13:58
- Offline cases set an initial level of confidence; **online scoring** applies quality
  checks to the traffic the app actually receives.
- Automations select incoming logs and run managed scorers. Start with a higher
  LLM-judge sampling rate to establish a baseline, then reduce to **5–10%** once
  satisfied — cost guidance, not a quality threshold. Cheap deterministic checks can
  run on every eligible execution.
- A rule can operate on a single span or an entire trace, and metadata can restrict it
  to relevant executions. Two different automations: setup automation packages assets
  and bootstraps the environment; an online scoring rule runs against incoming logs.
- A harder question: what can a judge evaluate when production data has no ground
  truth? Giran suggests capturing the case into a dataset and replaying it through the
  playground. Dataset replay does not itself define what a correct judgment means.

### Reproduce the failure before changing the prompt · 1:19:06
- The final exercise gives a concrete priority failure: a customer says the issue
  isn't urgent, but their CFO can't export invoices before a board meeting. A model
  following the explicit urgency wording misses the business impact — calm language
  doesn't make a blocked financial workflow low priority.
- The repair procedure connects everything built earlier:
  1. Replay the failure so its execution can be inspected.
  2. Establish an evaluation for the behavior that should change.
  3. Tighten the relevant prompt.
  4. Rerun the targeted case.
  5. Rerun the broader test set to check for regressions.
- The failure and remediation states live on different branches, so the change itself
  can be compared rather than remembered.
- In managed mode, Giran replays five JSON regression cases (including the invoice
  scenario), runs the new evaluation, and compares with existing runs in Experiments.
- Publishing needs a workshop-specific step: set `BRAINTRUST_IF_EXISTS=replace` so
  updated assets replace the managed definitions; the UI then shows change authorship
  and the prompt diff.
- The revised prompt includes the tool facts the triage decision needs. Giran reports
  recovery in the experiment comparison and stresses the visible link to a specific
  change — not just a better-looking reply.
- Closing advice: treat production tracing as a baseline, start with one important
  workflow rather than a whole suite, and route each new real failure into the test
  set. The value is shortening the time between a failure and knowing whether a fix
  worked.

## Resources

- [Braintrust cookbook](https://github.com/braintrustdata/braintrust-cookbook)
  — examples for building evaluations and integrating SDK tracing.
- Braintrust docs: tracing quickstart, evaluation workflow, online scoring, managed
  prompts, and the CLI quickstart.
- [Trainline FY2026 results](https://www.trainlinegroup.com/investors/results-reports-and-presentations/full-year-results-fy2026/)
  — results defining net ticket sales and the 27-million active customers.
