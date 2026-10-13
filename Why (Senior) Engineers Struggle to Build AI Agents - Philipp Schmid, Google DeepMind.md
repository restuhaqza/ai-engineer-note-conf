# Why (Senior) Engineers Struggle to Build AI Agents

| Field | Details |
| --- | --- |
| **Speaker** | Philipp Schmid |
| **Affiliation** | Google DeepMind |
| **Type** | Lightning talk |
| **Date** | 2026-05-30 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=3_gYbhABcAE) |
| **Tags** | `evals` `apis-mcp-and-protocols` `coding-and-developer-tools` `rag-context-and-search` `agent-engineering` |

## Summary

Traditional software is largely deterministic: write a spec, implement it, test it, deploy it. Philipp Schmid of Google DeepMind argues that building agents breaks several of those assumptions, and that experienced engineers feel the friction most because their habits come from that older world.

He lays out five shifts engineers must make. The underlying thread: we are moving from controlling every step to **setting a goal and letting the model choose the route**, then verifying the outcome rather than one fixed path.

The talk is a lightning talk, so it is a tour rather than a deep dive. Each shift is illustrated with a short, concrete example, and Schmid points to a companion article for code.

## Key Takeaways

- **From traffic controller to dispatcher:** specify the destination, not every step the agent takes.
- **Text is the new state.** Natural-language context carries meaning that flags and booleans cannot — so structured state needs to be rich enough to hold it.
- **Intent can change mid-conversation,** so an agent should follow the updated meaning rather than stay locked to an early classification.
- **Treat errors as inputs,** like Go error values, so a failed tool call informs the next step instead of restarting a long run.
- **Move from unit tests to evals:** agents are non-deterministic, so measure how *often* a task succeeds, using LLM judges or human experts.
- **Grade the output, not the path.** A longer route with more steps can still be the correct answer.
- **Make APIs and tools self-documenting.** Semantic names, described parameters, and stated failure behavior — because agents never see the code or your years of context.
- **Build to delete.** Better models will let you replace today's implementations, echoing the Bitter Lesson.

## Detailed Notes

### From prescribed steps to delegated goals · 00:15
- Schmid introduces himself as working on agents with **Gemini** and the **Gemini API** at DeepMind.
- The old sequence: write a spec or **PRD**, implement, test, deploy, let users use it.
- Agent development folds observation into the loop: define instructions, run the agent, **inspect behavior**, adjust prompts or tools, run again.
- You are improving the system's ability to reach an outcome, including behavior you never explicitly wrote.
- His analogy: a **traffic controller** manages lights, speed, and routes; a **dispatcher** specifies a destination and lets the traveler choose how to get there.
- The goal stays fixed while the route can vary — a coding agent may take odd intermediate steps and still succeed.

### Text as state · 02:05
- A research agent returns a plan. In older software you might have had accept/deny buttons — a **Boolean decision**.
- But a user may want to approve *and* adjust scope at once: approve the market research, focus on the US, exclude California.
- The language carries two meanings: permission to proceed **and** a constraint on the work.
- If the interface cannot hold both, the user must reject, answer a follow-up, and wait for a new plan.
- A better state keeps the approval together with the extra instruction, and does not treat approval as proof the research already happened.
- **Personalization has the same problem:** a user may prefer Celsius generally but Fahrenheit for cooking.
- A single `use_fahrenheit` flag loses the conditional; a coarse `is_europe` flag cannot supply it. Structured storage can hold preferences — the mistake is reducing meaning to a structure too narrow to express it.

### Let intent change during the conversation · 03:42
- Context is not only text: images, video, and audio can also shape what the agent should do.
- Classic support pipeline: a cancellation request goes through an **intent classifier**, gets a churn label, and enters a predefined retention or cancellation workflow.
- But suppose the agent offers an alternative, the user accepts, and changes their mind about canceling — the first label no longer describes the goal.
- The agent must respond to the **updated meaning**, not keep following the cancellation path because that was the first classification.
- Enumerating every offer, response, and change of intent as a stateful workflow is hard, so delegate some control to let the next action follow the conversation.

