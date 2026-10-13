# Playground in Prod: Optimising Agents in Production Environments

| Field | Details |
| --- | --- |
| **Speaker** | Samuel Colvin |
| **Affiliation** | Pydantic |
| **Type** | Workshop |
| **Date** | 2026-05-07 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=A48uhxfxbsM) |
| **Tags** | `pydantic-ai` `logfire` `agents` `production` `evals` `gepa` `prompt-optimization` `managed-variables` |

## Summary

Deploying an agent is merely the initial step; the true challenge and value lie in continuously improving agents once they are live in production. Samuel Colvin, creator of Pydantic, demonstrates how to achieve this post-deployment optimization using the Pydantic AI agent framework and the Logfire observability platform. The core idea revolves around making live parameter changes to agents—such as modifying prompts, switching models, or adjusting temperature—without the need for redeploying or restarting services, thereby drastically accelerating the iteration cycle.

The talk introduces two key technologies: GEPA (Genetic Pareto Optimization) and Logfire's managed variables. GEPA is an optimization library that employs a genetic algorithm to iteratively refine agent prompts, moving from manual, human-driven tweaks to automated, data-driven improvements against a "golden dataset." This process helps agents achieve higher accuracy and efficiency. Logfire's managed variables extend traditional prompt management by allowing any Pydantic model object to be dynamically updated in production, enabling A/B testing and controlled rollouts of new configurations.

Through a practical case study involving the analysis of political dynasties from Wikipedia data, Colvin illustrates how to set up robust evaluation pipelines (evals) against a golden dataset. He shows how to use these evals to measure agent performance, compare different prompt strategies, and then leverage GEPA to autonomously optimize prompts. The combination of Logfire's live configuration capabilities and GEPA's automated optimization provides a powerful infrastructure for maintaining and enhancing agent performance, reducing costs, and ensuring agents remain effective in dynamic production environments.

## Detailed Notes

### Introduction to Pydantic Ecosystem

*   **Speaker:** Samuel Colvin, creator of Pydantic.
*   **Pydantic Company Offerings:**
    *   Pydantic validation (core library).
    *   Pydantic AI (agent framework).
    *   Pydantic Logfire (observability platform).
*   **Logfire Philosophy:** Logfire is a general observability platform (OpenTelemetry, logs, metrics, traces). Samuel believes "AI observability" is a feature, not a category, that will eventually be absorbed by broader observability or AI platforms.
*   **Logfire Features:** Beyond standard observability, Logfire includes `evals` (evaluations) and `managed variables` (newer feature).
*   **Future of Logfire:** Working towards autonomous agent optimization directly from the platform.

### GEPA (Genetic Pareto Optimization)

*   **Concept:** A library for optimizing strings, which can be simple text prompts or JSON data.
*   **Mechanism:** Uses a genetic algorithm to find the "best" values. It takes candidates from the "Pareto frontier" (the best examples found so far), mixes them, and produces new candidates, similar to breeding racehorses.
*   **Goal:** To optimize prompts or other string-based configurations for improved agent performance.

### Managed Variables (Logfire)

*   **Evolution of Prompt Management:** Extends the idea of managing text prompts in a platform.
*   **Flexibility:** Can manage *any* object defined with a Pydantic model, not just text. This allows for dynamic changes to prompts, models, temperature, tools, and other agent parameters.
*   **Benefit:** Removes the deployment bottleneck, enabling live changes in production without redeploying or restarting services.
*   **Underlying Standard:** Uses the OpenFeature open standard for feature flagging and A/B testing.
*   **Future:** Aims for "self-driving managed variables" where the platform autonomously optimizes agent configurations based on evals.

### Case Study: Analyzing Political Dynasties

*   **Task:** Analyze Wikipedia articles of UK Members of Parliament (MPs) to identify political relations, specifically ancestors.
*   **Initial Challenge:** LLMs struggled to differentiate between relevant ancestral relations and irrelevant relations (spouses, children, siblings). The initial approach involved post-processing to filter out non-ancestors.
*   **Goal:** Optimize the agent's prompt to inherently filter for ancestors, improving efficiency and accuracy.
*   **Data:**
    *   Archived HTML of Wikipedia pages for MPs.
    *   `golden_relations.json`: A "golden dataset" containing the supposedly correct political relations for each MP, generated using a high-performing model (Opus 4.6) and manually checked.
*   **Agent Setup:**
    *   Pydantic AI agent configured for structured output (list of `PoliticalRelation` objects).
    *   Initial prompts: A simple one-liner and a more detailed "expert" prompt.
    *   Uses Pydantic AI Gateway for model access, allowing connection to various LLMs (Anthropic, OpenAI, Grok, Gemini) with a single API key.

