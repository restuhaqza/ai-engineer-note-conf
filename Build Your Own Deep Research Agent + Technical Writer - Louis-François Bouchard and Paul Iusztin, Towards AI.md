# Build Your Own Deep Research Agent + Technical Writer

| Field | Details |
| --- | --- |
| **Speaker** | Louis-François Bouchard, Paul Iusztin, Samridhi Vaid |
| **Affiliation** | Towards AI |
| **Type** | Workshop |
| **Date** | 2026-04-20 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=mYSRn6PC1mc) |
| **Tags** | `creative-and-generative-media` `agent-engineering` `coding-and-developer-tools` `evals` `apis-mcp-and-protocols` `rag-context-and-search` |

## Summary

This workshop from Towards AI shows how to build two connected systems: a deep
research agent and a technical writer. Louis-François Bouchard (CTO) frames the
problem, Samridhi Vaid builds the research side, and Paul Iusztin builds the
writing and evaluation side. The starting point is a generic AI-written LinkedIn
post: full of "delve" clichés, vague claims, and outdated facts.

The core argument is that better generated writing needs better evidence and a
deliberate editorial process, not just cleaner punctuation. The research agent
gathers sources and writes a `research.md` artifact; the writer turns that plus a
human guideline and reusable profiles into a first draft, then runs a
reviewer/editor loop. Files carry the handoff so humans can inspect and revise.

The most valuable part is the evaluation. They label real writer outputs, train an
LLM judge to agree with a human, and measure that agreement with F1. A judge that
scored perfectly on five-sample splits still passed failing posts in a larger
simulated run, which showed the evaluation data was too thin.

## Key Takeaways

- **Fluency is not substance.** Remove the clichés and a generic post still lacks
  evidence and a real angle.
- **Match the mechanism to the task.** Use a prompt, workflow, or agent depending
  on how much the next step depends on what the system discovers. When decisions
  share context, keep one agent and expose specialists as tools.
- **Context is a budget.** Bouchard reports degradation around 200,000 tokens, far
  below advertised context windows.
- **Research and writing need different freedom.** Research must search again for
  missing evidence; writing must obey strict format and tone.
- **Guide what to say, profile how to say it.** A guideline sets the angle and
  points; reusable profiles set structure, terminology, and voice.
- **An LLM judge is a classifier you must evaluate.** Measure when it agrees with
  a human expert using precision, recall, and F1 — and beware perfect scores on
  small splits, which can hide failures a larger run exposes.

## Detailed Notes

### The problem: fluent AI content still needs research and editing · 00:32
- The opening slide is a typical AI-written LinkedIn post. Problems include:
  - Slop words like "delve" and "intricacies."
  - Unsupported generalizations ("most teams," "most AI projects").
  - Formulaic contrasts ("it's not about X, but Y").
  - Hallucinations and outdated facts: it calls GPT-4 state of the art, though the
    post was written the previous week.
- The biggest issue is shallowness: no real value. They built a system that
  researches topics like "what is harness engineering?" into articles with code.

### Choose the right amount of autonomy · 06:09
- AI engineering faces constraints ordinary software avoids: cost per task,
  latency, quality, and data privacy.
- Think of an **autonomy slider**: more autonomy means less control and higher
  cost. Most client requests are simple workflows, not full agents.
- A **workflow** is a model augmented with data, tools, and memory in a fixed
  structure: chaining prompts, routing by condition, parallel calls, majority
  voting, or judging loops.
- An **agent** reacts to its environment: it plans, picks tools (and decides which
  not to use), and interprets results.
- Pick the smallest mechanism that fits: prompt with examples when the model knows
  enough; supply (and cache) known context under ~200,000 tokens; retrieve on
  demand when evidence depends on the question; or route a workflow when steps
  vary by known conditions.
- Example: support-ticket handling is always the same six ordered steps, so a
  workflow beats an agent. An agent would only add overhead.

### Keep coupled decisions in one agent · 13:15
- Autonomy helps when the next action depends on what the system finds.
- A Canadian CRM client asked for a multi-agent marketing assistant. The real work
  was one tight sequence: plan, retrieve client data, generate content, validate,
  repair.
- Each decision needed the same client and content context. Splitting them across
  agents would create handoffs where information can disappear.
- The fix: keep **one decision maker** and implement specialists as tools. An SMS,
  email, or validation tool can hold its own instructions and even its own LLM
  calls.
- **A specialist capability does not require a separate autonomous planner.**

### Treat context as a budget · 16:15
- One planner concentrates context: instructions, tool definitions, examples,
  documents, and history all compete for attention.
- Bouchard reports degradation becoming pronounced around 200,000 tokens, well
  before a model's advertised million-token window. The threshold is not fixed.
- Fitting a whole book into a request does not mean the model integrates every
  relevant passage. (The related "Lost in the Middle" work concerns where
  information appears, not a universal token limit.)
- Keep active context lean with trimming, summarization, selective retrieval, and
  compaction. Delegating a bounded task to a tool or sub-agent keeps its working
  context separate. More than roughly 20 tools is a heuristic for considering
  delegation, not a hard cutoff; other reasons to split include independent
  decisions, different tool sets, or a security boundary.

### A research agent that hands off a focused artifact · 20:22
- A **deep research agent** gets an objective, not a list of searches. It plans,
  reads sources, finds gaps, pivots, and synthesizes a cited answer.
