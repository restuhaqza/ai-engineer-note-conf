# Vibe Engineering Effect Apps

| Field | Details |
| --- | --- |
| **Speaker** | Michael Arnaldi |
| **Affiliation** | Effectful |
| **Type** | Workshop |
| **Date** | 2026-05-07 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=Wmp2Tku2PrI) |
| **Tags** | `effect` `typescript` `llm-coding-agents` `testing` `http-api` `agent-instructions` |

## Summary

This workshop explores a novel approach to leveraging Large Language Models (LLMs) for software development: instead of relying on documentation or pre-trained knowledge, give coding agents direct access to a library's source code. Michael Arnaldi, drawing from his experience of not writing code by hand for months, even for complex library-level TypeScript and Rust, argues that LLMs fundamentally differ from human brains. They don't learn continuously, operate within fixed context windows, and are primarily trained to consume and produce code, not to interpret human-written documentation or interact with Language Server Protocols (LSPs) they haven't encountered during training.

The core strategy, dubbed "just clone the repo," involves integrating the target library's source code (e.g., the Effect-TS repository) directly into the project as a `git subtree`. This method "tricks" the coding agent into perceiving the library as part of its own codebase, enabling it to extract patterns, conventions, and best practices directly from the source. This circumvents limitations where models might ignore code in `node_modules` or `.gitignore` directories, providing a more effective way for agents to understand and utilize complex libraries.

Through a live demonstration, Michael guides an LLM (GPT 5.4) to build a robust Effect application from an empty Bun project. The process covers configuring strict TypeScript diagnostics, establishing explicit agent instructions via an `agents.mmd` file, implementing an HTTP API with OpenAPI documentation, integrating Effect SQL for persistence, and refining testing strategies using `it.layer`. A key emphasis is placed on implementing "backpressure" through custom ESLint rules to prevent common agent-generated anti-patterns, effectively "babysitting a junior developer with a knife." The talk also touches upon the future potential of Effect's clustering and workflow capabilities for handling long-running, AI-driven processes.

## Detailed Notes

### Workshop Context & Philosophy

*   The workshop was "vibe engineered" to be authentic, starting "from scratch" with no pre-prepared code.
*   Michael Arnaldi has not been coding by hand since late summer, even for complex library-level development in TypeScript and Rust.
*   Initially, he believed AI would be more useful in application development than library development, but found this assumption to be "dead wrong."

### Understanding LLM Limitations

*   **Continuous Learning vs. Pre-training**: Unlike human brains that learn continuously, LLMs undergo distinct phases: pre-training (on vast datasets like the internet), specialization (e.g., coding agents reinforced on codebases), and post-training/fine-tuning. After these phases, "there's no more knowledge that comes into the model every day."
*   **Memory & Context Window**: LLMs do not "remember" past interactions in a continuous sense. Conversations are effectively "appending messages onto a fixed size array which is called the context window."
*   **Context Window Size**: While models with large context windows (e.g., 1 million tokens) exist, they are "not necessarily a good idea" as pushing too much information can "confuse the model," especially when handling multiple tasks.
*   **Compressed Knowledge**: Models operate with "compressed knowledge" due to parameter limitations, meaning they often have "knowledge of six months ago at best" and cannot store all human knowledge.

### The "Clone the Repo" Strategy

*   **Core Principle**: The session's main takeaway is to "just clone the repo and be done with it."
*   **Rationale**: Coding agents are primarily trained to "consume and produce code," understand your codebase, and replicate existing patterns. They are *not* trained to read human documentation or use Language Server Protocols (LSPs) they haven't encountered during their training.
*   **Implementation**: To make an LLM use a library effectively, add its repository (e.g., Effect) directly into your project directory (e.g., `repos/effect`), "masquerading the effect code base as my own codebase." This tricks the model into exploring and learning from the library's source.
*   **Why not `node_modules` or `.gitignore`?**: Models are optimized not to look at code in `node_modules` or files within `.gitignore` directories, making these locations ineffective for pattern extraction.

### Project Setup & Configuration

*   **Initial Stack**: The workshop started with an empty project, aiming to set up Bun, Vest (testing), and an HTTP server using Effect.
*   **TypeScript Diagnostics**: A critical step for AI-assisted development is to set *all* TypeScript diagnostics to `error`. This ensures "the LLM cannot pass cannot accept code that has any remote resemblance or an error."
*   **`effect-ts/language-service`**: While `effect.solutions` offers a quick start with a CLI and language service, it requires the model to learn the CLI. Direct repo cloning is preferred for deeper pattern understanding.
*   **Adding Effect Repo**: The Effect repository was added as a `git subtree` into `repos/effect` with squashed history.

### Agent Interaction & Backpressure

