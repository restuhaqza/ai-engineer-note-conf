# The Multi-Agent Architecture That Actually Ships

| Field | Details |
| --- | --- |
| **Speaker** | Luke Alvoeiro |
| **Affiliation** | Factory |
| **Type** | Talk |
| **Date** | 2026-05-06 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=ow1we5PzK-o) |
| **Tags** | `multi-agent` `architecture` `orchestration` `validation` `factory` `agent-systems` |

## Summary

Luke Alvoeiro, from Factory and creator of the open-source coding agent Goose, argues that the current bottleneck in software engineering is not intelligence but human attention. While today's models are smart enough to tackle numerous tasks, the human bandwidth to supervise their implementation is insufficient. This talk addresses the chaotic state of multi-agent frameworks by proposing a taxonomy of five frontier strategies: delegation, creator-verifier, direct communication, negotiation, and broadcast.

Factory's "Missions" architecture is presented as a production-ready solution that combines four of these strategies (delegation, creator-verifier, broadcast, and negotiation) into a cohesive workflow. Missions operate with a three-role architecture -- Orchestrator, Workers, and Validators -- and emphasize serial execution with targeted internal parallelization, structured handoffs, and a revolutionary approach to validation through "validation contracts" written *before* any code. This system enables agents to autonomously execute complex software development tasks for days, allowing humans to focus on higher-level strategic decisions.

The architecture is designed to be model-agnostic, promoting "droid whispering" -- the strategic selection of different LLMs for specific roles (e.g., planning, implementation, validation) to leverage their unique strengths. This approach ensures the system improves with each new model generation, rather than becoming obsolete. Missions have been successfully used in enterprise settings for rapid prototyping, internal tool development, large-scale refactors, and codebase modernization, ultimately leading to cleaner codebases and a significant increase in engineering team throughput.

## Detailed Notes

### Introduction & The Bottleneck of Human Attention
- **Speaker Background:** Luke Alvoeiro has a background in dev tools, started Goose (a leading open-source coding agent, now part of the AI Agentic AI Foundation) at Block, and currently leads the core agent harness at Factory.
- **Factory's Mission:** To bring autonomy to the entire software development life cycle.
- **Core Claim:** The bottleneck in software engineering is human attention, not intelligence.
    - Even the best engineers can only complete a couple of tasks at a time, limited by the need for their attention and review.
    - Today's models are intelligent enough to figure out many tasks, but there isn't enough human bandwidth to supervise their implementation.
- **Vision:** A human decides *what* to build, and a system figures out *how* to do so, allowing agents to work for hours or days autonomously.

### Taxonomy of Five Frontier Multi-Agent Frameworks
The field of multi-agent systems is fragmented, so a simple taxonomy is proposed:
1.  **Delegation:** One agent spawns another (e.g., a parent agent asks a sub-agent to figure out a database schema). This is the simplest and most common form, seen in sub-agents and coding tools.
2.  **Creator-Verifier:** One agent builds something, and another agent checks that work. Key is separation of concerns; a fresh agent with fresh context is more likely to find issues, similar to human code review.
3.  **Direct Communication:** Agents communicate without a central coordinator (like direct messaging). Hard to get right as state fragments across conversations without a single source of truth.
4.  **Negotiation:** Agents communicate over a shared resource (e.g., using the same API, modifying the same code portion). Best use cases involve net-positive sum trading where agents have win-win situations.
5.  **Broadcast:** One agent sends information to many (e.g., status updates, new context, shared constraints). Less flashy but critical for maintaining coherence over long-running tasks.

### Missions: The Three-Role Architecture
Missions is Factory's answer to assembling these building blocks into a system that can run for many days, combining four of the five frameworks: Delegation, Creator-Verifier, Broadcast, and Negotiation.
- **Workflow:** Describe a goal, scope it through conversation, approve a plan, and the system handles execution for hours or days.
- **Nature:** Not a single agent session, but an ecosystem of agents communicating through structured handoffs and shared state.
- **Three Roles:**
    - **Orchestrator:**
        - Handles planning and strategic questions.
        - Checks for unclear requirements.
        - Produces a plan including features, milestones, and a **validation contract**.
    - **Workers:**
        - Handle implementation.
        - Receive clean context without accumulated baggage.
        - Read specs, implement features, and commit changes via Git.
    - **Validators:**
        - Handle verification.
        - Perform traditional checks (lint, type check, tests, code review).
        - Crucially, they **validate behavior** end-to-end, ensuring the code works as intended, preventing system drift over long runs.

### The Importance of Validation Contracts
- **Problem with Traditional Validation:** Tests written *after* implementation are often shaped by the code itself, confirming decisions rather than catching bugs. This leads to system drift.
- **Missions Solution:** The **validation contract** is written during the planning phase, *before* any code is implemented.
    - It defines correctness independently of the implementation.
    - For complex projects, it can contain hundreds of assertions. Each feature is assigned one or more assertions it must satisfy.
    - The sum of all features must cover every assertion.
