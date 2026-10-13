# Skills at Scale

| Field | Details |
| --- | --- |
| **Speaker** | Nick Nisi and Zack Proser |
| **Affiliation** | WorkOS |
| **Type** | Talk |
| **Date** | 2026-05-06 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=pFsfax19yOM) |
| **Tags** | `mcp` `skills` `tools` `chatgpt` `claude` `vscode` `copilot` `agent-tools` |

## Summary

This workshop, led by Nick Nisi and Zack Proser from WorkOS's Applied AI team, delves into the evolving landscape of agent-first development, where traditional coding is increasingly augmented or replaced by AI agents. They highlight a significant challenge: the "cold start" problem where every LLM conversation begins from zero, requiring constant re-provisioning of context. While memory files like `claude.md` offer some relief by appending instructions, they often lead to context bloat, non-deterministic behavior, and lack portability across projects.

The core solution presented is **Skills**, which are discrete, portable units of work that codify specific instructions, constraints, and even script executions for LLMs. Skills move beyond simple text prompts, allowing companies to deliver branded, functional, and interactive app experiences directly within chat hosts like ChatGPT, Claude, VS Code, Cursor, and Copilot. This approach leverages the Model Context Protocol (MCP) to enable interactive UI, deterministic script execution, and efficient context management, making LLMs more reliable and powerful for complex, repeatable tasks.

The talk emphasizes that Skills represent a fundamental shift in application distribution and development. Instead of building separate UIs, developers can embed rich, interactive experiences directly into existing chat and agent hosts. This "agent-first" paradigm allows for minimal investment (as little as 30 lines of markdown can yield hyperspecific feedback), cross-platform UI delivery, and a standardized way to teach LLMs how to perform tasks precisely as expected, addressing the limitations of generic, text-only interactions.

## Detailed Notes

### The Agent-First Workflow
- **Personal Experience:** Nick and Zach haven't written a line of code by themselves in 6-8 months, relying heavily on agents since the "Opus 35 days."
- **WorkOS Context:** WorkOS focuses on securing MCP and setting up authentication for agentic startups. They offer an AI installer to simplify setup.

### Challenges with Traditional LLM Interactions
- **"Starts from Zero" Problem:** Every LLM conversation begins without prior memory, requiring fresh context each time.
- **Context Reloading:** Users must constantly reiterate project specifics, conventions, and concerns, consuming significant time.
- **Memory Files (`agents.md`, `claude.md`):**
    - Can provide project-specific instructions (e.g., "use `pnpm` instead of `npm`").
    - **Limitations:**
        - Tied to specific repos or globally applied (bloating context).
        - Non-deterministic: LLMs can still ignore instructions or skip steps.
        - Lack built-in script execution for deterministic results.
        - Not easily transferable or portable to other projects.

### Skills as the Solution
- **Definition:** Discrete units of work that encode important, non-repetitive information.
- **DRY Principle:** Applies the "Don't Repeat Yourself" pattern to the agentic era.
- **Benefits:**
    - **Portability:** Shareable across codebases and teams.
    - **Determinism:** Use scripts to inject real, consistent data.
    - **Composability:** Small, focused units of work.
    - **Efficient Context:** Only applicable skills are loaded, avoiding context window bloat (unlike `claude.md`).
    - **Minimal Investment:** As little as 30 lines of markdown can provide hyperspecific feedback.
    - **Codified Expectations:** Teach the LLM to perform tasks in a specific, repeatable format (e.g., a report in an exact way).

### Skill Structure
- **`skill.md` (or `skill.mmd`):** The core markdown file.
- **Frontmatter (YAML-based):**
    - `name`: Name of the skill.
    - `description`: **Crucial for LLM routing.** This is how the AI determines if the skill is relevant to the task. Not for humans.
- **Additional Context:** Can include more detailed instructions.
- **Scripts:** On-ramp for interleaving determinism with non-deterministic LLM conversations.
- **Skill as a Folder:** More accurately, a folder containing `skill.mmd`, references, scripts, images, etc.

### Skill Design Principles
- **Constraints over Prescriptions:**
    - Better performance by providing a few constraints (e.g., "never be vague," "cite code with line and git commit reference") rather than overly prescriptive instructions.
    - Overly verbose markdown files are a common failure mode.
