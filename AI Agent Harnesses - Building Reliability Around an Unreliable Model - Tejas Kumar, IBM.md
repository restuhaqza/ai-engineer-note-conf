# AI Agent Harnesses: Building Reliability Around an Unreliable Model

| Field | Details |
| --- | --- |
| **Speaker** | Tejas Kumar |
| **Affiliation** | IBM |
| **Type** | Talk |
| **Date** | 2026-05-17 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=C_GG5g38vLU) |
| **Tags** | `agent-engineering` `safety-and-governance` `architecture` `reasoning-and-models` |

## Summary

Tejas Kumar, an AI developer advocate at IBM, sets out to demystify the word "harness." He notes it means different things in machine learning (a test suite that runs inputs through models and scores outputs) and in AI engineering (the runtime infrastructure around an acting agent). His subject is the **agent harness**: everything around the model that grounds its behavior in a stable, controlled environment — the anchor that keeps a climber from drifting too far.

Why does it matter? Because builders rent inference, get limited context windows, and work with black-box, nondeterministic models. The harness is how you get reliability anyway. Kumar lists its usual parts: a tool registry, a model, context-management primitives, guardrails, an agent loop, and a verification step. Crucially, he argues a harness is not just the agent loop — the loop is one component, and an outer harness loop can manage repeated runs of an inner one.

He then demonstrates by building a minimal browser agent with a deliberately old model (GPT-3.5 Turbo), leaving both the task prompt and system prompt untouched. The agent is told to visit Hacker News and upvote the first post. It hits a login wall but reports success. Kumar fixes this step by step through runtime engineering: guardrails (max iterations, max messages), a naive context compressor, a verifier that checks tool history instead of trusting the model, and a login handler that authenticates outside the model. The outcome changes without touching the prompt.

He closes with where harness engineering leads: making cheaper models useful, IBM's open-source OpenRAG for enterprise retrieval over sensitive data, and a speculative future of dynamically generated harnesses.

## Key Takeaways

- **Reliability is the goal.** A harness exists to make agents do their job even when the model is a black box and its outputs vary.
- **A harness is more than the agent loop.** It also includes tools, context management, guardrails, and verification — and can wrap an outer loop around repeated attempts.
- **A harness is not an ML test harness.** The ML sense runs inputs through a model to score outputs; the agent harness is runtime infrastructure.
- **Agents can lie about success.** The demo's agent claimed an upvote it never made because nothing independently checked the browser state.
- **Verify against evidence you control.** Check tool history and browser state rather than trusting the model's summary; lint and tests play the same role for coding agents.
- **Guardrails bound cost and drift.** Caps on iterations and messages, plus context compression, keep a run within practical limits.
- **Do sensitive steps in the harness.** A login handler that fills credentials programmatically keeps secrets out of the prompt.
- **You can improve outcomes without prompting harder.** Changing the surrounding runtime, not the prompt, made the old model succeed.

## Detailed Notes

### Why an agent needs a harness · 01:17
- Kumar opens with an audience check: who feels confident explaining AI harnesses? The talk is meant to change those hands.
- He notes the term is overloaded — in ML it means roughly a test suite for models; in AI engineering it means something different.
- The core problem: builders pay for rented inference, get a limited context window, and use a model that is a black box. A provider could swap which model serves you and you might never know.
- Because so many variables are outside your control, the game becomes reliability: making agents do what they are meant to do regardless of the model.
- The name comes from physical harnesses — a climber's anchor or a dog's leash — things that tie you to something stable so you cannot drift too far.
- He distinguishes the ML harness from the agent harness, which is everything around the model that grounds it in reality. Claude Code, Cursor, and Codex are harnessed coding agents.

### The working parts of a grounded agent · 04:11
- Recurring components: a **tool registry** (read files, write files, run bash), a **model**, **context-management primitives**, **guardrails**, an **agent loop**, and a **verify step**.
- Most harnessed runtimes compact their own context — context management is the harness's job.
- Guardrails include things like a max-steps rule ("no more than five tool calls") that kills the run when exceeded.
- He stresses the harness is not just the agent loop; it is the surrounding structure, and it can itself be a loop around the agent loop.
- For a coding agent, verification means running lint and tests after the work to confirm nothing broke.
- The broader point: the harness grounds black-box models in a stable environment you control — and it works for any kind of agent, not just coding ones.

### A minimal browser agent exposes the failure mode · 06:13
- The demo: build a "baby's first harness" browser agent whose job is to visit Hacker News and upvote the first post.
- He deliberately uses GPT-3.5 Turbo (a 2023 model) to show a harness can make a weak model succeed, and commits to leaving both the task prompt and system prompt unchanged.
- The prompt is just "upvote a story." His reasoning: many engineers react to agent failures by prompting harder, which is not always the answer.
- Setup uses Playwright directly (not Playwright MCP) to launch Chromium, create a context, and open a page — ordinary browser automation.
- Tools follow the OpenAI SDK shape: name, description, parameters, and an execute function; the browser session is passed into them.
- The initial context is minimal: a basic system prompt plus the user's task.
- The agent loop requests a response, stops when the model signals "stop," and appends each event to a trace. On the first run it reaches Hacker News, clicks upvote, hits a login screen, and reports success anyway. It does not verify.

### Add limits, preserve evidence, and verify failure honestly · 09:24
- First change: default guardrails. `maxIterations` (stop after a set number of steps) and `maxMessages` (compress context past a threshold).
- The loop also logs metadata such as context size, making the runtime's interventions observable.
- The context compressor is deliberately naive: it keeps the system prompt, the user prompt, and the two most recent messages, discarding the middle. Kumar calls it basic and says better approaches exist.
- Next he renames the orchestration into a `runHarness` function — initially it just moves code around.
- Then he adds a `verify` step and a `maxAttempts` setting, so an outer harness loop can run an inner attempt up to three times.
- The verifier is deterministic and inspects the tool history, not the model's claims. It returns true only if a real, successful upvote click occurred, and fails early on a failed auto-login tool or an unrecovered login redirect.
- Result: the task still fails, but now it fails honestly. "Step one to solving a problem is admitting you have one."

### Handle authentication outside the model · 15:16
- The final change is a login handler that runs each agent-loop cycle just before trace events are pushed.
- It checks the browser's current URL: if not on a login page, it returns and does nothing (a cheap check).
- If on a login page, it fills in credentials and submits the form programmatically — deterministically and securely, with access to secrets the model never sees. Credentials could come from an environment variable.
- It also pushes a message telling the agent that the harness has logged in, so the agent knows it can continue.
- Key point: authentication is done by the harness, not the agent. The runtime handles the predictable, sensitive transition; the model keeps pursuing the task.
- With this in place, the agent opens Hacker News, authenticates, completes the upvote, and succeeds after six iterations. Kumar verifies the result by checking that an "unvote" option is now available. The prompts were never changed.

### Where harness engineering leads · 17:47
- Kumar's takeaway: harnesses can make cheaper or smaller models useful by replacing nondeterminism with stable application logic. (His demo shows one workflow, not proof for every task.)
- He notes 2025 was called the year of agents and suggests 2026 is the year of harnesses.
- He describes **OpenRAG**, IBM's open-source project deployed in enterprises to run RAG (retrieval-augmented generation) over sensitive, siloed data — Teams calls, PDFs, invoices — with enterprise-level security. He highlights its harness rather than its security architecture or any measured results.
- Speculative next step: **dynamically generated harnesses**. Before attempting something like buying a flight ticket, an agent might create its own task-specific constraints and verification — like plan mode on steroids, anticipating where it might hallucinate. He frames this as a possibility, not a demonstrated capability.