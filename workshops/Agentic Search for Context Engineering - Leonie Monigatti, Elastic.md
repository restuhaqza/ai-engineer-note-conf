# Agentic Search for Context Engineering

| Field | Details |
| --- | --- |
| **Speaker** | Leonie Monigatti |
| **Affiliation** | Elastic |
| **Type** | Workshop |
| **Date** | 2026-05-08 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=ynJyIKwjonM) |
| **Tags** | `context-engineering` `agentic-search` `retrieval` `semantic-search` `rag` `agents` |

## Summary

This workshop, led by Leonie Monigatti from Elastic, fundamentally reframes how we approach providing information to Large Language Models (LLMs). It argues that getting the right context into an LLM at the right time—a discipline termed "context engineering"—is not merely a retrieval task but primarily an "agentic search" problem, constituting approximately 80% of the effort. The core insight is that the choice and configuration of search interfaces often dictate the quality of LLM output more than the LLM itself.

The session traces the evolution from early, fixed Retrieval Augmented Generation (RAG) pipelines, which had inherent limitations like unnecessary retrieval or single-hop constraints, to more dynamic "agentic RAG." In this advanced paradigm, an intelligent agent actively decides *if*, *when*, and *how* to query various context sources, including local files, databases, the web, and memory. This dynamic decision-making allows for more nuanced and multi-step information gathering.

Monigatti delves into a taxonomy of search interfaces, from semantic search for fuzzy matching to structured database queries, shell-based retrieval for file systems, and purpose-built agent skills for domain-specific contexts. The workshop emphasizes that no single search method is a panacea; effective agent systems must combine multiple search interfaces into a robust "retrieval stack." Understanding the strengths, weaknesses, and common failure modes of each method is crucial for building less brittle and more capable LLM-powered applications.

## Detailed Notes

### The Core Problem: Context Engineering as Agentic Search
*   **Definition:** Context engineering is the art and engineering technique of deciding what specific information from all available context sources should be fed into an LLM's context window to generate the best possible responses.
*   **Beyond Curation:** While often referred to as "context curation," the critical component powering this process is the underlying *search tool(s)*.
*   **Leonie's Hot Take:** Context engineering is approximately 80% agentic search, highlighting the centrality of intelligent information discovery.

### Evolution of Retrieval Augmented Generation (RAG)
*   **Fixed RAG (Early Days):**
    *   **Mechanism:** User message directly used as a search query (typically vector search) to pull chunks from a database. User message + retrieved context fed to LLM.
    *   **Limitations:**
        *   **Always retrieves:** Even if no context is needed, leading to potential confusion for the LLM.
        *   **Single-hop:** Ineffective for complex questions requiring multi-step or iterative information gathering.
*   **Agentic RAG:**
    *   **Mechanism:** Replaces the fixed pipeline with a dynamic search tool. The agent decides *whether* to call the search tool, *when* to retrieve, *if* the retrieved information is relevant, and *if* further retrieval (e.g., query rewriting, multi-hop) is necessary.
    *   **Improvement:** Addresses the limitations of fixed RAG by introducing intelligent decision-making, but often still limited to a single context source initially.

### Diverse Context Sources for Agents
Agents need to access information from a multitude of locations:
*   **Local Files:** Code projects, scratchpads (working memory like `plan.md`), agent skills definitions.
*   **Databases:** Enterprise data, structured information.
*   **Web:** Real-time information, broad knowledge.
*   **Long-Term Memory:** A critical but debated context source, with discussions around whether it resides in local files or databases.

### Context Source Native Search Tools
Each context source often has specialized tools for efficient retrieval:
*   **Local Files:** `search_files` tool, skill loading tools.
*   **Databases:** Semantic search tools (vector similarity), general-purpose query execution tools (e.g., SQL, ESQL).
*   **Web:** Dedicated web search tools.
*   **Memory:** Specialized memory tools.

### The Versatile Shell Tool (Bash/Exec Tool)
*   **Functionality:** Allows agents to execute terminal commands (e.g., `ls`, `grep`, `curl`, custom CLIs).
*   **Versatility:**
    *   **Local Files:** Navigate and explore file systems (`ls`, `grep` for data).
    *   **Databases:** Interact via custom CLIs or `curl` commands (if exposed via HTTPS). Can even write entire scripts to connect and query.
    *   **Web:** Perform web searches using `curl`.
