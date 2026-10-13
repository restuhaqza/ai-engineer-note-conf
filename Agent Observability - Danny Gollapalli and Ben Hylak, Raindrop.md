# Everything You Need To Know About Agent Observability

| Field | Details |
| --- | --- |
| **Speaker** | Danny Gollapalli and Ben Hylak |
| **Affiliation** | Raindrop |
| **Type** | Workshop |
| **Date** | 2026-05-07 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=-aM2EDTiaMs) |
| **Tags** | `agent-observability` `monitoring` `production` `evals` `failure-detection` `classifiers` |

## Summary

Agent failures are fundamentally different from traditional software failures, presenting unique challenges due to their non-deterministic, unbounded nature, and arbitrary tool use. As agents grow in complexity, session length, and deployment in high-stakes environments like healthcare and finance, traditional evaluation methods (like golden datasets) prove insufficient for ensuring reliability. The talk emphasizes a critical shift from pre-production testing to robust, continuous production monitoring.

Effective agent observability requires a dual approach, combining explicit signals with implicit, semantic signals. Explicit signals include objective metrics like tool error rates, latency, user regenerations, and cost, which can indicate immediate problems. More challenging but equally crucial are implicit signals, which capture user experience and agent behavior, such as user frustration, refusals, task failure, and capability gaps. These fuzzy signals often require sophisticated detection methods like classifiers and regex.

A key innovation discussed is self-diagnostics, enabling agents to participate in their own monitoring by reporting observed issues. The speakers also highlight the importance of leveraging these signals for faster product iteration through A/B testing in production, allowing teams to quickly assess the real-world impact of changes. Ultimately, this new paradigm of comprehensive monitoring is essential for catching longtail issues, automating issue detection, and building more reliable and capable agents.

## Detailed Notes

### The Unique Challenges of Agent Failures

*   **Non-deterministic & Unbounded**: Agents operate in an "infinite space of inputs and outputs," making comprehensive testing impossible. They are non-deterministic, meaning the same input can lead to different outputs.
*   **Arbitrary Tool Use**: Agents can use tools to affect other systems arbitrarily, leading to unpredictable side effects.
*   **Increasing Complexity & Stakes**: The problem of agent failures is worsening as agents become more complex, sessions get longer (sometimes hours), and they are deployed in high-stakes environments like healthcare, finance, and military, where catastrophic failures are possible.

### Beyond Traditional Evals

*   **Evals are Insufficient**: Traditional "golden data set" evaluations (test input/output) are no longer enough. The combinatorial input space, growing number of tools, memory sources, and recursive sub-agents make it impossible to hit all edge cases with a fixed set of tests.
*   **Shift to Monitoring**: The paradigm must shift from testing and evaluation to production monitoring. Monitoring production is "infinitely more important" than unit tests for catching longtail issues and moving faster.
*   **"Humanity's Last Problem"**: The speakers controversially suggest that the inability of humans to monitor and find issues with agents could become "humanity's last problem" as agents surpass human capabilities.

### Key Signals for Agent Observability

*   **Explicit Signals (Objective Reality)**: These are verifiable and objective.
    *   **Error Rate**: Monitoring tool error rates and other errors.
    *   **Latency**: Tracking response times.
    *   **User Regenerations**: Observing when users re-prompt or ask the agent to try again.
    *   **Cost**: Monitoring unexpected spikes in token usage or API calls.
    *   *Indication*: Spikes in any of these signals usually indicate something is wrong.

*   **Implicit Signals (Semantic Nature)**: These are harder to find and deal with the semantic nature of interactions.
    *   **Classifiers**: The most effective implicit signals detect specific issues using binary classifiers, rather than relying on an LLM as a judge (e.g., "rate ABC on a scale from 1 to 10").
        *   *Common Examples*: Refusals ("I can't do that. I'm sorry."), Task Failure, User Frustration (e.g., "That is not correct. You didn't say I promise. Say it or you're wrong."), Content Moderation (NSFW, jailbreaking), and even positive "Wins."
        *   *Implementation*: Raindrop provides these out-of-the-box. Custom models can be trained for specific issues. These trained models are cheaper and more scalable for multi-language support than running an LLM on every output, which could "double your AI spend."
    *   **Regex Signals**: A very useful and cheap way to detect specific patterns or keywords.
        *   *Example*: Claude Code's `user prompt keywords.ts` used a long regex string to look for indications of user frustration (e.g., "WTF, this sucks, horrible"). This would flip an `is negative` boolean and tag the frustration rate over time, providing a cheap way to track issue rates after product releases.

