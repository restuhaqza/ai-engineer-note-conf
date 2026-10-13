# Bending a Public MCP Server Without Breaking It

| Field | Details |
| --- | --- |
| **Speaker** | Nimrod Hauser |
| **Affiliation** | Baz |
| **Type** | Talk |
| **Date** | 2026-04-08 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=U00AOI1eJUE) |
| **Tags** | `agent-engineering` `developer-workflows-and-testing` `safety-and-governance` `apis-mcp-and-protocols` |

## Summary

Third-party tools are supposed to make an agent more capable — but used out of the box they can make an application less reliable. Nimrod Hauser, a founding engineer at Baz, walks through a browser-based code reviewer and shows why a generic MCP server needs tailoring before it works in production.

An **agentic tool** is a callable function plus a description that tells the agent when and how to use it. The worked example is Baz's Spec Reviewer, which reads a ticket and a Figma design, opens the branch in a browser via Playwright MCP, and checks whether the implementation matches the spec. In a baseline run it fails — wrong verdict, no usable screenshot.

Hauser then applies five interventions that turn the failure into a pass: curate the tool list, rewrite descriptions with task-specific guidance, add a deterministic path guardrail, compose a dedicated evidence-capture tool, and call mandatory setup steps (like login) directly instead of leaving them to the agent. All of it reuses the existing browser code.

The lesson is a balancing act: decide case by case whether to reduce the agent's choices (shorten context) or add guidance (lengthen it), and enforce anything too sensitive to leave to a probabilistic decision.

## Key Takeaways

- **Generic tools cause generic behavior.** A tool written for every possible use case will not fit yours out of the box.
- **A tool = a callable function + a description.** The description steers when and how the agent uses it.
- **Curate the tool list.** Remove operations your task never needs to shrink the context and the number of bad choices.
- **Rewrite descriptions for your use case.** Tell the agent which tool to call first and which output to prefer.
- **Descriptions guide; they do not enforce.** Sensitive operations need deterministic checks before execution.
- **Prefer accessibility snapshots over screenshots for understanding a page.** They describe controls as text; screenshots are for evidence.
- **Keep mandatory setup deterministic.** Login is required every run and awkward for agents, so call it outside the loop.
- **Nothing beats an unmodified tool list.** Task-specific tool shape, not raw capability, is what makes agents reliable.

## Detailed Notes

