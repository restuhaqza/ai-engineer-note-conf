# Demand-Driven Context: A Methodology for Coherent Knowledge Bases Through Agent Failure

| Field | Details |
| --- | --- |
| **Speaker** | Raj Navakoti |
| **Affiliation** | IKEA Digital |
| **Type** | Workshop |
| **Date** | 2026-05-05 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=_QAVExf_1uw) |
| **Tags** | `context-engineering` `knowledge-base` `agents` `failure-driven` `enterprise-ai` `demand-driven` |

## Summary

This workshop introduces a "demand-driven context" methodology designed to address the critical challenge of incoherent and incomplete knowledge bases in enterprise AI. Raj Navakoti argues that despite advancements in AI agents, enterprise productivity remains stagnant because agents lack access to crucial "red" or institutional knowledge, which often resides only in people's heads or is outdated and unreliable in existing documentation. Traditional "push" strategies, where teams attempt to curate and feed all knowledge to agents, prove inefficient and lead to significant manual data entry.

The proposed solution flips this paradigm: instead of guessing what agents need, the demand-driven approach uses agent failures as a signal for missing knowledge. By assigning agents real-world problems, observing where they fail, and then providing the specific context needed to resolve those failures, the knowledge base grows organically and precisely. This "pull" strategy, akin to how a new employee learns on the job, transforms agents from mere knowledge consumers into active knowledge managers, curating and documenting information in structured "context blocks" that are directly relevant to problem-solving.

The methodology emphasizes automation for scalability, using past work items like Jira tickets or incident reports to systematically identify and prioritize knowledge gaps. It advocates for version-controlled Markdown repositories (like GitHub) for managing these context blocks and leveraging a "meta model" to provide agents with a navigational map of domain relationships. Ultimately, this approach aims to surface the "unknown unknowns" within an organization's knowledge, enabling agents to become semi-autonomous and significantly boosting enterprise AI's value creation by focusing on the 20% of context that solves 80% of problems.

## Detailed Notes

### The Enterprise AI Knowledge Problem

- **AI's Memory Limitations (Memento Analogy)**: Raj likens current AI agents to the protagonist in the movie *Memento*, who cannot form new memories and constantly needs external notes. AI agents, despite their reasoning and computational power, struggle with institutional and domain-specific knowledge
  - "If you go and watch this movie, you don't need to watch YouTube or blogs to understand agents and MCP. This movie actually tells you about everything. Literally."
- **The Enterprise AI Productivity Gap**: Despite 88% of companies using AI, only 6% see significant value creation (McKinsey report). This gap exists because AI agents, while excellent at general tasks (code generation, PR reviews), fail to move business-critical Jira tickets or epics due to a lack of specific enterprise knowledge
- **Three Tiers of Enterprise Knowledge**:
  - **Green Knowledge**: General, publicly available information that LLMs are already trained on (e.g., API standards). Agents handle this well
  - **Orange Knowledge**: Internally documented knowledge that can be taught to agents via skills or extensions (e.g., specific internal processes)
  - **Red Knowledge (Institutional/Tribal)**: The most problematic tier. This knowledge exists primarily in people's heads, is rarely documented, or is outdated. Agents struggle significantly with tasks requiring this knowledge
- **Flaws of Traditional Retrieval Layers (RAGs, MCPs)**: The industry solution of building retrieval layers on top of existing knowledge bases (Confluence, Jira, SharePoint, GitHub) often fails because the underlying data is:
  - **Undeterministic**: Unpredictable in its quality and relevance
  - **Unreliable**: Information may be incorrect or incomplete
  - **Untested**: Engineers rarely perform evaluations to verify the value or accuracy of retrieved information
  - Raj's experience: Building numerous MCP servers yielded only 10-30% accuracy, leading to more human effort in data entry and gap-filling
- **The Monolith Knowledge Base**: Enterprise institutional knowledge often resembles a "monolith," characterized by:
  - **20% Outdated**: Information is no longer current
  - **20% Unreliable**: Accuracy is questionable
  - **10% Duplicated**: Same information exists in multiple, potentially conflicting, places
  - **40% Tribal Knowledge**: Critical information known only by individuals, never documented
  - The speaker draws an analogy to transforming monolithic legacy systems into microservices, suggesting the need to break down knowledge into useful "context blocks"
  - "Nobody is going to come to your company and fix your knowledge base. You have to fix it yourself."

### The Demand-Driven Context Methodology

