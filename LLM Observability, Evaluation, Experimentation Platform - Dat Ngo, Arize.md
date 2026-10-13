# LLM Observability, Evaluation, Experimentation Platform

| Field | Details |
| --- | --- |
| **Speaker** | Dat Ngo |
| **Affiliation** | Arize |
| **Type** | Talk |
| **Date** | 2026-06-07 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=JsCCrBF7F1g) |
| **Tags** | `enterprise` `coding-and-developer-tools` `rag-context-and-search` `data-and-model-adaptation` `observability-and-reliability` |

## Summary

Dat Ngo, an AI architect at Arize AI, answers a practical question: once you
have built an agent, how do you make it actually work in production? He frames
the answer as three linked jobs — **observability** (what happened inside the
agent), **evaluation** (turning that behavior into a quality signal), and
**experimentation** (testing whether a change really improves things).

Agents are nondeterministic, so a fix in one place can quietly cause two or
three regressions elsewhere. The feedback loop therefore needs evidence from
more than the single example that motivated a change. Ngo shows how traces and
spans record an agent's actual path, how different kinds of signals measure
quality, and how teams turn weak signals into repeatable experiments.

A recurring theme is collaboration: engineers build and automate, while domain
experts and product managers define what a good experience looks like. He closes
by describing how this work is moving out of dashboards and into coding agents,
with the long-term goal of automating the whole improvement cycle.

## Key Takeaways

- **Three jobs, one loop.** Observability shows what happened, evaluation turns
  it into signal, experimentation tests changes.
- **Telemetry is the audit record.** Source code does not reveal which path a
  nondeterministic agent took; a trace does.
- **Watch distributions, not just one run.** Aggregating traces shows which
  branches and loops real traffic takes.
- **Five flavors of signal.** LLM-as-a-judge, human feedback, golden datasets,
  deterministic checks, and business metrics each answer different questions.
- **Let experts define quality.** Nontechnical domain experts should shape prompts
  and pick meaningful evaluations; engineers automate the execution.
- **Match scope to the question.** Evaluation flavor and evaluation scope (span,
  multi-span, trajectory, session) are separate choices.
- **Not everything should be evaluated.** Each check costs something; find the
  minimum set that tells you the app behaves as intended.
- **Experiments prevent regressions.** Collect examples into a dataset, then vary
  prompts, models, or orchestration and re-test.

## Detailed Notes

### How do you make AI work? · 00:31
- Ngo works with large enterprises on observability, evaluation, and
  experimentation. He estimates he used between 100 billion and one trillion
  tokens in the previous year — personal context, not a benchmark.
- He describes the AI space as "software reimagined": the same patterns, a
  different flavor. It feels like magic but is really engineering.
- Three terms drive the talk: **observability** asks what is happening inside
  the harness or agent; **evaluation** asks how to derive signal from the system;
  **experimentation** covers how to improve in a nondeterministic world.
- The core warning: when you make what you think is a fix, you may create two or
  three regressions you did not expect. Improvement needs evidence beyond the
  motivating example.

### Record execution, then follow state across runs · 02:50
- Arize uses **OpenTelemetry** (a common standard for telemetry data) across
  models, frameworks, SDKs, and agent harnesses.
- For supported integrations, auto-instrumentation is described as adding one
  line that observes framework or SDK activity and produces traces and spans.
  That shorthand assumes the surrounding setup; custom instrumentation and export
  still need configuration.
- A trace is "the audit record" of what the agent did. Code does not audit an
  agent; the telemetry does.
- The demonstrated trace tree shows `Financial_Data_Agent` with nested LangGraph,
  model, and tool calls next to an input/output panel — hierarchy and data
  together.
- Traces are only one layer. **Sessions** capture back-and-forth conversation and
  state across runs. Ngo points to Anthropic's Managed Agents work as related
  reading.
- Enterprise stakeholders may care less about each tool call than whether the user
  was satisfied and all their questions were answered — outcome questions that
  need a wider view than one invocation.

### Find the path that fails · 04:55
- In Arize AX, the next step is to aggregate executions. One trace answers what
  happened once; a distributional view shows branches and loops across observed,
  instrumented runs.
- That view can reveal what share of traffic follows each branch, and whether a
  component on one branch adds significant latency. It reflects recorded
  behavior, not every theoretically possible path.
- Evaluation signals become more useful attached to trajectories. Suppose one
  route performs well and another consistently scores worse.
- His example finds an ordering error: the agent calls `B` before `A`, even
  though `B` depends on `A`. The calls alone do not explain success — their order
  matters.
- The proposed fix is to add context telling the model that `A` must run before
  `B`. That change still has to be tested; it is not proof the issue is resolved.