- **Two Types of Validators (run after each milestone):**
    1.  **Scrutiny Validator:**
        - Runs the test suite, type checking, and lints.
        - Spawns dedicated code review agents for each completed feature within the milestone.
    2.  **User Testing Validator:**
        - Acts like a QA engineer.
        - Spawns the application and interacts with it via computer vision (fills forms, checks rendering, clicks buttons) to ensure functional flows work holistically.
        - This step takes significantly longer, with most of a mission's wall clock time spent here, not generating tokens.
- **Adversarial by Design:** Neither validator has seen the code before, ensuring they are not invested in the implementation and provide unbiased verification.

### Maintaining Long-Term Context Through Structured Handoffs
- To prevent context loss in multi-day runs, workers don't just declare "I'm done."
- **Structured Handoffs:** When a worker finishes a feature, it details:
    - What was completed and what was left undone.
    - Commands run and their exit codes.
    - Issues discovered.
    - Whether it abided by orchestrator-defined procedures.
- **Self-Healing:** Errors are caught at milestone boundaries. Corrective work is scoped, and the mission pulls itself back on track by addressing issues documented in handoffs.
- **Scale:** The longest mission ran for 16 days (much longer than a full sprint), with a goal of 30 days, enabled by this structure.

### The Case for Serial Execution Over Parallel Execution
- **Initial Thought:** Parallelism (e.g., 10 agents running simultaneously) for 10x throughput.
- **Problem with Parallelism in SDLC:** Agents conflict, step on each other's changes, duplicate work, and make inconsistent architectural decisions. Coordination overhead negates speed gains and burns tokens.
- **Missions Approach:** Features are run **serially** (only one worker or validator runs at any given time).
- **Targeted Internal Parallelization:** Within a feature, read-only operations (e.g., searching code, researching APIs) are parallelized. Within validators, read-only operations like code review are also parallelized.
- **Benefit:** While seemingly slower on paper, the error rate drops dramatically. For multi-day tasks, this correctness compounds, leading to faster overall completion.

### Mission Control: Monitoring Agent Progress
- A standard chat interface is insufficient for multi-day tasks.
- **Mission Control View:** A dedicated interface (itself built using a mission) allows users to:
    - See the active worker's current task.
    - Read handoff summaries detailing discoveries and course alterations.
    - Monitor project completion and budget burn.
- **Asynchronous Oversight:** This view allows users to oversee missions asynchronously, acting as a project manager or stepping away from the system entirely.

### Strategic Model Selection Per Role ('Droid Whispering')
- **Model Specialization:** Different roles benefit from different LLM strengths:
    - **Planning:** Slow, careful reasoning.
    - **Implementation:** Fast code fluency and creativity.
    - **Validation:** Precise instruction following.
- **Challenge:** No single model or provider is best at all three.
- **"Droid Whispering":** A new skill required to mentally model how different LLMs interact, where they fail, how failures compound, and then deliberately choose the right model for each role.
    - Example: Validation might use a different model provider entirely to avoid bias from the same training data.
- **Model-Agnostic Architecture:** This is a structural advantage. Being locked into one provider constrains the system by that family's weakest capability. As models specialize, the ability to select the best model for each seat becomes a compounding advantage.
- **Compensating for Weaker Models:** The structure of Missions (validation contracts, milestone checkpoints) can enable successful runs even with open-weight or non-frontier models.

### Production Data Analysis: Building a Slack Clone
- **Example Project:** Building a clone of Slack.
- **Key Metrics:**
    - 60% of time and tokens spent on implementation.
    - Validation rarely succeeds on the first attempt, almost always requiring follow-up features, demonstrating the value of the QA loop.
    - Final codebase: 50% of lines of code are tests, achieving 90% code coverage.
    - Heavy use of prompt caching to offset the cost of long-running tasks.

### Designing Systems That Improve with Each Model Generation
- **The "Bitter Lesson" Fear:** The concern that the next model release will render an architecture obsolete.
- **Missions' Solution:** Designed to get better with every model improvement.
    - **Prompt-Driven Logic:** Almost all orchestration logic is defined in prompts and skills, not hard-coded state machines.
    - Failure decomposition and feature decomposition are defined in ~700 lines of text; even four sentences can dramatically alter execution strategy.
    - Worker behavior is driven by orchestrator-defined skills per mission, allowing for customized behavior.
    - **Thin Deterministic Logic:** Only essential bookkeeping (running validation, blocking progress on handoff issues) is hard-coded.
- Missions provide discipline, while models provide intelligence using familiar primitives (agents.md, skills).

### The Shifting Economics of Software Engineering
- **Addressing the Bottleneck:** Missions shift the bottleneck from human attention.
- **Increased Throughput:** A team of five engineers might increase from 10 work streams to 30 with Missions.
- **Human Focus:** Engineers can focus on higher-level, interesting problems like architecture and product decisions, rather than execution.
- **Cleaner Codebases:** The system produces cleaner codebases with end-to-end tests, unit tests, and structured skills, making agents and humans more productive moving forward.