- **Script Interpolation (`!```):**
    - Claude (and potentially Pi) supports executing shell commands within markdown using `!```command```.
    - **Example:** `Stale To-Dos: !```git grep -i "TODO" | grep -v "DONE"``` `
    - **Power:** Provides deterministic data (e.g., latest 10 commits in a specific format) directly to the LLM, preventing speculation and token waste.
- **Descriptions as Routing Rules:** Primarily for the AI to decide when to use a skill.
    - **Example:** Different image generation skills for a personal blog (pixel art) vs. work (formal, S3 images).
    - **Self-Correction:** Ask Claude, "When would you load this currently?" or "Is this the best description if I only want it to run in these conditions?"

### Skill Distribution and Loading
- **Local to Repo:** Place a `.claude/skills/skill-name/skill.mmd` directory within a repository.
- **Global:** Place `.claude/skills/skill-name/skill.mmd` in your home directory.
- **`npx skills`:** A tool (e.g., from Vercel) that symlinks skills into these directories for easy installation.
- **Cross-Platform Support:** Codec, Claude (desktop, web), Cursor, Pi, Copilot.
- **Non-Technical Users:** Skills can be used and shared by non-technical team members (e.g., recruiting team using Claude Desktop with Slack/Notion/recruiting software connectors).
- **Sharing Skills:**
    - **`.zip` to `.skill`:** Zip a skill folder, rename to `.skill`, and drag into Claude Desktop for easy sharing (though not ideal for versioning or credentials).
    - **Marketplaces:** Claude's plugin marketplace, internal Git repos.

### Skill Development Loop
1.  **Edit Skill:** Modify the `skill.mmd` and associated files.
2.  **Save:** Save changes.
3.  **Invoke:** Run the skill.
4.  **Observe Output:** See what the LLM produces.
5.  **Test/Iterate:** Repeat the process.
-   **Claude's Skill Builder:** A built-in skill that can critique, set up, and evaluate your skills. Ask it, "Have I done this right?" or "When would this apply?"

### Managing Shared Skills (Challenges & Approaches)
-   **"My Skill, Your Skill" Problem:** Engineers creating similar skills, leading to duplication and conflicts.
-   **Version Control:** Challenges with merge requests for skill changes, multiple versions of similar skills.
-   **Evaluation:** Who evaluates skills for verbosity or relevance as new models emerge?
-   **WorkOS Approach:**
    -   **Public Skills Repo (GitHub WorkOS Skills):** Generic, open-source skills.
    -   **Internal Skills:** More specific to WorkOS engineers (e.g., O specialist, DX specialist, ghostwriter).
    -   **Personal Plugin Marketplace:** Individual engineers load skills applicable to them.
    -   **Monorepo Integration:** Skills specific to a monorepo live within it.
-   **Potential Solutions:**
    -   **Plugin System:** Install versioned plugins (like npm packages) to manage forks.
    -   **Tooling:** Tools that read from repos and install skills into various places.
    -   **Packaging:** Grouping skills (e.g., front-end skills package).
    -   **LLM Assistance:** Ask Claude's skill builder to analyze workflow and suggest skill splits or improvements.

### Skill Pickup and Overrides
-   **LLM Ignoring Skills:** Models might ignore a skill or decide to perform a task without it.
-   **Solutions:**
    -   **Specific Descriptions:** Ensure the `description` is precise for routing.
    -   **Slash Commands:** Explicitly call a skill by name (e.g., `/workos`).
    -   **Forced Execution:** Use `!` to force a command or script execution.
-   **Overrides:** If LLM doesn't respect overrides, ask Claude directly: "Why did you pick that over the other thing? How can I improve it?"

### Sub-agents vs. Skills
-   **Sub-agents:** Ideal for tasks requiring a standalone context, allowing them to perform extensive work without bloating the main task's context window.
    -   **Example:** An ideation plugin uses sub-agents for reviews, reporting back only problems, thus not eating the full context window of the main task.
-   **Skills:** Discrete units of work, often used *by* agents or sub-agents.

### Skill Libraries and Examples
-   **Superpowers:** A library with helpful skills (e.g., brainstorm, web browser mocking).
-   **Slide.dev:** Used to write the workshop slides, demonstrating non-coding applications.
-   **Remotion Skill:** Creates videos from prompts, even launching a web-based editor for real-time adjustments. Used for demo videos from Git history.
-   **Nano Banana Skill:** Python/TypeScript wrapper around Google's image generation model.
    -   Takes a single prompt (e.g., "child running through a field").
    -   Generates a static image.
    -   Uses a video API (e.g., VO) to animate the static image.
    -   Used for interstitial scenes in a 32-minute film.
-   **Claude's Official Plugins:** Includes a "skill reviewer" or "skill creator" skill.
-   **Obsidian Connector:** For reading/writing to Obsidian vaults, useful for daily notes and consolidated memory.
-   **OpenAI Codex Skills:** A skill from OpenAI that pipes Claude's output to Codex for review.

### Progressive Disclosure
-   **Concept:** Load additional information (e.g., specific markdown files) only when relevant to the current task or skill phase.
-   **Benefit:** Prevents context window bloat.
-   **Example:** A "migration skill" can point to specific migration guides (e.g., `o0-migration.md`) only when needed.
-   **WorkOS Skill Router:** A `reference_map` within a skill that routes to pertinent information (e.g., `offkit-nextjs.md` only loaded if working with Next.js).

### Confidence Scoring
-   **Concept:** An internal counter or rubric for the LLM to assess its understanding and readiness to execute a task.
-   **Ideation Skill Example:**
    -   Asks clarifying questions to the user to improve "problem clarity," "goal definition," "success criteria," "scope boundaries," and "consistency."
    -   Uses Claude's `ask_user_question` tool to prompt the user for more information.
    -   Aims for a high confidence score (e.g., 95%) before proceeding, leading to better output.
-   **Value:** Forces the LLM to "think more" and clarifies user intent, even if the underlying math is "fuzzy."

### Iterative Skill Refinement
-   **Process:**
    1.  Build an initial skill (manual or with Claude's skill builder).
    2.  Use it for a period (days/week).
    3.  Evaluate its performance (using JSONL logs of conversations).
    4.  Identify shortcomings, edge cases, or annoying omissions.
    5.  Iterate and improve the skill.
-   **Meta-Skills:** Skills that analyze the performance of other skills or how Claude is being used to suggest improvements.
-   **Example:** If a skill consistently asks for Git information, add a `!```git log``` command to provide it deterministically.