*   **Risks:** High risk of unintended actions (e.g., deleting files). **Requires sandboxing** and careful implementation (LangChain's `ShellTool` has no default safeguards).

### Fundamentals of Building Robust Search Tools (Common Failure Modes & Mitigation)
Building effective agentic search is challenging, with several common failure modes:
1.  **Agent doesn't call any tool:** Agent relies solely on its parametric knowledge, failing to retrieve necessary context.
2.  **Agent calls the wrong tool:** A frequent issue, e.g., calling a web search tool instead of a database tool.
3.  **Agent generates wrong search parameters:** Difficulty in formulating correct or complex parameters for a tool.

**Mitigation Strategies:**
*   **Tool Description (Crucial!):**
    *   Often overlooked, but the most important aspect. Avoid short, uninformative descriptions.
    *   Include: Core purpose, clear trigger conditions (when to use/not use), and relationships (e.g., "first call Skill A before Tool B").
*   **Agent System Prompt:** Reinforce tool usage instructions and relationships within the main system prompt.
*   **Parameter Complexity:**
    *   **Simple Parameters:** (e.g., `customer_id`, a search string) are generally easy for agents to generate.
    *   **Complex Parameters:** (e.g., full SQL/ESQL queries, multiple filter conditions, `top_k` values) significantly increase the difficulty and failure rate for agents.
*   **Error Handling:** Implement `try-except` blocks within tools to catch errors (e.g., invalid query syntax). Instead of crashing, return the error message to the agent, allowing it to self-correct and rewrite the query. This is crucial for general-purpose tools.

### Code Demos & Examples (LangChain, ElasticSearch, Bash)
The workshop demonstrated these concepts using LangChain, ElasticSearch, and local file systems.
*   **Setup:** GPT-5.4 Nano/Mini LLMs, LangChain, ElasticSearch cluster, conference session data.
*   **System Prompt:** Standard search agent instructions, augmented with details on data structure (e.g., text fields for embeddings, metadata for filters).

#### 1. Semantic Search Tool (ElasticSearch)
*   **Implementation:** Uses `genome_embeddings_v5` for embedding queries, `similarity_search` method with `top_k=3`. LangChain's `@tool` decorator converts a Python function into an agent tool.
*   **Failure Mode:** When asked about "GPA" (a specific acronym), the semantic search returned irrelevant results like "Gemma models" due to token similarity, highlighting its limitations for exact or keyword-driven queries.

#### 2. General-Purpose Database Query Tool (ESQL)
*   **Motivation:** To overcome semantic search limitations, allowing the agent to write full ElasticSearch Query Language (ESQL) queries.
*   **LLM Choice:** Switched to GPT-5.4 Mini, anticipating the increased complexity of query generation.
*   **Error Handling:** Implemented a `try-except` block to return ESQL syntax errors to the agent, enabling self-correction.
*   **Failure Mode:** Agent initially generated invalid ESQL, using `%` as a wildcard instead of `*`, resulting in zero search results. This demonstrated the challenge of complex parameter generation.

#### 3. Agent Skills for ESQL
*   **Concept:** Provides additional, progressively disclosed documentation for complex tools. A brief description is in the system prompt, but the full skill content is loaded into the context window only when needed.
*   **Implementation:** A custom ESQL skill provided basic syntax rules (e.g., double quotes for string literals, `*` for wildcards).
*   **Tool Description & System Prompt Reinforcement:** Explicitly instructed the agent to "always use ElasticSearch ESQL skill to generate the ESQL query before using this tool."
*   **Result:** Agent successfully loaded the skill, generated a valid ESQL query with the correct wildcard, and found the "GPA" session. It also demonstrated the ability to perform aggregations (e.g., counting sessions on a specific date) by outsourcing calculations to the search tool, which is more efficient and accurate than relying on the LLM to count.

#### 4. Shell Tool for Local File System
*   **Disclaimer:** Emphasized the risk of using `ShellTool` without sandboxing.
*   **LLM Choice:** GPT-5.4 Nano was sufficient, as LLMs are generally good at navigating file systems and generating shell commands.
*   **Implementation:** Agent used `ls` to explore folder structure and `grep` to search for keywords in session files.
*   **"Cheating" Semantic Search:** The agent surprisingly mimicked semantic search by chaining multiple synonyms (e.g., `regulate`, `compliance`, `constraints`, `GDPR`, `governance`) with `grep`. While it worked for the demo, this is inefficient and not scalable for truly semantic queries (e.g., "movies with animal superheroes").

#### 5. Custom CLIs with Shell Tool (Gina Grap)
*   **Motivation:** To provide a more efficient semantic search alternative for local files than chaining `grep` commands.
*   **Implementation:** Introduced `gina-grap` (a multi-vector embedding based tool) via the `ShellTool`. The system prompt explained `gina-grap`'s usage and differentiated its use from `grep` (fuzzy vs. exact matches).
*   **Result:** Agent correctly used `gina-grap` for semantic queries, finding relevant sessions efficiently on the first try.

## Key Technical Concepts

*   **Context Engineering:** The overarching discipline of strategically selecting and delivering relevant information to an LLM's context window to optimize its output. It moves beyond simple data retrieval to intelligent information discovery and synthesis.
*   **Agentic Search:** A dynamic, agent-driven approach to context retrieval. Instead of a fixed pipeline, an intelligent agent makes real-time decisions about *what*, *when*, and *how* to search across diverse information sources, adapting its strategy based on the query and intermediate results.
*   **Retrieval Stack:** The concept of combining multiple, specialized search interfaces (e.g., semantic search, keyword search, structured query languages, shell commands) into a cohesive system. This acknowledges that no single search method is universally optimal and that a layered approach is necessary for comprehensive context retrieval.
*   **Semantic Search:** A search method that uses vector embeddings to find information based on conceptual similarity rather than exact keyword matches. It's effective for fuzzy queries but can struggle with precise keyword or filtered searches.
*   **General-Purpose Database Tools:** Tools that allow agents to execute structured queries (like SQL or ESQL) directly against databases. These provide powerful capabilities for filtering, aggregation, and complex data manipulation, but require the agent to generate syntactically correct and semantically appropriate queries.
*   **Shell-based Retrieval:** Utilizing a shell tool (e.g., Bash, `exec`) to allow an agent to run arbitrary terminal commands. This offers immense flexibility for interacting with file systems (`ls`, `grep`), web services (`curl`), and custom command-line interfaces (CLIs), effectively turning the agent into a power user of the underlying operating system.
*   **Agent Skills (Progressive Disclosure):** A mechanism to provide agents with detailed documentation and instructions for complex tools or domain-specific knowledge. A brief description is always available, but the full, detailed skill content is loaded into the LLM's context window only when the agent decides it needs that specific information, optimizing context window usage.
*   **Low Floor, High Ceiling:** A user experience design principle applied to agent tools. "Low floor" tools are easy for the agent to use for common, simple tasks with minimal errors. "High ceiling" tools provide powerful, general-purpose capabilities for complex or unexpected queries, even if they require more sophisticated agent reasoning or iteration.
*   **Error Handling in Tools:** Implementing robust error detection and reporting within search tools. Instead of simply failing, the tool returns informative error messages to the agent, allowing the agent to understand the problem (e.g., invalid query syntax) and attempt to self-correct by rewriting its query or choosing a different strategy.

## Practical Applications

*   **Build a Retrieval Stack, Not a Single Pipeline:** Recognize that no single search method is a silver bullet. Combine specialized tools (e.g., `get_customer_by_id`) for common, predictable tasks with general-purpose tools (e.g., shell tool, ESQL executor) for unexpected or complex queries.
*   **Craft Detailed Tool Descriptions:** Invest significant effort in writing comprehensive tool descriptions. Include the core purpose, clear trigger conditions (when to use and when *not* to use), and explicit relationships with other tools or skills (e.g., "always use Skill X before Tool Y").
*   **Leverage Agent Skills for Complexity:** For tools requiring extensive documentation or specific syntax rules (like ESQL), implement agent skills. This allows the agent to access detailed guidance on demand without cluttering the main system prompt.
*   **Prioritize Robust Error Handling:** Embed `try-except` blocks or similar error-catching mechanisms within your search tools. Crucially, return informative error messages to the agent so it can understand why a tool call failed and attempt to self-correct, rather than the entire system crashing.
*   **Start General, Then Specialize:** If you don't yet understand your agent's typical query behavior, begin with general-purpose tools. Log the agent's interactions and identify common failure modes or repetitive multi-tool calls (e.g., 4-5 tool calls for a single question). This data will guide you in developing more specialized, efficient tools for frequently encountered patterns.
*   **Consider Hybrid Agent Architectures:** Explore combining different tool types for enhanced accuracy. For example, use a database tool for primary data retrieval and then a shell tool for verification or secondary exploration, as demonstrated by the Vercel experiment.
*   **Be Mindful of Parameter Complexity and LLM Choice:** The more complex the parameters a tool requires (e.g., generating a full query language statement), the more powerful your LLM needs to be, and the higher the likelihood of errors. Balance tool power with LLM capability.
*   **Sandbox Shell Tools:** Due to the inherent risks of granting an agent terminal access, always run shell tools in a sandboxed environment to prevent unintended system modifications or data loss.

## Notable Quotes

*   "Context engineering is about 80% agentic search."
*   "If you take home only one thing from today is that doing good search is incredibly difficult and that's why we have many different techniques to do search."
*   "I personally hate this slide because I feel like everyone in this room probably knows that the tool description is the most important aspect, but anytime I see a tool description, it's like the least effort, one sentence, and then you're wondering why your agent isn't calling the right tool."

## Takeaways

- Stop treating context as a retrieval-only problem; it's a search problem
- Build a retrieval stack, not a single retrieval pipeline
- Profile your failure modes to understand which search interface is missing
- The quality of agent output is bounded by the quality of context retrieval

## Speaker Links

- Twitter: https://x.com/helloiamleonie
- LinkedIn: https://www.linkedin.com/in/804250ab/