- **Core Philosophy: Pull vs. Push**: Instead of a "push" strategy (humans curating and pushing all knowledge to agents), this is a "pull" strategy. Agents are given problems and "pull" the necessary information from humans, documenting it as they go
- **Analogy: New Joiner Onboarding**: A new employee is given tasks, and through attempting them, they identify missing information, ask questions, and gradually build their understanding. Agents should learn the same way
- **Research & Validation**: The approach has been tested with data sets, and a preprint was published in RXP in March, titled "Demand-Driven Context"

### The Agent Lifecycle: Failure-Driven Knowledge Acquisition

The methodology follows an iterative cycle:
1. **Agent Attempts Problem**: An agent is assigned a real-world problem or task
2. **Agent Fails & Demands Context**: The agent attempts the task, fails due to missing information, and provides a checklist of specific knowledge gaps
3. **Human Provides Context**: A domain expert provides the missing information to the agent
4. **Agent Curates Knowledge**: The agent takes the provided information, solves the problem, and then curates/documents this new knowledge into structured "context blocks" for future reuse
5. **Iterate**: This cycle is repeated with multiple problems, gradually building a coherent and relevant knowledge base
- **Analogy: Test-Driven Development (TDD)**: Similar to TDD, where failing tests drive the creation of necessary code, agent failures drive the creation of necessary knowledge

### Implementation & Scaling

- **Framework Components**: The approach can be implemented with any agent framework (e.g., Copilot, Claude) and typically involves a combination of skills, rules, agents, hooks, and a knowledge base storage mechanism
- **Live Demo: Incident Root Cause Analysis**:
  - An agent is tasked with performing RCA for an incident, which contains a mix of green, orange, and red knowledge requirements
  - The agent first performs retrieval from the "monolith" knowledge base
  - It then identifies specific undocumented terminologies and business logic it doesn't understand, surfacing "unknown unknowns"
  - "Unless you don't do this way, you will never know what is not documented."
  - After a human provides the missing context, the agent successfully processes the incident and curates new entities into its knowledge base. One incident can surface and curate 6-12 new entities
- **Confidence Improvement Over Iterations**: After 14 incident-solving cycles, an agent's confidence score improved from 1.5 to 4.4 (on a 5-point scale)
- **Automating Knowledge Gap Discovery**: The process can be automated by feeding archived work items (Jira tickets, customer support tickets) to the agent framework
  - **Steps**:
    1. **Generate Probes**: The agent writes basic tests to validate the existing knowledge base against the work item
    2. **Run Tests**: The agent executes these tests
    3. **Analyze Gaps**: The agent identifies what knowledge is good, stale, incomplete, or entirely missing
  - **Output**: A consolidated list of knowledge gaps, classified by criticality (critical, high, medium), and a Kanban board for human domain experts to prioritize and fix
- **Agent as Knowledge Manager**: This approach shifts the agent's role from a passive consumer of knowledge to an active manager and curator of the knowledge base

### Knowledge Management & Structure

- **Preferred Storage: GitHub Repository**: Raj strongly recommends storing curated context blocks in a GitHub repository
  - **Benefits**: Inbuilt version control, PR processes, review mechanisms, and collaboration features
  - **Format**: Markdown files are preferred for accessibility and ease of use
  - **Flexibility**: Knowledge can be published from GitHub to other platforms like Confluence or Slack
- **The Importance of a Meta Model**: A meta model defines the relationships within an organization's domain (e.g., how business processes relate to systems, systems to APIs, and technical jargon to specific components)
  - Provides agents with a "map" to navigate the knowledge base, enabling them to understand impact of changes and make more informed decisions

### Value Proposition & Considerations

- **Key Benefits**:
  1. **Knowing the Unknown**: Surfacing undocumented, tribal knowledge that would otherwise remain hidden
  2. **Agent-Managed Knowledge**: Shifts the burden of knowledge management from humans to agents
  3. Improved agent autonomy and performance
- **Drawbacks and Limitations**:
  1. **Not for Small Teams**: Less relevant for small teams with already excellent, well-documented knowledge bases
  2. **Manual Pain**: Performing the cycles manually is tedious; automation is crucial
  3. **Early Approach**: The field of AI is rapidly evolving
- **Strategic Placement in the AI Pipeline**: The demand-driven context methodology sits between the "monolith" knowledge base and the "retrieval layer". Its purpose is to build curated "context blocks" that act like a cache database
  - **80/20 Rule**: 20% of the documentation is most useful for agents. These context blocks capture that 20%, allowing agents to solve 80% of problems

### Q&A Insights