### Composition of Strategies in Missions
Missions effectively compose the frontier multi-agent strategies:
- **Delegation:** Evident in how the orchestrator spawns workers and research sub-agents.
- **Creator-Verifier:** Fundamental, as validation and implementation are always handled by separate agents with separate contexts.
- **Broadcast:** Achieved through the shared mission state that every agent references.
- **Negotiation:** Occurs at milestone boundaries, where the orchestrator evaluates handoff summaries and decides on follow-up features or rescoping.

### Future Directions
- Further parallelizing mission workloads for faster execution.
- Orchestrating missions themselves into even more complex workflows.
- The data from production missions confirms this approach works on real projects at scale today.

## Key Technical Concepts

- **Multi-Agent Taxonomy:** A classification of five fundamental multi-agent interaction patterns:
    - **Delegation:** Hierarchical task assignment.
    - **Creator-Verifier:** Separation of concerns for building and validating work.
    - **Direct Communication:** Peer-to-peer agent interaction without a central hub.
    - **Negotiation:** Agents interacting over shared resources, ideally for mutual benefit.
    - **Broadcast:** One-to-many information dissemination for coherence.
- **Missions Architecture:** A production-grade multi-agent system combining four of the taxonomy's strategies, structured around three core roles:
    - **Orchestrator:** Plans, scopes, and defines validation contracts.
    - **Workers:** Implement features based on specifications.
    - **Validators:** Verify implementation correctness and behavior.
- **Validation Contracts:** A critical innovation where correctness criteria (hundreds of assertions) are defined *during planning* and *before any code is written*. This ensures validation is independent of implementation and catches actual bugs, preventing system drift.
- **Structured Handoffs:** Detailed reports from workers upon task completion, outlining work done, issues, commands run, and adherence to procedures. These enable context retention and self-healing in long-running missions.
- **Serial Execution with Targeted Internal Parallelization:** The core execution strategy where features are processed one at a time to avoid conflicts and errors, but read-only operations within features or validators are parallelized for efficiency. This prioritizes correctness and compounds value over long tasks.
- **Droid Whispering:** The skill of strategically selecting and configuring different LLMs for specific roles within the multi-agent system (e.g., a reasoning-focused model for planning, a creative model for coding, a precise model for validation) to leverage their specialized strengths.
- **Model-Agnostic Architecture:** A system design that is not locked into a single LLM provider or model family, allowing for flexible model selection and ensuring the system can adapt and improve with the evolution of diverse AI models.

## Practical Applications

- **Rapid Prototyping:** Quickly build new ideas and features overnight, accelerating innovation cycles.
- **Internal Tool Development:** Significantly increase the speed at which internal tools can be built and deployed.
- **Large-Scale Refactors and Migrations:** Automate and manage complex code refactoring and migration projects that would otherwise be time-consuming and error-prone for human teams.
- **ML Research:** Support machine learning research by automating code generation, experimentation, and data analysis tasks.
- **Codebase Modernization:** Use agents to update and improve existing codebases, making them more productive for both human and agent developers.
- **Shift Human Focus:** Enable engineering teams to move away from execution details and concentrate on higher-level architectural design, product strategy, and complex problem-solving.
- **Build for Model Evolution:** Design systems where orchestration logic is defined in prompts and skills, ensuring the architecture improves with each new model generation rather than becoming obsolete.

## Notable Quotes

> "The bottleneck in software engineering nowadays is not intelligence. It's now limited by human attention."

> "Tests written after implementation don't catch bugs. They confirm decisions."

> "Using systems like missions requires the development of a new skill, which internally we've been calling droid whispering, but it's this idea that you need to be able to mentally model how different LLMs interact, where they fail, how those failures compound over a multi-day run, and then you need to make a deliberate choice as to which model sits in which seat."

## Timestamps

| Time | Topic |
|------|-------|
| 0:00 | Introduction to multi-agent systems and the bottleneck of human attention |
| 1:50 | Taxonomy of five frontier multi-agent frameworks |
| 4:04 | Introducing 'Missions': The three-role architecture |
| 6:34 | The importance of validation contracts |
| 8:09 | Maintaining long-term context through structured handoffs |
| 9:17 | The case for serial execution over parallel execution |
| 10:30 | Mission control: Monitoring agent progress |
| 11:22 | Strategic model selection per role ('Droid whispering') |
| 13:06 | Production data analysis: Building a Slack clone |
| 14:34 | Designing systems that improve with each model generation |
| 15:51 | Conclusion: The shifting economics of software engineering |

## Takeaways

- The orchestrator-workers-validators pattern is battle-tested in production
- Serial execution with validation beats parallel execution without it
- Model selection per role compounds in value as models improve
- Build for model evolution, not a specific model version

## Speaker Links

- GitHub: https://github.com/lukealvoeiro
- LinkedIn: https://www.linkedin.com/in/lukealvoeiro