### Return errors to the agent · 04:53
- Handing over control means leaving a purely deterministic workflow — and it changes how failures enter the flow.
- **An error is another input the agent can reason about**, not a stop signal.
- He points to **Go** as an analogy: functions commonly return a result alongside an error value, and the caller checks and handles it explicitly.
- Cheap retries (a small HTTP request, a product search) are fine. **Restarting a long agent run is not:** repeating five or fifteen minutes of work wastes compute and may discard useful context.
- A small adapter can expose both outcomes, e.g. a search observation that carries `Results` and an optional `Error` back into the agent loop.
- Returning an error as data does **not** repair the failure — recovery still needs a decision. The point is to continue from the current state instead of automatically restarting.

### Trace the route, grade the outcome · 06:13
- Classic tests assume a deterministic link: input A through code B yields output C.
- Agents are **non-deterministic**: the same input can produce different steps and results, so test how *often* it succeeds.
- His illustration: a customer agent that works only one time in ten is too flaky to release (an illustration, not a measured benchmark or a universal threshold).
- Success is also task-dependent: a research report and a customer-feedback reply need different **qualitative criteria**, judged by an **LLM judge** or a human expert.
- **Trace what the agent does** so you can inspect its behavior, but **grade whether the output satisfies the request**.
- One user's task might legitimately need four more research steps and more tokens; a longer route can still be the successful one, so matching one expected sequence would miss it.

### Make the tool contract carry the meaning · 07:47
- Agents evolve, but existing APIs keep assumptions built for their human developers.
- A `delete_item(id)` endpoint feels obvious to someone who has worked on it for years; the agent has none of that context.
- Initially the agent sees only **function schemas, docstrings, and tool definitions** — not the code behind them, and not your accumulated knowledge.
- Those exposed surfaces must carry what is needed to choose and use the operation:
  - **Semantic names** that identify the operation and the kind of object it acts on.
  - **Parameter descriptions** that explain what an identifier refers to, instead of just labeling it `id`.
  - **Failure behavior** describing what the caller receives when the operation cannot complete.
- The interface should be understandable **without** the experience of the team that built it.
- Delegating tool selection does not remove the application's job of **validating an operation** before executing it.

### Choose what reliability must mean before release · 09:04
- Trusting the model and verifying its work are **complementary**, not opposites.
- Avoid forcing every request through one fixed sequence just because that sequence is familiar.
- Preserve the meaning in the context and provide a way to **recover** when something goes wrong.
- Longer runs make recovery especially important: imperfect models can hit surprises well after useful work has built up.
- Release becomes an explicit reliability judgment: how often must the agent succeed for the user-facing task?
- Schmid gives **no single percentage** that makes an agent ready — choosing the acceptable level is part of engineering the product.

### Build to delete · 09:53
- Schmid invokes **The Bitter Lesson** (Rich Sutton's essay arguing that methods which scale with computation tend to win).
- Better models and agents will lead engineers to **rebuild the same capabilities** repeatedly.
- **Build to delete** is his engineering interpretation: expect some of today's software to become disposable as capabilities improve.
- He points to his companion article for a deeper treatment with code examples.
- The enduring responsibility: make the current agent useful and reliable, while staying willing to replace the implementation that got you there.

## Resources

- [Why (Senior) Engineers Struggle to Build AI Agents](https://www.philschmid.de/why-engineers-struggle-building-agents) — Schmid's companion article with deeper treatment and code.
- Gemini API function calling — docs for defining tools and handling function calls.
- [The Bitter Lesson](https://www.cs.utexas.edu/~eunsol/courses/data/bitter_lesson.pdf) — Rich Sutton's 2019 essay on scaling with computation.
- Errors are values — Rob Pike on organizing Go programs around error values.