- **Scaling Challenges & Scoping**: Best applied by scoping down to smaller teams or specific domains initially
- **Engineer Overload**: As agents become more autonomous, enterprises will shift focus to systematically fixing knowledge gaps
- **Codebase vs. Textual Documentation**: Applying the method to codebases yielded mixed results when combined with textual documentation. Conflicts arose regarding the "source of truth"
- **Evolving Agent Skills**: An interesting suggestion was made for "evolving skills" where the skill itself could adapt based on failures
- **Pre-retrieval Context Fixing**: Raj advocates for fixing context *before* agents operate in real-time rather than during operational tasks
- **Context Window & Cost**: Modern LLMs (like Claude with 1M tokens) can handle large context windows. Average domain knowledge is around 96K tokens, fitting easily. Directly providing context in the window can be more effective than complex RAG for smaller contexts

## Key Technical Concepts

- **Demand-Driven Context**: A methodology where knowledge need (demand) is driven by agent failures on real-world problems, rather than a top-down "push" of information
- **Context Blocks**: Curated, structured pieces of knowledge, often in Markdown format, that are highly relevant and useful for agents to solve specific problems. They act as a "cache" of frequently needed information
- **Monolith Knowledge Base**: The traditional, often disorganized, incomplete, and outdated collection of institutional knowledge within an enterprise
- **Agent as Knowledge Manager**: A paradigm shift where AI agents actively participate in identifying, acquiring, curating, and documenting knowledge based on their operational failures
- **Probes (Knowledge Tests)**: Automated tests generated by agents to validate the completeness and accuracy of an existing knowledge base against specific work items
- **Meta Model**: A structural representation of an organization's domain, outlining relationships between business processes, systems, APIs, and technical jargon
- **Green/Orange/Red Knowledge**: Classification of enterprise knowledge by accessibility -- publicly available, internally documented, and tribal/institutional respectively

## Practical Applications

- **Incident Root Cause Analysis**: Agents analyze incidents, identify missing diagnostic information or system knowledge, and curate that knowledge for future resolution
- **Jira Ticket Resolution**: Agents attempt to resolve tickets, flagging missing process documentation or tribal knowledge
- **Onboarding New Agents/Employees**: Systematically building out the knowledge base required for new agents to become effective
- **Knowledge Base Validation & Improvement**: Automating the "context gap scanner" across historical work items to continuously assess quality and prioritize documentation efforts
- **Identifying Gaps in Internal Tooling/Business Processes**: Observing agent failures when interacting with internal tools to highlight inefficiencies or missing functionalities

## Notable Quotes

- "Nobody is going to come to your company and fix your knowledge base. You have to fix it yourself."
- "Unless you don't do this way, you will never know what is not documented."
- "We are moving agent from consumer to a knowledge manager. So you just don't consume from me. I'm going to tell you but the whole knowledge management is also your job and you have to do it for me."

## Timestamps

| Time | Topic |
|------|-------|
| 0:00 | Introduction and speaker background |
| 2:47 | Analogy to the movie Memento and AI's memory constraints |
| 3:55 | Evolution of AI: From prompt engineering to deep agents |
| 4:33 | Enterprise AI challenge: Why productivity isn't moving |
| 5:33 | The problem: Green, Orange, and Red knowledge |
| 10:11 | The Monolith: Why institutional knowledge is outdated |
| 11:24 | Solution introduction: Demand-driven context |
| 13:05 | The "Pull" strategy: Learning by doing |
| 14:48 | The agent lifecycle: Problem to discovery to documentation |
| 17:46 | Demo introduction: Using a framework for context management |
| 19:12 | Live demo: Incident root cause analysis and context discovery |
| 24:05 | Scaling: 14 incidents to show confidence level improvement |
| 26:27 | Automated scale: Validating knowledge across the monolith |
| 33:01 | Storage strategy: Why GitHub is preferred |
| 34:47 | The Meta Model: Navigating domain relationships |
| 36:27 | Value proposition: Knowing the unknown |
| 39:02 | Summary: The 80/20 rule and cache-based context blocks |
| 40:15 | Workshop takeaways |
| 43:33 | Q&A Session |

## Takeaways

- Stop guessing what agents need; let failures tell you
- The pull strategy (demand-driven) beats the push strategy (top-down curation)
- Knowledge bases should grow organically from real agent failures
- The 80/20 rule applies: 20% of context solves 80% of problems
- Use version-controlled Markdown repositories for knowledge management

## Speaker Links

- LinkedIn: https://www.linkedin.com/in/raj-navakoti-529880b1/
