# Cognitive Exhaust Fumes, or: Read-Only AI Is Underrated

| Field | Details |
| --- | --- |
| **Speaker** | Šimon Podhajský |
| **Affiliation** | Waypoint AI |
| **Type** | Talk |
| **Date** | 2026-04-08 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=u0TOSBbAw7c) |
| **Tags** | `apis-mcp-and-protocols` `coding-and-developer-tools` `agent-engineering` |

## Summary

Most personal AI projects chase agents that act on your behalf. Šimon Podhajský built the opposite: a system that knows him but cannot send, edit, or change anything on his behalf. It reads six data sources with no write access and writes its analysis only to a separate review workspace.

His term for the raw material is **cognitive exhaust fumes** — the digital activity left behind as a byproduct of thinking. Individually a trace looks like waste, but combined across apps it reveals patterns the apps themselves cannot see, such as gaps between intentions and actions, drifting attention, and fading relationships.

He demonstrates two uses: a weekly reflection assembled from all six sources, and a recommendation engine that maps his reading to people in his network he should discuss it with. Both produce suggestions only, keeping the action with the human.

The talk ends on risk. Read-only protects the record from modification but does not make it private: combining sources creates a tempting "mosaic" target, and shell access keeps the exfiltration path open. His claim is not that the system is secure, but that he has examined where it is not.

## Key Takeaways

- **Read-only is a design choice, not an unfinished feature.** The system observes and interprets; it never acts for you.
- **Cognitive exhaust = the byproduct of your thinking.** Waste alone, but a diagnostic signal when combined.
- **The cross-source signal is the product.** No single app knows your reading, commitments, and relationships together.
- **Three zones: sources, workspace, outputs.** The AI analyzes in the workspace and writes results to a separate location.
- **A read-only error costs nothing; a write error can be unbounded.** Personal AI works in high-stakes areas like relationships and reputation.
- **Untouched sources make better evidence.** If the AI writes into the data it studies, you can no longer tell human from machine patterns.
- **Observers and agents are different products.** Read-only is not a stepping stone to "real" agents.
- **Read-only is not secure by default.** The mosaic effect and the "lethal trifecta" both survive it.

## Detailed Notes

### What can an AI learn without acting for you? · 00:01
- The starting point is deliberately restricted: six sources, read access only, no write permissions.
- The whole personal-AI space is obsessed with agents that act on your behalf; this is something different.
- **Cognitive exhaust fumes** is his term for digital activity that is a byproduct of your cognition, like exhaust from an engine.
- Three valuable patterns emerge only across sources: **intention-action gaps** (what you meant vs. what you did), **attention drift**, and **relationship decay**.
- Your email client does not know what you journaled; your task manager does not know what you browsed. Only the combination reveals these signals.

### Three zones: sources, workspace, outputs · 01:29
- **Sources** supply personal activity and receive no AI writes.
- **The workspace** is where the analysis happens.
- **Outputs** hold results for human review, kept separate from the sources. Podhajský uses a separate Obsidian vault, but a separate Notion space or plain text files would work the same.
- Read-only does not mean the system cannot create files — only that it cannot modify the sources it observes.
- Keeping outputs separate makes the analysis something to review without rewriting the underlying record.

### Weekly reflection from six sources · 01:54
- Inspired by David Allen's Getting Things Done weekly review, but synthesized from all six sources instead of memory.
- The result is an "occasionally brutal" account of how the week was actually spent.
- Workflow: invoke a Claude skill (a reusable slash command), launch a Python script that gathers data from the read-only sources, call the Anthropic API with prepared prompts for structured outputs, then render a Markdown document.
- He opens the Markdown in Cursor. It surfaces themes of the week, tensions and conflicts, commitments and relationships, notable moments, and reflection questions.
- The relationship and commitment sections are revealing partly through omissions — what got no attention can matter as much as what did.
- His framing: this is not a productivity score, it is a reflection on how you are thinking.

### Who should I discuss this reading with? · 03:38
- The goal: given recent reading, who in his network should he talk to — beyond the same three habitual recipients.
- Four sources combine, none designed to work together. A plain-language request activates a cross-source query skill.
- A Vivaldi SQLite database (regularly ingested) shows the most-read articles and those still open in tabs; Clay MCP then searches his contacts.
- He calls the Clay search the weakest part because it is slow. Clay is his CRM — or, more personally, his friend-relationship system.
- The output is only a **suggestion**: nothing is sent, nothing is scheduled. The finished list even surfaces an article's author already in his network, with contact, rationale, and an opener.

### Keep action, and the feedback loop, with the user · 07:28
- Why not let it send the message? The risk is asymmetric: an incorrect read-only suggestion costs nothing if ignored, while an incorrect write can affect a relationship, career, or reputation.
- He would rather miss out on automated emails than accept a consequential misfire.
- There is also a subtler argument about evidence quality: once the AI writes into the sources it studies, the "exhaust" becomes a human-machine hybrid and you cannot attribute patterns.
- The observer still changes behavior, but the loop runs through the person: read, decide, then act — it does not close automatically through an AI rewrite.
- He extends this to a preference for reclaiming personal agency, even in drafting, while admitting that may be a hard sell to an audience building AI tools.

### An observer is a different product · 08:44
- He has tried OpenClaw on a read-only mount. His preference for a dedicated observer comes from value per interaction.
- His illustration: an agent saves thirty seconds on a weather check; an observer shows him he has been avoiding his most important project for two weeks. (His assessment, not a measured comparison.)
- He also sees less exfiltration risk and less contamination of the observed record.
- The product distinction stands on its own: agents perform actions, observers help you notice and interpret patterns.
- Read-only is not a capability users eventually graduate from — as he puts it, "A mirror isn't a broken butler."

### The mosaic effect and the lethal trifecta · 09:40
- The same cross-referencing that makes the observer valuable creates its central privacy risk.
- The **mosaic effect**: many small pieces of information combine into a revealing picture. A fragment that seems modest alone can become sensitive when connected. His own product description doubles as a threat description.
- Read-only access protects the source record from modification; it does not make the combined picture confidential.
- Simon Willison's **lethal trifecta** names three risky capabilities together: private data, untrusted content, and external communication.
- Podhajský initially thought read-only broke that combination. It does not — removing source writes removes some exfiltration channels, but shell access can still communicate externally, so the third leg remains.
- In normal operation the system still sends data to Anthropic over a mostly open network. His posture: not "this is secure," but "I have examined its exposure and decided which risks I will carry." The worst security posture, he says, is the one you have not examined.

## Resources

- [Simon Willison's lethal trifecta](https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/) — private data, untrusted content, external comms
- [David Allen on the GTD Weekly Review](https://gettingthingsdone.com/2018/08/episode-43-the-power-of-the-gtd-weekly-review/) — inspiration for the reflection
- [Claude Code skills documentation](https://code.claude.com/docs/en/skills) — reusable skills/slash commands
- [OpenClaw sandboxing and workspace access](https://docs.openclaw.ai/gateway/sandboxing) — read-only mount setup
- [Claude Opus 4.6 launch announcement](https://www.anthropic.com/news/claude-opus-4-6) — the one-million-token context reference
