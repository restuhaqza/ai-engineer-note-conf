# Context Is the New Code

| Field | Details |
| --- | --- |
| **Speaker** | Patrick Debois |
| **Affiliation** | Tessl |
| **Type** | Keynote |
| **Date** | 2026-05-03 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=bSG9wUYaHWU) |
| **Tags** | `apis-mcp-and-protocols` `rag-context-and-search` `developer-workflows-and-testing` `coding-and-developer-tools` |

## Summary

Patrick Debois argues that the instructions, skills, documentation, and organizational knowledge that guide coding agents now deserve the same engineering discipline as source code. Just as he once asked what would happen if operations looked more like development (the origin of DevOps), he asks what a **context development lifecycle** looks like. The answer is a familiar infinity loop: generate context, evaluate it, distribute it, observe whether it works, then adapt and regenerate.

Most of the talk is spent on the evaluate step. If you change two lines in an instructions file, do you know what behavior changed? Debois wants evals — tests for your context — that check whether local conventions actually affect generated output, and that exercise the running system rather than only inspecting generated text. Because evals are non-deterministic, he suggests running them repeatedly and using error budgets instead of simple pass/fail gates.

He then covers distribution and observation. Reusable skills can be packaged and shared through registries like libraries, but that brings dependency conflicts and supply-chain risk. Feedback from agent logs, pull-request reviews, and production failures points to missing context that should be authored once and shared. His closing image: models are the engine, context is the fuel, and context is the part an engineering team actually controls.

## Key Takeaways

