# Don't Build Slop: Four Levels of AI Agent Maturity

| Field | Details |
| --- | --- |
| **Speaker** | Ara Khan |
| **Affiliation** | Cline |
| **Type** | Talk |
| **Date** | 2026-05-19 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=yUmS-F9IX90) |
| **Tags** | `agent-engineering` `coding-and-developer-tools` `infrastructure-and-deployment` `apis-mcp-and-protocols` `reasoning-and-models` `rag-context-and-search` |

## Summary

Ara Khan opens with a familiar anxiety: everyone else seems to have a room full of agents working for them, so should you run fifteen agents while supervising, or still read every line of code? His answer is to slow down and split the vague task "build an agent" into four maturity levels, each addressing a different constraint.

The four levels progress from testing an idea with a framework, to building your own agent as an explicit state machine, to coordinating several agents through a Kanban board, to moving execution into the cloud so it is easy to share and scale.

For the custom-agent stage, Khan gives five opinionated rules: treat the agent as a state machine, assume every addition can make it worse, make it easy for other agents to build and test, spend human attention on architecture, and respect each provider's API semantics. He argues that newer frontier models need less instruction, and that provider-specific "reasoning traces" create real lock-in.

The talk closes with Kanban as the interaction surface and cloud agents as the execution model, plus an audience question on where planning and human review fit inside a task card.

## Key Takeaways

- **Split "build an agent" into four questions.** Does the idea work? Can we control it? Can we coordinate the work? Can others run it easily?
- **Level 1 is a framework prototype.** LangChain, LangGraph, CrewAI and others quickly show whether a workflow is useful; Khan does not use them himself.
- **Level 2 is your own state machine.** At any moment you should know what state the agent is in and what can happen next.
- **Every addition risks making an agent worse.** Large prompts and extra edge-case logic can overload a frontier model; newer prompts tend to be shorter.
- **Make the agent easy for other agents to build and test.** A CLI, good `AGENTS.md`, skills and CI/CD create a build-and-test loop a coding agent can run.
- **Spend human attention on architecture.** Read the code and own the design even when generation is cheap.
- **Provider APIs create lock-in.** Reasoning traces must be passed back in the format each provider expects, or quality silently drops.
- **Kanban is the interface, cloud is the compute.** Boards let you manage agents like a team; cloud agents run long tasks without occupying your machine.

## Detailed Notes

### Fifteen agents or read every line? · 00:29
- Khan describes a "mass psychosis" feeling: other people's agents look hugely productive, so it is hard to judge how much automation your own work needs.
- Three frontier-lab interfaces (Factory, Codex, Cursor) look nearly interchangeable, so copying appearances gives little guidance.
- His response is to slow down and identify a problem actually worth solving, balancing the need to build fast with the need to eventually build for production.
- He splits the task into four levels: prototype with a framework, build a custom agent, coordinate through Kanban, and ship to the cloud.

### Level 1: prototype with a framework · 03:00
- When the goal is to find out whether an idea works, the best model or a bespoke runtime matters less than learning whether the workflow is useful.
- **Frameworks** such as LangChain, LangGraph, CrewAI, AutoGen and LlamaIndex can get a rudimentary prototype running quickly.
- Khan estimates a working prototype in about half an hour for a simple task like aggregating emails.
- He is explicit that he builds Cline's agent experience himself and does not use these frameworks, so he is not offering a detailed comparison.
- His opinionated objection: the customisation, modularity and ability to adapt to future models he wants are hard to get through a framework.

### Rule 1: every agent is a state machine · 04:17
- Treat the agent as a **state machine**: a loop with conditions, transitions and terminal (end) states.
- At any point you should be able to say where execution is and what can happen next.
- His example: Claude Code is asked to read a few files and explain them. It decides on a file-reading action, gets the contents, decides it has enough, and calls a completion tool that ends the task.
- In a small TypeScript sketch, a `decide` function represents the model-facing decision step, while the runtime owns tool execution and termination.
- More complex tasks can loop for eight to ten minutes or hours; longer runs make knowing the current state more valuable, not the loop itself different.