- Educational content needs both recall (enough evidence) and precision (no
  noise), plus runnable code and useful images. The business case is the cost of
  senior expertise and repeated editorial review.
- Research re-searches when evidence is missing; writing obeys strict vocabulary,
  format, and tone. They therefore separated the two.
- The handoff is a file, `research.md`. A basic script runs stages in sequence and
  shared files in one project carry the artifacts.
- **Files give humans a place to inspect or revise the work** and keep the
  relationship between stages visible.

### Separate the harness from the MCP server · 34:41
- Sources map to existing capabilities:
  - **Web pages:** Firecrawl handles supplied pages, including dynamic ones.
  - **Web questions:** Gemini grounded search returns focused answers with sources.
  - **YouTube:** URL-based video analysis replaces a download-and-transcribe step.
  - **Repositories:** a GitHub library turns material into prompt-friendly text.
- The stack uses Python, `uv`, and FastMCP. MCP tools (a standard for exposing
  tools to agents) expose capabilities; an MCP prompt gives a recipe for using
  them. They alternate Gemini Pro and Flash by task complexity.
- Claude Code is both the harness and the MCP client. Its model decides when
  research has gaps and which capability to call; a different compatible harness
  could take that role.
- Three MCP surfaces: **tools** execute actions (search, analyze video, compile
  research); **prompts** supply reusable instructions; **resources** expose
  readable data (model names, versions, config).
- Research uses three tools. The first two store results in `.memory` so evidence
  is inspectable; the compiler reads them and writes `research.md`. Opik
  provides tracing. Video runs are prompted to return timestamps, visual
  descriptions, speaker labels, and section summaries.
- Local setup uses an environment file for the Google API key plus an optional
  Opik key; `.mcp.json` launches the server as a subprocess over stdin/stdout.
- **Skills vs MCP:** a skill's name and description support discovery; its body
  loads the procedure when needed (progressive disclosure). Their skill replaces
  the workflow prompt; distribution is the broader reason to keep a server.

### Build the writer's context and its review loop · 1:10:57
- The writing workflow consumes `research.md` and `guideline.md`, adds reusable
  profiles and examples, and builds the generation context. It writes a first
  draft, then loops a reviewer and editor.
- **The guideline controls what to say:** the angle, key points, and narrative
  flow. This is the changing human input for each post.
- **Profiles control how to say it:** structure (length, hook, body, CTA),
  terminology (active voice, banned words like "delve" and "tapestry"), and
  character (the author's background, voice, and preferences).
- The writer also gets three of Iusztin's real posts as few-shot examples, chosen
  pragmatically: start with several, remove while output stays good, restore any
  that hurt. Every retained example costs tokens and latency.
- The **evaluator-optimizer loop** uses separate writer and reviewer contexts to
  avoid favoring text written in the same conversation. They keep every draft,
  because a later revision can satisfy more rules while reading worse, and use a
  small fixed iteration budget rather than a noisy score threshold.
- The reviewer returns structured Pydantic objects, not free text. Each issue
  names the profile or requirement, the location, and the correction. When
  suggestions conflict, priority is guideline, then research, then profiles.
- The writer has three entry points: a `make` command, an MCP interface, and a
  write-post skill that takes a directory path and writes the draft plus
  revisions there.
- **Why keep a server at all:** a code-bearing skill may force each user to
  install Python, configure tools, and supply credentials. Centralizing logic
  behind a server reduces per-user setup while skills personalize the invocation.

### Evaluate the writer with a calibrated judge · 1:37:00
- Reading one post is easy; reviewing a hundred after every prompt change is a
  different job. Evaluation serves optimization, regression testing, and live
  monitoring.
- Dataset construction keeps the real writer in the loop: derive a guideline from
  an existing post, run research on it, feed guideline and research to the writer,
  label the output pass/fail with a short critique, then split into training,
  development, and test sets.
- Synthetic help may create inputs, but outputs must come from the system being
  evaluated. Labeling practice: stop at the first failure, explain it in a couple
  of sentences, move on.
- The **LLM judge is a classifier that must itself be evaluated.** Its purpose is
  to agree with a domain expert, not just sound confident. Agreement is measured
  with **F1**, which combines precision and recall: F1 = 2PR / (P + R).
- Build the dataset, construct the judge, calibrate on the development split, then
  validate once on the held-out test split. Opik manages this, but the sequence is
  portable.
- **Read perfect scores carefully.** The judge scored F1 = 1.0 on both splits,
  with only five samples each. These measure judge agreement, not writer quality;
  Iusztin suggests at least 20–30 samples per split.
- Then it got worse: on a larger simulated online run, the judge passed every
  trace even though human labels marked failures. The small held-out sets missed
  real behavior. The evaluation layer proved useful precisely because its own
  failure was visible.
- Traces tie it together. A thread represents the whole interaction; traces
  descend into each operation, showing nested tool and LLM calls with model, cost,
  and latency. The final entry point, `/research-and-write`, runs research then
  writing, with both stages visible in Opik.

## Resources

- [Workshop repository](https://github.com/iusztinpaul/designing-real-world-ai-agents-workshop)
- [Anthropic: Building Effective AI Agents](https://www.anthropic.com/engineering/building-effective-agents)
- [Lost in the Middle: How Language Models Use Long Contexts](https://arxiv.org/abs/2307.03172)