- **Instructions are reusable artifacts.** When the thing you maintain is a skill or instruction rather than a function, it needs the same care as code.
- **A context lifecycle mirrors DevOps.** Generate, evaluate, distribute, and observe — then feed observations back into better context.
- **Test whether conventions change output.** A behavioral eval checks that a rule in `AGENTS.md` actually appears in the generated artifact.
- **Exercise the running system.** Give an LLM judge tools and a sandbox and it becomes an agent that can run the app and call the endpoint.
- **Evals are non-deterministic.** Run them several times and count successes; use error budgets to express how much variation each test set tolerates.
- **Distribution brings dependencies and risk.** Context packages can conflict, and published skills need provenance — he proposes an "AI SBOM" (a record of a package's origins).
- **Observability closes the loop.** Agent logs, PR comments, and production failures reveal missing context.
- **Sandboxes do not check instructions.** A proposed "context filter" would inspect incoming instructions for prompt injection, like a web application firewall.
- **The hidden cost is eval design.** The effort saved by writing context shifts into defining and maintaining good evaluations.

## Detailed Notes

### Code becomes reusable context · 01:02
- Debois opens the AI Architects track and asks how many people have used an AI coding agent; nearly all hands go up.
- He calls the talk an unfinished thought: he increasingly asks a coding agent for changes instead of editing code himself.
- A concrete shift: he turned large helpers into a skill. Onboarding agents across Python, Node.js, and different package managers would have needed lots of branching code, but a skill with a simple procedure handled more variation:
  1. Identify the user's package manager.
  2. Identify the surrounding ecosystem.
  3. Walk through the onboarding steps with the user.
- The reusable asset is the procedure for adapting to the environment, not a fixed implementation for every case. Code is transforming back into context.

### A lifecycle for context · 02:37
- He draws a parallel to 2009, when he asked what would happen if operations looked more like development; collaboration and deployment practices followed.
- Applying the same move to context raises a maintenance question: if instructions influence what gets built, how should teams develop them consistently?
- His **context development lifecycle** is an infinity loop: generate, evaluate, distribute, observe, then adapt and regenerate.
- Writing the first instruction is only the beginning; keeping it useful is what closes the loop.

### Generate context: prompts, files, docs, MCP · 03:47
- Generation starts with ordinary prompting. He was surprised that asking when his talk was scheduled made an agent fetch the website and find the session — a simple request that worked because he had already supplied identity context.
- Repeated requests become reusable instructions. Files such as `AGENTS.md` give them a home; he contrasts that emerging convention with `CLAUDE.md`.
- Library documentation is another input. When a model is unsure whether an API is version two or three, retrieving the right version's docs — ideally agent-friendly — guides generation toward the library actually in use.
- **MCP (a standard way for agents to call tools)** pulls in material from GitLab, GitHub, and Slack; the ticket describing a task is itself context.
- Spec-driven development adds a transformation: write a specification, let the agent break it into a plan, then work through step-by-step prompts. Context can be authored, retrieved, or generated from other context.

### Evaluate context like code · 06:02
- Change two lines in an instructions file and you may not know the impact; switch coding agents and the same instructions may not work. These are regression questions, and evals (tests for AI behavior) should apply to everyday coding context, not only to models.
- **Linting layer:** a skill validator checks that required metadata exists and that a description stays within the permitted length — structural validity before behavioral usefulness.
- **Clarity layer:** can the agent understand what the author means? A two-word instruction may be too terse to act on. Asking a model to assess the context exposes implicit assumptions and missing detail. He likes voice coding here because speaking makes him more expansive than typing.
- **A behavioral eval** needs a rule, a task, and an observable expectation. Example: a convention states every API endpoint must use the `/awesome` prefix; asking for an endpoint that saves a user should produce `/awesome/user`.
- An **LLM judge** can inspect the generated code and decide whether it starts with `/awesome`. A regex would suffice for this rule, but the judge shows how a broader criterion could be scored. Without the instruction, no model would pick that prefix — so the eval tests whether local knowledge changed the artifact.
- The same suite reveals differences between coding systems such as Gemini and Copilot. Collected checks become something like unit tests for context, and can cover infrastructure-as-code and configuration files too.

### Run the system, then improve the context · 10:44
- Finding `/awesome/user` in a file does not prove the endpoint works. Give the judge tools and a sandbox and it becomes an agent that can run the app and issue a request (for example, a `curl` POST with a JSON user record).
- This turns the check into an end-to-end test: it exercises the running system, not just the generated text.
- Scenarios become more useful with versioning: select a repository commit, supply a piece of context, and ask whether that context made a difference. Commit instructions alongside code so the evaluated input is identifiable.
- When tests report failures or missing requirements, a code action can use that feedback to propose a context revision — giving improvement a target rather than another round of unguided wording changes.

### CI/CD for context: repeated trials and error budgets · 12:26
- The obvious next step is CI/CD (automated build-and-test pipelines), but agent evals do not always produce the same result on successive runs, which makes a single pass or failure hard to interpret.
- His illustrative procedure: run an eval five times and count the successes. One case may pass all five; another may vary. That describes the trials without guaranteeing future reliability.
- **Error budgets** express how much variation matters: give critical test sets very little failure tolerance and allow more in less consequential cases.
- Because context edits change which tests pass, evaluate the pattern across the suite rather than treating each check as an ordinary deterministic test.

### Distribute context like a library · 13:52
- Checking context into a repository already distributes it: a colleague checks out the project and receives the instructions.
- Reuse across projects or teams calls for something closer to a library. Package shared conventions — say, front-end guidelines — so each project installs what it needs; a registry answers the discovery question, as in the Tessl Skills Registry and skill marketplaces.
- He uses "99.9" as a rhetorical dismissal of skill quality, not a measured statistic. His practical concern is that many published skills do not meet a useful standard, even if reading them offers ideas — so teams may want their own maintained registries.
- A skill bundles more than a prompt: instructions, scripts, and documents; plugins can also include MCP integrations. Support across coding agents makes skills an emerging package format.
- **Dependencies:** install multiple context packages and their guidance can conflict (a general front-end package disagreeing with a React-specific one). Matching context-package versions to library versions keeps instructions aligned with the code.
- **Supply-chain risk:** published packages raise a trust question. He points to OpenClaw as a moment when people realized they were running strangers' material on their laptops. A Snyk context scan flags findings such as "Improper Credential Handling" and "Third-Party Content Exposure."
- **Provenance** is next: who built the skill, how, and which model contributed? He proposes an **AI SBOM** — a record of a context package's origins and composition, extending the questions teams already ask about dependencies.

### Observe behavior: logs, reviews, production, context filters · 17:42
- Maintaining context for others creates an obligation to check whether it still works. Agent logs are one feedback channel: a developer can notice a mistake and add the missing instruction. He compares this to TDD, then qualifies it — it is a reaction to a discovered problem, not test-first.
- At organizational scale, look for repeated gaps: if many agents report the same missing information, author it once and distribute it so the fix reaches people who have not hit the failure yet. Emerging log standardization helps this channel reveal both missing context and whether supplied context was used.
- A correction on a generated pull request is also feedback on the context that produced it. Arguing repeatedly about the same omission fixes one output; improving the originating instructions prevents the next iteration from repeating it.
- Production adds evidence review may miss: the generated code must handle real inputs. He describes a wrapper that instruments code and ships it to production; when something fails it identifies the changed code and the input/output behavior that went wrong, and that failure can become a regression test.
- Runtime observation also needs to catch unexpected agent behavior. In a sandboxing experiment he asked an agent to break out of the system; it used environment variables, and after he removed a secret it turned to memory files.
- Execution isolation and instruction intake are different boundaries. Coding agents load `AGENTS.md` and `SKILL.md` without an intervening restriction, so sandboxing execution does not inspect the instructions being consumed.
- He proposes a **context filter** — analogous to a web application firewall — that examines incoming material for suspicious patterns and prompt injection.

| Boundary | Purpose |
| --- | --- |
| Execution sandbox | Restrict what agent actions can access or do |
| Context filter (proposed) | Inspect instructions entering the agent |

- Harness engineering (logs, traces, feedback) ties these observations together, useful both for training and for running an organization's own agent workflows.

### Closing the loop and the cost of evals · 22:33
- Two loops emerge. The **inner loop** is library authoring: create context and test it. The **outer loop** begins when others use it: distribute it, observe whether it stays useful, and feed that evidence into the next revision.
- He compares the ongoing improvement to a Sonar/CI-CD model for context. Personal Markdown craftsmanship can become a team habit — when something is missing, add it to the maintained context — and across a team of teams one correction becomes reusable knowledge.
- His closing analogy is an engine and its fuel: models and coding agents are the engine, context is the fuel. He cannot change the model he is given, but he can improve its context — an engineering surface under the team's control.
- In the audience Q&A, an attendee describes using consistency as a form of context or eval: turn a loose plan into a crisp definition in parallel and compare results. Wide divergence suggests the original plan was ambiguous; agreement is a signal of consistency, not proof it is correct.
- Debois does not give a specific answer for that architectural case. Instead he names the hidden cost of moving from code to context: apparent savings in implementation become time spent designing the right evals. There is no longer one prompt to get right — all the eval prompts need care too.

## Resources

- "The Context Development Lifecycle" — Debois's write-up of the generate/evaluate/distribute/observe loop (related article)
- "CI/CD for Context: Same Pipeline, Different Rules" — repeated evaluations, error budgets, and context pipelines (related article)
- [AGENTS.md](https://agents.md/) — a convention for giving coding agents repository-specific instructions and conventions
- [Agent Skills specification](https://agentskills.io/specification) — defines `SKILL.md` structure and metadata constraints for on-demand skills
- [Tessl Skills Registry](https://tessl.io/registry) — versioned agent skills with quality, evaluation, and security signals
- [Snyk Agent Scan — Skill Inspector](https://labs.snyk.io/resources/agent-scan-skill-inspector/) — scanning third-party skills for risks such as improper credential handling
