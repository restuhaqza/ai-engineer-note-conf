# Teaching coding agents to use Langfuse

| Field | Details |
| --- | --- |
| **Speaker** | Marc Klingen |
| **Affiliation** | Langfuse, part of ClickHouse |
| **Type** | Talk |
| **Date** | 2026-05-20 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=vNCY9kXXyDQ) |
| **Tags** | `coding-and-developer-tools` `rag-context-and-search` `evals` `apis-mcp-and-protocols` |

## Summary

Marc Klingen, a co-founder of Langfuse, describes what his team learned while building an **agent skill** — packaged instructions a coding agent loads when it needs them — so that agents can add Langfuse to a project reliably. Langfuse provides tracing and evaluation infrastructure for LLM (large language model) applications, and its documentation had grown to 478 pages across five feature areas.

The old expectation was that users would read the docs and build a mental model. Now they expect a coding agent to do the integration. But agents carry stale knowledge: pretraining makes them confident in SDK calls and CLI flags that no longer exist. A plain request such as "add tracing to my agent" produced a loop of implement, fail, fetch current docs, repair — and the resulting trace still did not reveal much about what the agent was doing.

Klingen's skill splits responsibilities: a SKILL.md sets the interaction style (ask follow-up questions), progressively loaded references supply product knowledge, current documentation is fetched instead of remembered, and a CLI exposes actions that previously required clicking through the UI. He then walks through six lessons — starting with the reminder that reading real traces gets you most of the way, and ending with a warning that an incomplete optimization objective can quietly delete the very behavior that makes a skill useful.

## Key Takeaways

- **Skills add procedural guidance, not capability.** The agent already has tools; a skill tells it how to use them well.
- **Stale training data is the core problem.** Without fetching current docs, an agent confidently writes obsolete SDK code.
- **478 doc pages needed a navigation path.** The skill points the agent at an `llms.txt` sitemap and asks for Markdown instead of HTML.
- **Four-part skill design:** interaction guidance, progressive product references, current documentation, and a CLI for actions.
- **Traces are the feedback loop.** Watching real runs shows what guidance is missing or inefficient — Klingen's informal estimate is that traces give ~80% of the detail.
- **Reuse the RAG search endpoint.** It answers agent questions directly and, through logged queries, shows maintainers where docs are missing.
- **Start evals small.** Five examples with an LLM-as-a-Judge (a model grading results against plain-language checks) made changes testable.
- **The objective defines what survives.** Optimizing for fewer turns made the agent drop documentation fetches — the exact behavior the skill exists to enforce.

## Detailed Notes

### Skills as guidance, and traces as the feedback loop · 00:16
- Klingen's analogy: a child can turn a Rubik's Cube's faces but cannot solve it. A bash tool is the same — power without a procedure.
- **A skill supplies procedural guidance to an agent that already has tools.** The manual is what makes the capability useful.
- Skills sit between rigid workflows and fully autonomous agents. Not every application needs a coding agent; flexibility can add latency and cost.
- Example: a password-reset workflow routes reliably, but breaks down when a user also wants to change their email. An agent can load the relevant guidance progressively and handle the combination.
- Open-ended input means you cannot predict every request. Traces of real executions reveal the skills you never knew you needed — and show when existing skills are outdated or inefficient.

### Flexible infrastructure makes integration a reasoning task · 04:01
- Langfuse had 478 documentation pages across five feature areas. That breadth is deliberate: Langfuse is unopinionated infrastructure, adaptable to many applications, and Klingen says its tracing handles billions of ingested traces.
- Flexibility used to push work onto the person integrating it. With coding agents, that flips: the infrastructure stays general while the agent adapts it to each repository.
- The catch: the agent must figure out which setup is appropriate. Knowing that Langfuse exists is not enough.
- Pretraining is awkward here. Early on, an agent producing Langfuse SDK code felt like a milestone. Later, the same familiarity produces methods that have since disappeared.
- **Current documentation must beat remembered SDK usage.** The failure mode is not ignorance — it is confidence in an obsolete interface.

### What the baseline Claude Code integration got wrong · 05:49
- Baseline request: "Add tracing to my agent." Claude Code implemented instrumentation from outdated training knowledge, failed verification, and only then fetched current docs to fix it — implement, fail, retrieve, repair.
- There was a requirements problem too. Online evals, offline evals, and human-in-the-loop evals serve different purposes, and nobody had told the agent to help choose. The resulting trace captured two LLM calls and little else about the agent's behavior.
- The goal of the Langfuse skill: give thousands of community teams and cloud customers the help an expert would provide, without individual consultations.
- Skill architecture:
  - **Interaction guidance:** `SKILL.md` says how the agent should work — including asking follow-up questions before deciding.
  - **Product references:** module-specific material, disclosed progressively, for the selected use case.
  - **Current documentation:** retrieved at run time instead of relying on model memory.
  - **Execution access:** the Langfuse CLI wraps existing APIs so an agent can do what humans previously did in the UI.