### Skills Beyond the Editor (Broad Applications)
-   **Code-Related:**
    -   **Bus Factor:** Identify key contributors using `git shortlog`.
    -   **Commit Crimes:** Analyze commit messages.
    -   **Zombie Branches:** List unused branches.
    -   **Readme Drift:** Check if `README.md` accurately describes the current codebase.
    -   **Code Review:** Automate aspects of code review.
    -   **CI Pipelines:** Integrate skills into CI/CD for automated tasks (e.g., updating documentation, generating demo videos on merge).
-   **Non-Code Related:**
    -   **Blog Writing:** Ensure uniform tone, format, and conventions.
    -   **Image Generation:** Generate specific styles (e.g., pixel art).
    -   **Sales/Customer Reports:** Generate standardized reports.
    -   **WorkOS CLI:** The `workos install` command is entirely skills-driven, using the Claude Agent SDK to provide a zero-friction setup experience for WorkOS authentication.
    -   **Automating Cognitive Resistance:** Turn nagging, repetitive tasks (e.g., context switching between Slack, code, and Linear for ticketing) into skills.

### Measuring Skill Effectiveness (Eval Framework)
-   **Claude's Eval Framework:** Ships with Claude, generates HTML reports showing before/after performance with skills.
-   **Metrics:** Tracks confidence, accuracy, and grades. Fails if skill performance is worse than without.
-   **"Apple Watch" Analogy:** Not perfectly accurate, but provides a general baseline and vector for improvement.
-   **Example:** Over-prescribing in a Next.js installer skill led to a 30% drop in accuracy, identified by evals.

### Memory Management
-   **Claude's Built-in Memory:** Claude has its own memory system.
-   **`memory.mmd` Files:** Can be used internally by agents (e.g., a DX engineer agent named "Case" uses framework-specific memory files).
-   **Retrospective Analysis:** Agents can analyze their own performance and update memory files to avoid spinning in circles.
-   **Obsidian Connector:** Useful for reading/writing daily journals and consolidating memories.
-   **OpenClaw:** Ships with a surprisingly good memory system using daily journal MDs and consolidated memory.

### Audience-Specific Disclosure
-   Skills can adapt their behavior based on the user.
-   **Example:** Use `git config user.email` or commit count to tailor the "repo roast" tone (gentler for new hires, more critical for experienced contributors).

## Key Technical Concepts

### Model Context Protocol (MCP)
MCP is the underlying protocol that enables interactive UI and functional app experiences within various chat hosts (ChatGPT, Claude, VS Code, Cursor, Copilot). It allows tools to manifest as visual, interactive components rather than just text, maintaining context within the host application. This is the emerging standard for UI inside chat, fundamentally changing how applications are distributed and consumed.