### Rule 2: additions can make the agent worse · 05:55
- More instructions do not automatically mean better behaviour. Large prompts, piled-up edge cases and elaborate `if`/`else` logic can get in a frontier model's way.
- Khan calls this a bitter lesson: instructions added to patch earlier limitations may become unnecessary constraints as models improve.
- He points to the Codex repository's prompts, saying the GPT-5.3 prompt is about one third the size of the GPT-5 one — a comparison he presents loosely.
- The engineering lesson: revisit accumulated instructions instead of assuming each one still helps.
- Pruning can go past the prompt — he says his team rewrote Cline in full after finding outdated material, and tentatively recalls several full rewrites of Claude Code.

### Rules 3 and 4: build for agents, protect architecture · 07:19
- **Rule 3:** make the agent easy to operate from a CLI so another coding agent can change it, run it, inspect the result, and retry. Khan calls this a "pseudo RL pipeline" — the feedback is from building and testing software, not training model weights.
- This shifts what humans prepare: instead of only instructing an AI, you shape the repo so an AI can work through it — good `AGENTS.md`, skills, and accessible CI/CD commands.
- End-to-end tests matter because the coding agent needs a way to check behaviour after a change.
- **Rule 4:** don't build slop. High token throughput makes it easy to generate a large implementation before deciding whether its design makes sense.
- Spend human attention on the agent's intended behaviour, architecture and design outline, and read the code even if you did not hand-write it.

### Rule 5: provider APIs create lock-in · 09:47
- Switching models is not always just changing an endpoint and a model name; provider-specific API behaviour is a source of lock-in.
- He uses the broad term **reasoning traces** for information that participates in caching and the model's reasoning loop.
- Across turns, an application must preserve and return that information in the format the provider expects; flattening a conversation to plain text can lose behaviour that matters.
- Incorrect handling can leave replies looking fine while performance quietly degrades — and failure behaviour depends on the provider.
- Khan argues that routing through OpenRouter alone is not enough assurance; test the conversation and tool-use path for each integration.
- Examples he names: Opus 4.6, Gemini 3.1 Pro and GPT-5.3 Codex.

### Level 3: coordinate parallel work with Kanban · 11:09
- Once the agent works, the next question is how to interact with several at once.
- The reason starts with waiting: Codex or Opus tasks can run eight to ten minutes, which tempts users to keep two or three agents busy at the same time.
- Because those agents may be editing the same source, **parallel execution needs isolation of mutable state** — a board shows the work, but the execution setup must stop tasks interfering.
- **Kanban** boards (familiar from tools like Linear) give a headline view of each task and make dependencies easier to reason about.
- The user starts to look like an engineering manager coordinating contributors, inspecting the work that needs attention rather than watching every output.
- Khan notes he argued for Kanban in a March 26 tweet, and says Claude Code released a similar interface shortly before the talk.

### Level 4 and Q&A: cloud execution and where planning lives · 13:11
- An agent can be useful and easy to inspect yet still hard to distribute; supporting millions of tasks or an 8,000-person company is a deployment question, not a demonstrated result.
- Requiring everyone to install software and maintain a local workflow raises the barrier, so Khan proposes doing the environment work centrally and running agents in the cloud.
- A separate machine per agent allows parallel execution with no local dependencies, and enables longer runs — a capability he thinks is underused.
- He reports sending cloud tasks from his phone that run fifteen to twenty minutes; in his VS Code extension example, the agent builds, signs in, opens settings, picks a theme, and tests the result, iterating on failures, with a full implementation-plus-QA cycle taking fifty to sixty minutes.
- A cloud setup can also be a shared, modifiable environment for collaborators; his forecast is that most agent interaction shifts to Kanban while most execution moves to cloud compute.
- **Q&A:** planning still happens inside the task card. Opening a card shows the whole task trace, and that inner interface is the actual Codex CLI. Khan converses there until the task is clear enough to run alone.
- Cards change state too: a task starts `in progress` and moves to `review` when it needs human input, making the need for attention visible on the board.

## Resources

- Introducing Cline Kanban — launch article on live agent task cards and dependencies.
- Gemini thought signatures — provider documentation on preserving reasoning state.
- Claude extended thinking — current guidance on manual thinking budgets and adaptive thinking.