- In short: the skill gives direction, references give context, the CLI gives actions.

### Lesson 1: inspect traces before building elaborate evals · 09:04
- The team instrumented Claude Code, used it interactively with Langfuse, and read the traces to see where the agent wandered or made assumptions.
- Failure one: a human-friendly default. Langfuse had defaulted the data region to Europe to reduce setup variables. US enterprises also care about data regions. The skill now tells the agent to determine the user's region instead of assuming.
- Failure two: hallucinated CLI parameters. Seeing the word `trace`, an agent inferred flags from other tracing tools. The fix was to advertise the help flag more aggressively — one extra, fast turn to learn the real command surface.
- Klingen's informal estimate: inspecting traces gives you about 80% of the detail you need (his number, not a measured score).

### Lesson 2: make docs easy to navigate, and learn from searches · 10:46
- With hundreds of pages, an agent can burn many turns fetching, reading, discovering links, and fetching again.
- Fix: point the agent at the documentation index (`llms.txt`) as a sitemap, and tell it to request Markdown (via a `.md` suffix or an `Accept: text/markdown` header) so it does not parse HTML and waste tokens.
- The skill therefore explains **how to acquire information** rather than duplicating every page.
- Lesson 3 reused existing infrastructure: the docs Q&A agent's RAG (retrieval-augmented generation) stack was exposed as a search endpoint. The agent submits a natural-language question and gets documentation chunks back.
- The endpoint works in both directions:
  - The agent avoids a chain of page fetches.
  - Maintainers see the queries — intent that a bare page request would hide — and learn where documentation is missing.

### Lesson 4: a small evaluation suite beats none · 12:29
- Why evaluation felt hard: Langfuse users build chat apps, real-time voice, video generation, and background invoice processing. There is no single ideal instrumentation setup.
- The team wrote just five examples, each pairing an instrumentation request with a sample repository folder.
- Checks were plain-language statements graded by an **LLM-as-a-Judge** against the filesystem and before/after diffs. Example: for an OpenAI app with custom RAG functions, expect OpenAI instrumentation *and* retrieval spans — capturing only LLM calls would leave retrieval unexplained.
- This let the team change the skill and look for regressions without building a full evaluation framework.

### Lessons 5–6: reference dynamic content, then experiment · 13:52
- Lesson 5: resist copying documentation into the skill. Contributors love adding context, but every copy becomes another representation of the product that can drift. The stale-knowledge problem reappears inside the skill itself. Reference the docs instead.
- Lesson 6: apply automated research to improve the skill. The target task was migrating prompts from local Git repos into Langfuse prompt management, so product managers can edit prompts in a playground.
- Result: the team accepted three of six suggested improvements — useful for a small team exploring more alternatives than it could try by hand.
- But the outcome depended heavily on the target function. See next section.

### An incomplete objective removes valuable work · 15:13
- Prompt migration was assumed to be "fast," measured in number of turns. Optimizing for fewer turns removed the instructions to fetch documentation — the optimizing agent believed it already knew prompt management. That negates the point of the skill: current context.
- The intended workflow had an approval gate (propose a plan before uploading prompts, since user data leaves the laptop). The sandbox could not exercise that interaction, so the experiment could not test it.
- The desired end state included linking prompt versions to production traces. Because the objective did not reward that linkage, instructions encouraging it were dropped.
- **The target function defines which work the optimizer has a reason to preserve.** Trace inspection, search signals, evals, and canonical references are complementary; none fixes an incomplete objective.

### After installation: success criteria and distribution · 17:23
- An installed skill becomes a local copy on a user's machine. There is no package manager telling the user it is outdated a year later.
- Ideas under discussion: stamp the skill with the date it was fetched and have the agent re-fetch when it is old, or warn the user. But installing or updating a skill may need user action depending on the agent environment.
- Product question: should the skill aim for a quick first success, or the perfect setup (which can take an AI engineering team months without a skill)? Klingen leans toward getting something useful working, then letting the user invoke the skill again to improve it — rather than overwhelming them with questions up front.
- In an audience Q&A, Klingen confirmed humans review every change to the skill; autonomous experiments only generate candidates. On distribution, he was wary of maintaining separate integrations for Anthropic, OpenAI, and Cursor, favoring simple discovery: a well-known place an agent can find the skill.
- A trust question remains open: if skills carry the same trust as public web content, does discovery need an approval step? Klingen framed this as a question about the agent environment's trust model, not a settled rule.
- Current practice: the skill is now the advertised way to get started — users are told to ask their coding agent to do the integration.

## Resources

- [Langfuse Skills](https://github.com/langfuse/skills) — including the current [`SKILL.md`](https://github.com/langfuse/skills/blob/main/skills/langfuse/SKILL.md)
- [Langfuse CLI](https://github.com/langfuse/langfuse-cli)
- [Langfuse documentation index](https://langfuse.com/llms.txt)