### Why adding third-party tools can hurt · 00:00
- A tool that should help can instead degrade reliability, produce wrong results, or execute awkwardly.
- **Agentic tools** are callable functions wrapped in a description. Hauser treats tools from MCP servers, libraries, or copied code the same way.
- The example is [Playwright MCP](https://github.com/microsoft/playwright-mcp) — browser automation whose descriptions must serve many different applications, so they stay generic.
- Security is a real concern: in a multi-tenant app, an agent may not understand which folders, databases, or schemas belong to which customer. Without guardrails, one client's data can leak to another.
- The framing: application-specific constraints must survive the tool call, even when the tool's author knows nothing about your architecture.

### The Spec Reviewer example · 04:51
- Baz's Spec Reviewer compares requirements against implementation automatically, saving product managers repetitive validation work.
- Workflow: read a ticket (Jira, Linear, etc.) and a Figma design, start Playwright, open the branch, then judge whether the implementation meets the spec.
- The toy project defines a screenshot output directory, configures one Playwright MCP server, and initializes an MCP client.
- A base class exposes `get_tools`; successive subclasses return increasingly customized tool sets. `main` runs `login_to_baz` *before* creating the agent — a detail explained at the end.
- The system prompt sets a meticulous QA role: read the ticket, navigate the app, return pass/fail with observations, and cite screenshots as evidence. The human message embeds the ticket and design images.
- The task: find the Spec Reviewer configuration drawer in the Agents tab and check that it matches the design. (Using a ticket screenshot is a toy shortcut, not the real product.)

### Baseline failure: wrong verdict, missing evidence · 10:26
- Version `v0` loads tools with LangChain's `load_mcp_tools` and changes nothing.
- The baseline exposes **21 tools** covering browser actions like close, resize, dialog handling, file upload, form fill, and key presses, with broad descriptions such as "Resize the browser window."
- Login succeeds and the agent takes over on the Changes homepage. Instead of finding the drawer, it tries an invented URL (`Baz.co/spec-reviewer`).
- The result: a **failed** verdict claiming the requirement is unimplemented, based on a failed navigation, and a 404 screenshot that was not even saved properly.
- The repair path is five interventions: curate tools, wrap descriptions, enforce a deterministic guardrail, compose a task-specific tool, and call selected functions directly. All reuse the third-party code.

### v1 and v2: curate tools, then rewrite descriptions · 15:55
- **v1 — curate.** Add an exclusion list and filter loaded tools with a list comprehension (exclude resize, drag, and running code in the page). The count drops from 21 to **16** tools, so the agent has fewer choices.
- **v2 — wrap.** `ToolWrapper.wrap_playwright_tools` keeps a dictionary mapping tool names to enhanced descriptions written from experience, including ordering guidance ("use this tool before clicking").
- Key distinction: an **accessibility snapshot** is text describing the page's controls and menu items, while a **visual screenshot** is an image. Despite its name, `browser_snapshot` produces the accessibility representation.
- The wrapper creates a replacement tool with the new description but **the same callable**, so execution is unchanged — only the model-facing instructions differ. It also filters a few more tools.
- At the breakpoint, remaining descriptions are visibly longer and include the intended guidance, e.g. telling `browser_click` to inspect the page first.

### v3: deterministic path guardrail · 22:53
- Descriptions can be ignored — Hauser cites needle-in-a-haystack and "Lost in the Middle" behavior — so sensitive operations need enforcement.
- The rule here is simple: visual screenshots must be saved under the configured output directory.
- **v3** adds a `PathValidation` class inside the tool wrapper. Before forwarding a screenshot request, it extracts the proposed destination and checks it lies under the screenshots root.
- The historical demo checks `path` or `filename`; the point is the *check before invocation*, not the exact argument names of today's schema.
- An invalid destination never reaches the original tool. Instead of raising and killing the run, the wrapper returns an agent-facing message naming the allowed destination and asking for a corrected filename — so the agent can retry. This is a bounded guardrail, not full filesystem or tenant isolation.

### v4: a dedicated evidence-capture tool · 28:45
- An exploratory screenshot and a final evidence screenshot use the same browser capability but serve different purposes.
- **v4** injects a separate **evidence tool** with its own description, reserving it for the end-of-review verdict. In the demo it is added only if the underlying screenshot tool was not filtered out (an explicit dependency policy).
- The description tells the agent to include the relevant ticket number in the saved filename, giving it both a selection rule and a naming convention.
- Two contracts over one operation: general screenshots during exploration, and evidence screenshots tied to a ticket.
- The wrapper is also a place to add purpose-specific deterministic checks or preparation before the real call.

### Call mandatory functions directly (login) · 32:53
- The fifth practice returns to `login_to_baz`. A tool's callable stays useful even without asking the agent to interpret its description.
- `main` gets the MCP tools and runs login **before** creating the agent, keeping mandatory setup under deterministic control.
- Authentication is a good candidate: every Spec Reviewer run needs it, and different client systems have different mechanisms and secrets. Hauser reports subpar results when leaving it to the agent.
- The toy version reads JWT tokens from environment variables, injects them into browser local storage, and clicks Login — a shortcut distinct from the real product's auth. Once authenticated, control passes to the agent.
- This avoids spending the agent's reasoning and context on a sequence the application already knows must happen.

### The rerun: a pass, with caveats · 36:38
- Removing the breakpoint and rerunning the same task, login runs first, then the agent navigates to Agents and finds the configuration drawer.
- The run reports a **pass**: the drawer is present with the required sections, and the evidence screenshot's filename contains the ticket identifier.
- Hauser notes the supplied design was dark mode while the app ran in light mode. The toy reviewer found the drawer and its contents but does not prove pixel-perfect matching; the real product supports finer checks of padding, margins, and styling.
- The recording shows one failed baseline and one successful improved run — no repeated reliability measurement.
- The closing trade-off: removing irrelevant tools reduces choices, while longer descriptions make the remaining choices better informed. Deterministic checks enforce constraints, and direct calls remove work that does not need agentic decisions.

## Resources

- [Playwright MCP](https://github.com/microsoft/playwright-mcp) — official browser automation MCP server
- [LangChain MCP Adapters for Python](https://github.com/langchain-ai/langchain-mcp-adapters) — loading MCP tools into agents
- [Spec Reviewer's Figma and Jira workflow](https://baz.ai/resources/blog/spec-reviewer-bringing-figma-and-jira-context-into-every-pull-request) — product background
- [Lost in the Middle](https://arxiv.org/abs/2307.03172) — how models use long contexts