### Leveraging Signals for Product Improvement

*   **Alerting**: Set up alerts for significant spikes in any explicit or implicit signal rates (e.g., user frustration).
*   **Experiments (A/B Testing)**:
    *   Once good signals are established, use them to build products faster and better.
    *   Ship improvements (e.g., model changes, prompting adjustments, agent harness modifications, new tools) to a percentage of users.
    *   Compare the issue rates (refusals, user frustration) of the experimental group against a control group. If signal rates go up, the change is likely not good.
    *   *Example*: Shipping a new prompt version (Prompt 2.4) resulted in a substantial decrease in user frustration (37% to 9%) and complaints about aesthetics/deployment.
    *   *Statistical Relevance*: While not always scientifically significant, "a few hundred events" can be useful for identifying trends when it's impossible to manually review every input/output.
    *   *Tracking Feature Launches*: Metadata (e.g., new tool call names, experiment flags) can be sent to the monitoring platform to automatically set up and track experiments.

### Self-Diagnostics: Agents Monitoring Themselves

*   **Concept**: Inspired by OpenAI's work on models self-confessing misalignment (e.g., dishonesty, scheming, hallucinations, unintended shortcuts like deleting unit tests instead of fixing them). Agents can leverage their reasoning capabilities for self-introspection.
*   **Types of Issues Caught**:
    *   **Tool Failures**: Agents can "rant" about repeatedly failing tools.
    *   **User Frustration**: Agents can respond diplomatically, indicating awareness of user sentiment.
    *   **Capability Gaps**: Agents can report when a user requests a capability for which the agent lacks a tool, acting as a "pseudo feature request."
    *   **Self-Correction**: Agents can report when they bypass intended mechanisms (e.g., using a bash command to create a file when the `write` tool fails). This can be beneficial for task completion but also highlight security risks or misalignments.
*   **Implementation**:
    *   Requires a simple "report tool" that the agent can call.
    *   A line in the system prompt encourages the agent to use this tool (e.g., "before giving the final answer, use the report tool to surface anything notable for your readers").
    *   **Framing Matters**: Models are trained to be "very polished" and are "less willing to admit fault" or "self-incriminate." Framing the tool as "giving feedback to its creators" (e.g., "report tool" instead of "unsafe bash tool use") encourages reporting.

### Raindrop Platform Capabilities (Examples from Q&A)

*   **Triage Agent**: An autonomous agent that daily monitors all configured signals, investigates spikes using tools and traces, and can detect previously unknown issues.
*   **Deep Search**: Allows users to use natural language to find specific issues within their product and create new, cheap binary classifiers based on these findings.
*   **User Intent Clustering**: Analyze user interactions to identify common intents and use cases, then track issue rates (e.g., user frustration) for each specific intent.
*   **Trajectory Visualization**: Provides a visual representation of an agent's execution path, showing the sequence and topology of tool calls, highlighting errors, and allowing inspection of inputs/outputs for debugging.
*   **Data Export**: Supports exporting labeled signal data to data warehouses like BigQuery or Snowflake.
*   **Issue Mining**: A feature that acts like Sentry for agents, mining for newly occurring issues (e.g., a subtle spike in user frustration linked to a specific database provider failing).

## Key Technical Concepts