*   **`agents.mmd` File**: A markdown file (`agents.mmd`) is used to instruct the agent. It lists available project commands (e.g., `bun run type-check`) and explicitly states that the agent has access to `repos/effect` for pattern extraction. This file is expected to evolve with the project.
*   **ESLint for Backpressure**: ESLint is described as an "essential piece of the backpressure loop." Custom ESLint rules are used to prohibit undesirable coding patterns (e.g., `as any`, `as unknown`, explicit type assertions, `as never as X`), effectively "babysitting a junior developer with a knife."
*   **Branded Types**: Recommended to force identifiers to be branded types to improve type safety and prevent the model from using plain strings where specific IDs are expected.
*   **Validation at the Edge**: Custom rules can prohibit constructors inside handlers, forcing validation logic to be implemented directly within schemas at the API's edge.
*   **Prompting Nuances**:
    *   **GPT Models**: Using uppercase in prompts (e.g., "NEVER EVER") can "scare" GPT models, leading to deoptimization and passive agreement.
    *   **Claude Models**: Uppercase prompts with Claude can make it "pay attention to that specific sentence."
*   **Spec-Driven Development**: Instead of relying on an LLM's "plan mode," the speaker prefers discussing a specification with the model, persisting it as a markdown file, and then instructing the model to implement that spec.
*   **Ralph Loops**: Simple bash scripts are used to manage LLM interactions, instructing the model to "pick up a small task, implement the small task, and then exit." This "less is more" approach often works better than complex context management.
*   **Tool Reduction**: Experimentation suggests that reducing the number of tools available to a model (e.g., a single `execute` tool for TypeScript code) can lead to improved performance.

### Implementing an HTTP API with Effect

*   **Goal**: Create a Todo API (create, update, flag as done, list) with OpenAPI documentation and a type-safe client.
*   **Pattern Extraction**: The model explores `repos/effect` to find patterns for defining a `shared HTTP API`, deriving OpenAPI documentation, and mounting it.
*   **Schema & Types**: The initial API generation used plain strings for identifiers, which was noted as an area for future improvement (e.g., using branded types). The model sometimes generated `schema.struct` instead of classes, which the speaker personally prefers.
*   **OpenAPI Docs**: The API successfully generated and exposed OpenAPI documentation.

### Persistence with Effect SQL & SQLite

*   **Strategy**: `effect-sql` was chosen for persistence, using SQLite as the database.
*   **Pattern Extraction**: The model explored `repos/effect` to find patterns for using Effect SQL. This "self-selection" of patterns is crucial for brownfield projects, allowing developers to pick and choose only the necessary components.
*   **Generated Code**: The model generated an SQL client, inlined migrations, and the `SqlLive` layer for database interaction.
*   **Identified Issues**: Duplication in the generated code and the use of plain strings for IDs were noted as areas for refinement.

### Testing Effect Applications

*   **`vest` Configuration**: The `vest` test runner was configured to ignore the `repos/` directory to prevent it from running tests from the cloned Effect library.
*   **Bad Pattern**: The model initially generated custom `withHTTP` wrappers for tests, which was identified as an unnecessary and verbose pattern.
*   **`it.layer`**: The correct pattern for providing Effect Layers (dependencies) to tests is `it.layer`. The model was instructed to refactor to this pattern, leading to cleaner tests.
*   **Database Testing**: For production databases like PostgreSQL, the recommendation is to use transactions with rollback per test to ensure atomicity and prevent long test runtimes.

### LLM Model Comparisons

*   **GPT vs. Opus**: GPT 5.4 was found to be more concise than Opus for certain tasks (e.g., `agents.mmd` generation). GPT generally produces good output but often asks for constant input, while Opus can be faster but sometimes takes shortcuts (e.g., using `as any`).
*   **Policy Restrictions**: The speaker prefers OpenAI models due to Anthropic's restrictions on "open code" usage.
*   **Specific Use Cases**: Opus is noted as being better for UI tasks compared to Codex.

### Workflows & Clustering (Future)

*   **Problem**: Guaranteeing multi-step operations (e.g., user registration involving a database write and an email send) where traditional database transactions are not possible across services. Server crashes can lead to inconsistent states.
*   **AI & Long-Running Processes**: AI integrations (chatbots, AI-driven processes) significantly increase average response times from milliseconds to minutes. This dramatically increases the probability of server failure during a process, even with a small user base.
*   **Effect Cluster / Workflows**: Effect offers a solution (currently unstable but nearing stability) that guarantees a procedure will finish even if a server crashes, by moving its execution to another instance in a cluster.
*   **Relevance**: This capability is becoming increasingly critical as more companies integrate AI into their products, leading to longer-running and potentially more failure-prone processes.

## Key Technical Concepts

