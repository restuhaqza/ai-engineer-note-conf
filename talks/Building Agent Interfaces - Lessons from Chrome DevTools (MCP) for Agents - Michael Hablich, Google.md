# Building Agent Interfaces: Lessons from Chrome DevTools (MCP) for Agents

| Field | Details |
| --- | --- |
| **Speaker** | Michael Hablich |
| **Affiliation** | Google |
| **Type** | Talk |
| **Date** | 2026-06-05 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=_B4Pv9ttFgY) |
| **Tags** | `apis-mcp-and-protocols` `infrastructure-and-deployment` `safety-and-governance` `rag-context-and-search` |

## Summary

Coding agents can write code but often cannot check whether that code actually works. Michael Hablich, Google's product manager for Chrome DevTools, explains how his team built an agent-facing version of DevTools so an agent can open a browser, measure a page, and validate its own fixes. The demo uses Gemini CLI with the Chrome DevTools MCP server, but any MCP-capable client works.

The first attempt failed. The team assumed they could just throw browser data at the agent, but a raw performance trace was multiple megabytes and roughly 50,000 lines of JSON, which blew through the context window. The fix was to return short semantic summaries in Markdown instead, while keeping the raw trace available for other tools.

Hablich frames the rest of the talk around four engineering concerns: token cost, error recovery, tool discoverability, and trust boundaries. Each one is a trade-off, not a free win. His closing point is that agents are a new kind of user, with their own non-functional requirements, and that trust should never be traded away just to remove a click.

## Key Takeaways

- **Treat agents as a separate user segment.** They share the goal of finding and fixing errors, but they hit different bottlenecks than humans.
- **Return summaries, not dumps.** Semantic Markdown summaries beat huge raw traces; keep raw data available in case post-processing is needed.
- **Measure tokens per successful outcome.** Track effectiveness (did the task finish?) and efficiency (tokens, tool calls, time) together.
- **Compare within a task class.** Scraping and debugging cost different amounts, so a single global average hides the real picture.
- **Cut context without losing capability.** Tool categorization, slim mode, and a command-line interface each reduce tokens, each with a trade-off.
- **Make errors actionable.** A sentence explaining why something failed can let the agent recover on its own.
- **Description length is not description quality.** State the tool's purpose and when to use it; overly long text can bias smaller models.
- **Keep a real trust boundary.** The team deliberately kept an "Allow" click for browser access instead of optimizing it away.

## Detailed Notes

### Give the agent a way to check its work · 00:57
- Chrome DevTools is built into Chrome and used by millions of web developers to find errors, audit pages, and profile performance.
- The agent version puts those same abilities into an agent workflow. In the demo, Gemini CLI with the Chrome DevTools MCP server opens Chrome, records a performance trace, reads the results, edits the page, and measures again.
- MCP (Model Context Protocol) is the standard interface between the debugging server and the agent harness. The demo uses Gemini CLI, but Claude Code, Codex, and OpenClaw work too.
- The team shipped this because coding agents were "flying blind": good at generating code, unable to validate it.

### Replace the trace dump with semantic output · 03:29
- The early assumption was that machines could handle lots of data. That was wrong.
- A performance trace held several megabytes and about 50,000 lines of JSON. Passing it to common agent harnesses overflowed their context windows.
- The interface was the problem: the evidence was there, but the agent could not reason over that much of it.
- So performance endpoints began returning semantic summaries in Markdown, surfacing useful metrics like LCP (Largest Contentful Paint) and INP (Interaction to Next Paint).
- The analogy: point the reader at the relevant sentence instead of making them read a whole book. Humans and agents want the same result, but think differently.

### Measure token cost against successful journeys · 06:47
- Four concerns follow: token burn, error recovery, tool discoverability, and trust boundaries.
- Token burn matters because every token and every reasoning step costs money and time. Hablich compares a clunky API to a fourteen-step checkout flow.
- **Tokens per successful outcome** combines effectiveness (does the agent complete the whole task?) and efficiency (tokens, tool calls, duration).
- The fuel-efficiency analogy has a limit: efficient travel is worthless if you never arrive. So always measure success, not just cost.
- Compare within the same task class. Scraping a site is cheap; debugging a broken responsive layout is intricate and reasonably costs more.
- The team's internal chart uses longer bars for more effective use cases. Short bars mark journeys to improve next. Imperfect data still beats gut feeling.