- Customizable real-time analytics round this out. Observability means moving
  between the single run, the conversation, the path distribution, and the
  operational picture — not treating a trace viewer as the whole system.

### Choose the signal that answers the question · 06:49
- **LLM-as-a-judge:** a model assesses quality, though making that dependable can
  be more complex than the basic idea suggests.
- **Human feedback:** end users and reviewers provide evidence about the
  experience the system delivers.
- **Golden datasets:** trusted domain labels establish judgments a model judge
  can be tuned to approximate.
- **Deterministic checks:** logic tests concrete properties such as valid JSON,
  schema compliance, and required non-null fields.
- **Business metrics:** outcomes track whether the app makes money, saves money,
  or saves time.
- These sources serve different purposes. A model judgment is not a substitute
  for every structural check or business outcome.

### Let domain experts define the intended experience · 08:44
- Evaluation brings two groups together. Engineers and developers know how to
  build frameworks and automate work.
- Subject matter experts and product managers know what the AI experience should
  be: which domain expectations matter, how prompts should express them, and
  which evaluations are meaningful.
- The division of labor should let engineers build while domain experts shape the
  behavior being assessed.
- Arize's interface supports this by letting nontechnical users pick an
  evaluation model, start from a template, or customize an evaluator. Technical
  users can attach and run evaluations programmatically.
- Both routes expose the workflow to the people who own the criteria and the
  people who run it.

### Match evaluation scope to the evidence it needs · 09:59
- Choosing an evaluation method is only part of the design; you also choose how
  much evidence it inspects.
- **Span scope:** one component's input and output. Example question: was this
  call's output appropriate?
- **Multi-span scope:** data from several components. Example: did agents pass
  data correctly?
- **Trajectory scope:** the execution sequence across spans. Example: were calls
  ordered to complete the process?
- **Session scope:** conversation and state across runs. Example: was the user
  frustrated or left with unanswered questions?
- A multi-span evaluation of agent handoffs needs data from all participating
  agents; one isolated output cannot show the exchange. **Evaluation flavor and
  scope are separate choices.**
- The goal is not to evaluate everything possible. Each check has a cost, so find
  the minimum set that gives enough signal. Granularity helps when it answers a
  question; exhaustiveness is not the aim.

### Turn weak signals into experiments · 12:11
- **Collect examples.** Find traces where the signal is poor or something is
  missing, then add those cases to a dataset. Without traces, upload
  input/output pairs directly.
- **Run experiments over the dataset.** Its rows are the examples against which
  changes are assessed.
- **Test changes to the system.** Vary prompts, models, orchestration, or
  configuration, through a UI or programmatically.
- This gives a proposed fix a broader test than the single example that prompted
  it — the practical answer to the earlier warning about regressions.

### Move the workflow into the coding agent, then choose your entry point · 13:16
- Dashboards make the process visible, but teams do not want to do every
  investigation by hand.
- Arize exposes platform primitives through a CLI, tools, and opinionated skills
  so Claude Code or Codex can work with them from the coding environment. The
  Arize AX CLI is one entry point; its current command surface may differ from
  what the recording shows.
- There is also a built-in AI system, **Alyx**. You can ask it an application-level
  question — whether anything is wrong — and, with access to telemetry and hooks,
  it plans and runs investigation tasks.
- The long-term ambition goes beyond automating existing buttons: Ngo wants trace
  context to inform which evaluations should exist, with changed behavior
  prompting new evaluations.
- During the demo, Alyx flags high latency and detected errors. Those are
  investigation findings; the recording does not show a completed autonomous
  fix-and-retest cycle. Automatic evaluation selection and a self-improving
  system remain the goal.
- Two entry points close the talk. **Arize Phoenix** is open source and
  engineering-first: it runs locally in a single container without Kubernetes.
  **Arize AX** targets large enterprises; Ngo names Uber, Booking, and Reddit as
  examples. Phoenix's local deployment is a way to start without a cluster.

## Resources

- [The Role of OpenTelemetry in LLM Observability](https://arize.com/blog/the-role-of-opentelemetry-in-llm-observability/) — instrumenting and exporting telemetry
- [Arize Phoenix](https://github.com/Arize-ai/phoenix) — open-source tracing, evaluation, datasets, and experiments
- Run Phoenix locally with Docker — instructions for a single-container local instance
- Instrument agent trajectories in Arize AX — span attributes that expose agent handoffs
- [Arize AX CLI](https://github.com/Arize-ai/arize-ax-cli/blob/main/README.md) — command-line access to traces, datasets, evaluators, and experiments
- [Alyx](https://arize.com/products/alyx/) — Arize's built-in agent for investigating and improving AI applications
- Scaling Managed Agents — Anthropic article on separating durable sessions from agent compute