*   **LLM Context Window**: The limited, fixed-size memory buffer that an LLM uses to process information for a given interaction. Information outside this window is not directly accessible.
*   **Reinforcement Learning (RL)**: A machine learning paradigm where an agent learns to make decisions by performing actions in an environment and receiving rewards or penalties. Coding agents are often fine-tuned using RL on vast codebases.
*   **Effect (Library)**: A TypeScript library for building robust, type-safe, and composable applications. It leverages functional programming principles, dependency injection (via "Layers"), and structured concurrency to manage complexity and ensure reliability.
*   **TypeScript Go (TSGo)**: A high-performance TypeScript compiler written in Go, designed to offer significantly faster type checking compared to the standard TypeScript compiler. (Mentioned as an attempted integration in the workshop).
*   **OpenAPI**: A standard, language-agnostic interface description for REST APIs. It allows both humans and machines to discover and understand the capabilities of a service without needing to access its source code.
*   **Effect SQL**: An Effect-based library for interacting with SQL databases, providing type-safe queries and schema definitions, enhancing reliability and developer experience.
*   **Effect Test**: The testing framework integrated within the Effect ecosystem, specifically designed to facilitate testing of Effect-based code, particularly when dealing with its dependency injection system (Layers).
*   **`it.layer`**: A utility within `Effect Test` that simplifies the process of providing Effect Layers (dependencies) to individual tests or test suites, ensuring proper setup and teardown for isolated and reliable testing.
*   **ESLint**: A pluggable linting utility for JavaScript and TypeScript. In this context, it's used to enforce coding standards and provide "backpressure" to LLMs by automatically flagging undesirable code patterns as errors.
*   **Branded Types**: A TypeScript pattern where a unique, nominal type is added to a primitive type (e.g., `string`) to prevent accidental assignment of semantically different but structurally identical types, enhancing type safety.
*   **Effect Cluster / Workflows**: Effect's solution for building resilient, long-running, and fault-tolerant processes. It ensures that a procedure will complete its execution even if individual server instances crash, by coordinating across a cluster.

## Practical Applications

*   **Provide LLMs with Direct Source Code Access**: For complex libraries, clone their repositories directly into your project (e.g., using `git subtree`). This allows coding agents to learn patterns and best practices directly from the source, leading to higher quality and more idiomatic code generation than relying on documentation alone.
*   **Enforce Strict Type Checking for Agent Feedback**: Configure your TypeScript project with the strictest possible diagnostics, treating all warnings as errors. This creates an immediate and unambiguous feedback loop for the LLM, forcing it to produce type-safe code and catching errors early in the development cycle.
*   **Guide Agents with Explicit Instructions and Pattern Files**: Create dedicated markdown files (e.g., `agents.mmd` for commands, `patterns/http-api.md` for best practices) to explicitly instruct your coding agent. These files should list available tools, specify reference repositories, and document preferred coding patterns and conventions.
*   **Implement ESLint as a "Backpressure" Mechanism**: Develop custom ESLint rules to enforce specific coding standards and prohibit common LLM-generated anti-patterns (e.g., `as any`, `as unknown`, explicit type assertions, or specific structural issues). This acts as a crucial "backpressure loop" to steer the agent towards desired code quality.
*   **Adopt Spec-Driven Development with LLMs**: Instead of relying on an LLM's internal "plan mode," collaborate with the model to define a detailed specification for a feature in a markdown file. Once the spec is finalized, instruct the model to implement it, using the spec as its primary guide.
*   **Automate Pattern Generation and Adaptation**: Recognize that optimal patterns might vary based on the specific LLM or project. Consider automating the process of exploring libraries and generating these pattern files, potentially even adapting them for different LLM models.
*   **Strategize LLM Prompting**: Be mindful of how you phrase prompts for different LLM models. For example, avoid "screaming" (using all uppercase) at GPT models, as it can lead to deoptimization, while Claude models might interpret it as emphasis.
*   **Utilize `it.layer` for Robust Effect Testing**: When testing Effect applications, leverage `it.layer` to efficiently provide dependencies (Layers) to your tests. For database-intensive applications, consider implementing transactional tests with rollback to ensure test isolation and speed.
*   **Plan for Workflows in Long-Running Processes**: For operations involving multiple, non-transactional steps (especially with AI integrations that introduce longer response times), explore workflow solutions like Effect Cluster. This guarantees process completion and system resilience against failures, even if individual servers crash.

## Notable Quotes

*   "What if the best way to get coding agents to use a library well is not better prompts, but giving them the library's actual source code?"
*   "LLMs like people treat them like a human brain but they are very different. We learn continuously... with LLMs, this does not happen."
*   "The only way I found the models to be good regardless of the language, regardless of what you use, is if you just clone the repo."
*   "As programmers, if our job is not to write code, our job should be to set up the repositories in ways that the models can act good on it."
*   "It's kind of babysitting a junior developer with a knife running through the kitchen instead of a kid running through the kitchen with the knife."

## Takeaways

- Give agents source code, not just documentation
- Strict TypeScript diagnostics catch agent mistakes early
- Agent instructions should be explicit about patterns and conventions
- The Effect ecosystem is well-suited for agent-assisted development due to its type safety

## Speaker Links

- Twitter: https://x.com/MichaelArnaldi
- LinkedIn: https://www.linkedin.com/in/michael-arnaldi-52858114a/