### Reduce context without removing capabilities · 11:11
- **Tool categorization:** niche tools, like Chrome extension debugging, sit behind command-line parameters instead of loading into every session.
- **Slim mode:** pushes this further, exposing a tiny tool set (in the talk, `select_page`, `navigate_page`, `evaluate_script`). A smaller surface saves context but can force extra turns or drop needed capability, such as network inspection.
- **Command-line interface:** offers nearly the same functionality, letting the agent chain commands and process data locally.
- Example: fetch the page's accessibility tree, filter it with `grep` to find a control's ID, then pass that ID to a click command. The filtering happens on the computer, not in the model's context.

### Make failures lead to a next action · 13:12
- Errors cost tokens: the agent must understand the failure, decide what to change, and retry. Recovery affects both completion and cost.
- **Explain the reason.** In a back-navigation example, the chosen page had no history entry. Adding an explanatory sentence let the agent recover without a human.
- **Proactive detours.** Models have learned tool preferences. For profiling, the interface steers the agent to the performance-trace tool rather than a Lighthouse audit.
- **Diagnostic playbooks.** A troubleshooting skill gives recurring setup failures a structured recovery path.
- Together these make the interface resilient, so a fixable mistake does not become a dead end.

### Make tool intent discoverable · 15:27
- The first design was one broad tool, `Debug Webpage`. It looked clean, but it did not work well.
- The team split it into 25 tools, which created a new problem: choosing which tool to use, and when.
- A cited paper found that roughly 97% of analyzed MCP tool descriptions had quality problems. Descriptions matter because the schema is the agent's interface.
- Longer descriptions cost context and can push smaller models toward the wrong tools. Quality is not the same as length.
- Two things make a description useful: **purpose** (the tool's core function) and **activation criteria** (when to use it).
- The performance-tracing description names Core Web Vitals — LCP, INP, and CLS (Cumulative Layout Shift) — so the agent links a "make loading faster" request to the right tool.
- Skills help with complex workflows, but they move the same trade-off elsewhere: too many raise context and cause irrelevant calls.

### Match browser access to trust boundaries · 18:33
- **Auto-connect** lets a coding agent join the browser where a human is stuck. Users asked the team to remember their `Allow` choice.
- The team kept the friction on purpose. Delegating browser work needs an explicit trust boundary.
- The talk cites Simon Willison's "lethal trifecta" (private data access, exposure to untrusted content, and ability to communicate externally) as the lens.
- Three deployment tiers need different access models:
- **Local development:** a human in the loop grants time-bound access to the default Chrome profile.
- **Controlled CI (continuous integration):** separate the agent's data and environment with containers and separate Chrome profiles. A remote debugging port is a connection mechanism, not an isolation boundary.
- **Full-internet browsing:** public pages can carry prompt-injection attacks. Keep the CI separation, then add domain allowlists and injection mitigations.
- A local assistant and a fleet of research agents may share the DevTools interface, but they should not share the same security model.

### Agents are users with requirements · 21:31
- User experience is expanding to include agent experience: another user segment with its own non-functional requirements.
- Those requirements span efficiency, discoverability, security, and stability, across the whole task.
- Hablich's four takeaways: measure fuel efficiency with tokens per successful outcome, turn errors into recovery playbooks, audit descriptions for intent, and never compromise trust for convenience.

## Resources

- [Chrome DevTools MCP](https://github.com/ChromeDevTools/chrome-devtools-mcp) — installation and tool documentation.
- [Current slim tool reference](https://github.com/ChromeDevTools/chrome-devtools-mcp/blob/main/docs/slim-tool-reference.md) — current slim-mode tool names differ from the talk's examples.
- [MCP Tool Descriptions Are Smelly!](https://arxiv.org/abs/2602.14878v1) — the study behind the 97% figure.
- [The lethal trifecta for AI agents](https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/) — the security framework Hablich cites.