### Skills
Skills are codified, portable units of work for LLMs. They encapsulate specific instructions, constraints, and executable scripts within a structured markdown file (`skill.mmd`) and an accompanying folder. Unlike generic prompts or static memory files, skills provide deterministic execution through script interpolation, efficient context management via progressive disclosure, and clear routing rules for the LLM through their `description` frontmatter. They allow for consistent, repeatable, and high-quality outputs across different LLM platforms.

### Script Interpolation (`!```command```)
This feature allows LLMs (like Claude) to execute shell commands directly within the markdown content of a skill. The output of the command is then injected back into the LLM's context, providing deterministic, real-time data. This is crucial for tasks requiring up-to-date information (e.g., `git log`, `grep`) and significantly reduces token usage and non-deterministic guessing by the LLM.

### Progressive Disclosure
A technique used within skills to manage the LLM's context window efficiently. Instead of loading all possible information upfront, a skill can reference external markdown files (e.g., `testing.md`, `migration-guide.md`) and instruct the LLM to load them *only* when that specific context is relevant to the current task. This prevents context bloat, improves performance, and allows for highly specialized, yet modular, skills.

### Confidence Scoring
An internal mechanism within an LLM (often guided by a skill) to assess its understanding of a task and its readiness to execute. By defining a rubric for confidence (e.g., problem clarity, goal definition, success criteria), the LLM can iteratively ask clarifying questions using tools like `ask_user_question` until it reaches a desired confidence threshold. This iterative loop ensures a deeper understanding of user intent and leads to more accurate and desired outcomes.

## Practical Applications

-   **Codify Repetitive Tasks:** Identify any task you find yourself explaining repeatedly to an LLM or performing manually, and turn it into a skill. This applies to coding (e.g., "repo roast," commit analysis) and non-coding workflows (e.g., blog post generation, report formatting).
-   **Prioritize Constraints over Prescriptions:** When designing skills, focus on defining *what not to do* or *what rules to follow* (e.g., "never be vague," "always cite code with line numbers") rather than dictating every step. This allows the LLM more flexibility while ensuring adherence to standards.
-   **Leverage Script Interpolation for Determinism:** For any task requiring specific, up-to-date data (e.g., current Git status, file contents, API responses), use `!```command``` to inject the exact output into the LLM's context. This avoids speculation and ensures consistent results.
-   **Use Progressive Disclosure for Efficiency:** Structure complex skills with references to sub-files that are only loaded when needed. This keeps the context window lean and focused, especially for skills with many potential branches or specialized knowledge.
-   **Implement Confidence Scoring for Clarity:** For critical tasks, build in a confidence scoring mechanism. Have the LLM ask clarifying questions until it's highly confident in its understanding of the problem, goals, and success criteria. This iterative dialogue improves output quality.
-   **Utilize Claude's Skill Builder:** Actively use Claude's built-in "skill builder" or "skill creator" skill to critique, refine, and evaluate your own skills. Ask it questions like "Is this skill well-designed?" or "When would you use this skill?"
-   **Automate "Nagging" Workflows:** Identify tasks that cause cognitive resistance or context switching (e.g., creating Linear tickets from Slack messages, generating demo videos from Git history) and automate them with skills, potentially using connectors to other services.
-   **Reflect on Transcripts and Evals:** Regularly review your LLM conversation logs (JSONL files) and use evaluation frameworks (like Claude's built-in eval) to identify areas where skills are underperforming or where new skills could be created. Treat these evaluations like an "Apple Watch" -- a directional guide for improvement.

## Takeaways

- MCP Apps represent a shift from text-only chat to interactive embedded experiences
- The protocol enables cross-platform UI delivery through a single implementation
- Agent-first distribution means building for the chat host, not the standalone app
- This is an emerging standard worth watching and adopting early

## Notable Quotes

> "Every single conversation that you have starts completely from zero. You're always just like passing in new information to it. You've got to reiterate how you do things. Claude, for example, never remembers that it ever talked to you." - Nick Nisi

> "It's a way to make things more portable. And you can use scripts to inject real data. And you can make them composable." - Nick Nisi

> "The value is in the iterative loop of like clarifying and clarifying your own thinking by by responding." - Zack Proser

## Speaker Links

- Nick Nisi: https://nicknisi.com/about/
- Zach Proser: https://zackproser.com/