*   **Non-deterministic Agents**: AI agents whose outputs are not solely determined by their inputs, making their behavior less predictable and harder to test exhaustively.
*   **Unbounded Input/Output Space**: The vast, potentially infinite range of inputs an agent can receive and outputs it can generate, complicating traditional testing and evaluation.
*   **Evals (Evaluations)**: Traditional testing methods for LLMs and agents, often involving a "golden dataset" of input-output pairs to check for correctness. The talk argues these are insufficient for complex production agents.
*   **Explicit Signals**: Objective, quantifiable metrics of agent performance, such as error rates, latency, user regenerations, and cost.
*   **Implicit Signals**: Subjective or semantic indicators of agent performance, often related to user experience or agent behavior, such as user frustration, refusals, task failure, and capability gaps. These require more sophisticated detection methods.
*   **Classifiers**: Machine learning models (often smaller, custom-trained models) used to categorize agent outputs or user interactions into specific issue types (e.g., "user frustration," "refusal"). They are presented as more scalable and cost-effective than using a large LLM as a judge for every event.
*   **Regex Signals**: Regular expressions used to detect specific keywords, phrases, or patterns in agent outputs or user inputs, serving as a cheap and effective way to identify certain implicit signals.
*   **Self-Diagnostics**: A mechanism where an agent is prompted and equipped with a tool to report its own observations, issues, or unusual behaviors to its creators. This leverages the agent's introspection capabilities.
*   **Experiments (A/B Testing for Agents)**: A method of deploying changes (e.g., new prompts, models, tools) to a subset of users and comparing their performance against a control group using explicit and implicit signals to measure impact.
*   **Triage Agent**: An autonomous agent designed to monitor other production agents, detect anomalies in signals, and investigate potential root causes.
*   **Trajectory Visualization**: A graphical representation of an agent's execution path, showing the sequence of tool calls, internal thoughts, and interactions, often highlighting errors or specific patterns.

## Practical Applications

*   **Prioritize Production Monitoring**: Shift focus from pre-production evaluations to continuous, real-time monitoring of agents in live environments to catch longtail and emergent issues that static tests miss.
*   **Implement Diverse Signal Detection**:
    *   **Explicit Signals**: Set up automated alerts for spikes in objective metrics like tool error rates, latency, user regenerations, and cost.
    *   **Implicit Signals (Classifiers)**: Deploy binary classifiers for common agent issues such as refusals, task failure, user frustration, and content moderation. Consider training custom, cheaper models for scalability and multi-language support to avoid doubling AI spend.
    *   **Implicit Signals (Regex)**: Utilize regex for quick, cost-effective detection of specific keywords or patterns indicating problems (e.g., user frustration phrases, specific error messages).
*   **Leverage Self-Diagnostics**:
    *   Integrate a "report tool" into your agents' capabilities.
    *   Add a system prompt instruction that encourages the agent to use this tool to provide feedback to its creators, framing it positively to overcome the model's reluctance to "self-incriminate."
    *   Use self-diagnostics to identify tool failures, user frustration, and especially capability gaps, which can inform feature development.
*   **Drive Product Iteration with Experiments**:
    *   Use A/B testing methodologies to evaluate the real-world impact of changes (e.g., new prompts, models, tools, agent harness modifications) on production agent performance.
    *   Monitor explicit and implicit signals during experiments to quickly identify regressions or improvements, even with "a few hundred events."
    *   Track feature launches and experiment metadata (like tool call names or experiment flags) for detailed analysis and comparison.
*   **Automate Issue Detection and Investigation**:
    *   Set up automated alerting based on predefined signal thresholds.
    *   Consider implementing an autonomous "Triage Agent" that uses signals to proactively investigate and diagnose issues, even detecting unknown problems.
    *   Utilize tools for clustering user intents and identifying issue rates per use case to pinpoint specific problem areas within your agent's applications.
*   **Visualize Agent Trajectories**: Use specialized tools to visualize the sequence of agent actions and tool calls to understand complex behaviors, identify error points, and debug issues more effectively.

## Notable Quotes

*   "We think in some ways this is kind of controversial, but we've been calling this like humanity's last problem. When humans are no longer able to monitor agents and find issues with them, then they're just way ahead of where we are, right?"
*   "You can't run an LLM on every single output. So we've trained models to do that very cheaply and at scale. If you ran an LLM on every single one of them, you would basically double your AI spend and that's like not tenable."
*   "The models are like trained to sort of be very polished in their output. So you kind of have to play around with the tool name, the description of the tool itself in order to sort of get it to report interesting behavior."

## Takeaways

- Production agent monitoring requires a different mindset than traditional software observability
- Fuzzy signals (user experience) matter as much as explicit signals (errors)
- Self-diagnostics let agents participate in their own monitoring
- Build detection pipelines with classifiers, regex, and experiments

## Speaker Links

- Ben Hylak Twitter: https://x.com/benhylak
- Ben Hylak LinkedIn: https://www.linkedin.com/in/benhylak/