### Evals (Evaluations) Process

*   **Purpose:** To measure agent performance against the golden dataset.
*   **Custom Evaluators:** Preferred over "LLM as a judge" for deterministic and reliable assessment.
*   **Metrics:** Primarily accuracy (correct, incorrect, partial scores for minor discrepancies).
*   **Initial Performance:**
    *   Simple prompt: ~85% accuracy.
    *   "Expert" prompt: ~92% accuracy (demonstrating improvement from better prompting).
*   **Importance of Data:** Evals require a comprehensive and diverse golden dataset for both training and validation to prevent overfitting.
*   **Variance:** To reduce variance in eval results, run each case multiple times (e.g., 100 times).

### GEPA Optimization in Action

*   **Proposer Agent:** A Pydantic AI agent is used to propose new system prompts for the main agent. This agent is given context about previous prompt performance.
*   **Iterative Process:** GEPA iteratively generates new prompts, evaluates their performance using the defined evals, and selects the best-performing candidates (Pareto frontier) to inform subsequent prompt generations.
*   **Demo Result:** Achieved 96.7% accuracy with the optimized prompt, significantly higher than manual prompts. The optimized prompt was notably verbose.
*   **Cost Reduction Example (Shopify):** Shopify used GEPA to optimize prompts and model choice, reducing costs from $5 million/year to $73,000/year for classifying Shopify sites.
*   **Prompt Verbosity:** A potential issue with GEPA generating very long prompts. A solution is to optimize subsets of prompt components (e.g., selecting from a list of sentences) rather than generating entirely new prompts.
*   **Overfitting:** Small test sets can lead to overfitting, where the optimized prompt performs well on the test set but poorly on unseen edge cases (e.g., excluding "aunts and uncles" if not in the test data).
*   **Model Dependence:** Optimized prompts are often specific to the model they were trained on. Changing the underlying LLM typically requires re-optimization.
*   **Relevance:** Optimization is most critical for tasks involving private data (where models aren't pre-trained) or when aiming to use cheaper/faster models effectively.

### Managed Variables in a FastAPI Web Server Demo

*   **Setup:** A simple FastAPI server with an agent that answers questions about MP data.
*   **`logfire.variable`:** A decorator used to define a Pydantic model (`MPConfig`) for agent configuration (instructions, model, max_tokens).
*   **Pushing Variables:** Configuration is pushed from local code to Logfire using `logfire push variables`. Requires a Logfire API key with variable permissions.
*   **Live Updates:**
    *   Demonstrated changing the agent's response language (French, German) by editing the `instructions` variable in the Logfire UI. The server updated its behavior without redeployment.
    *   Demonstrated changing the underlying LLM (Anthropic to OpenAI) live.
*   **Targeting:** Logfire's managed variables support A/B testing by allowing control over the percentage of calls using different variable values (via OpenFeature).
*   **Future:** Autonomous optimization of managed variables, where Logfire automatically adjusts configurations based on real-time performance metrics.

### Implicit User Feedback Collection

*   **Challenge:** Explicit feedback mechanisms (e.g., thumbs up/down) often have low engagement.
*   **Best Approach:** Implicit feedback based on user interaction.
    *   **Chatbots:** If a user says "You idiot, try again" or rephrases their question, it's strong evidence of poor agent performance. If they say "Thanks, that's great" or disengage, it suggests success.
    *   **Websites (Google's method):** Time spent on a page; quick return to search results indicates dissatisfaction.

### Q&A Highlights

*   **Internal Use Cases:** Logfire uses an internal agent for free-text-to-SQL conversion, which is optimized for good SQL generation. It cannot be demoed due to sensitive data concerns.
*   **Data Privacy:** For sensitive data (e.g., medical, legal), options include:
    *   Not sending raw input/output to Logfire, only recording categorical performance metrics.
    *   Enterprise self-hosting Logfire within a private VPC.
*   **Data Classes vs. Pydantic Models:** Data classes are canonical for simple Python objects when validation isn't required. Performance difference is negligible in these contexts.
*   **Context Engineering & Compaction:** Prompt optimization is a type of context engineering. Combining it with compaction strategies (e.g., for smaller models) is crucial in complex cases to manage context windows and reduce degradation.
*   **Optimization vs. Fine-tuning:**
    *   Fine-tuning is expensive (tens of thousands of dollars) and often superseded by newer foundation models.
    *   Optimizing prompts and agent components (model choice, compaction strategy, tool registration) can be more cost-effective, especially for specific domains with large amounts of private data.

## Key Technical Concepts

*   **Pydantic AI:** An agent framework built on Pydantic, designed for creating LLM-powered applications with an emphasis on structured output and type safety.
*   **Logfire:** An observability platform that provides comprehensive insights into AI applications, including logs, metrics, traces, and specialized features like `evals` and `managed variables` for AI-specific needs.
*   **Evals (Evaluations):** A systematic process of testing an agent's performance against a predefined "golden dataset" to measure metrics like accuracy, latency, and cost. Evals are fundamental for data-driven agent optimization.
*   **Golden Dataset:** A meticulously prepared collection of inputs and their corresponding correct outputs, serving as the ground truth against which agent performance is measured during evaluations.
*   **GEPA (Genetic Pareto Optimization):** A library that implements a genetic algorithm to optimize string-based configurations, such as LLM prompts. It iteratively generates, evaluates, and refines candidates based on their performance on a given task, aiming for optimal solutions along the Pareto frontier.
*   **Managed Variables:** A Logfire feature that allows developers to dynamically update configuration parameters (e.g., prompts, models, temperature settings, tool choices) in live production environments without requiring code redeployment or service restarts. It supports Pydantic models for structured configuration and integrates with OpenFeature for A/B testing.
*   **Proposer Agent:** An LLM-powered agent used within the GEPA optimization loop. Its role is to suggest new or improved prompts and configurations based on the evaluation results of previous iterations, driving the genetic optimization process.
*   **Context Engineering:** The practice of designing and refining the input context provided to an LLM (including system prompts, instructions, and retrieved information) to elicit desired behaviors and improve performance. This also encompasses strategies for managing context window limitations and degradation.

## Practical Applications

*   **Live A/B Testing and Rollouts:** Use Logfire's managed variables to A/B test different prompts, models, or agent configurations in production. Gradually roll out new versions to a percentage of users, monitoring real-world performance before full deployment.
*   **Automated Prompt Optimization:** Integrate GEPA with Logfire evals to continuously and autonomously improve agent prompts. This reduces manual effort and ensures agents adapt to new data or model versions.
*   **Cost and Latency Reduction:** Optimize prompts and agent architectures to enable the use of cheaper, faster, or smaller LLMs for tasks that might otherwise require more expensive, larger models, leading to significant cost savings (e.g., Shopify's example).
*   **Accelerated Iteration Cycles:** Decouple configuration changes from code deployments using managed variables, allowing immediate experimentation and adjustments to agent behavior without lengthy CI/CD pipelines.
*   **Robust Agent Performance Monitoring:** Establish comprehensive evaluation pipelines with golden datasets and custom evaluators in Logfire to objectively measure, track, and drive improvements in agent accuracy, reliability, and other key metrics.
*   **Secure Handling of Sensitive Data:** For applications dealing with private or confidential information, leverage Logfire's capabilities to obfuscate inputs/outputs, record only categorical performance metrics, or self-host the observability platform within a secure VPC.

## Notable Quotes

*   "I don't really believe in AI observability. I think it's a feature, not a category, and it will get eaten by either observability or AI at some point."
*   "The single best way of getting of evaluating the performance of a given question response pair is to look at what the user did next cuz if the user says, 'You idiot, try again,' you can get pretty strong evidence that they the agent behaved badly."
*   "Most people don't run evals or and don't run optimization. They write out a decent prompt. They ask their coding agent of choice, 'Does this prompt look good?' If it says yes, they kind of eyeball it and then they put it into prod and they worry about other things."

## Timestamps

| Time | Topic |
|------|-------|
| 0:00 | Introduction to Samuel Colvin and the Pydantic ecosystem |
| 1:29 | Overview of GEPA for prompt optimization |
| 3:02 | Introduction to Logfire managed variables |
| 3:55 | Case study: Analyzing political dynasties using Wikipedia data |
| 10:04 | Getting started: Setting up the environment and API keys |
| 16:55 | Running the initial evaluation (evals) against a golden dataset |
| 25:16 | Comparing different prompt performance |
| 34:00 | Running the full GEPA optimization process |
| 43:43 | Q&A: Handling prompt size and systemic errors |
| 57:01 | Demonstrating managed variables in a FastAPI web server |
| 1:11:06 | Discussing implicit user feedback collection |
| 1:15:42 | Q&A: Real-world internal use cases and context engineering |

## Takeaways

- Production agent optimization requires infrastructure for live parameter changes
- Evals against golden datasets are the foundation of data-driven optimization
- GEPA provides a practical path from manual to automated prompt optimization
- Managed variables remove the deployment bottleneck from iteration speed

## Speaker Links

- Twitter: https://x.com/samuelcolvin
- LinkedIn: https://www.linkedin.com/in/samuel-colvin/
- GitHub: https://github.com/samuelcolvin
