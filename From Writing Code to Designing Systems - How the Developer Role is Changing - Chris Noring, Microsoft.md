# From Writing Code to Designing Systems: How the Developer Role is Changing

| Field | Details |
| --- | --- |
| **Speaker** | Chris Noring |
| **Affiliation** | Microsoft |
| **Type** | Talk |
| **Date** | 2026-07-11 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=GdvKNwMcfd0) |
| **Tags** | `rag-context-and-search` `coding-and-developer-tools` `architecture` `developer-workflows-and-testing` |

## Summary

Chris Noring, who works on AI engineering at Microsoft, argues that AI-assisted development shifts engineers from writing every line of code toward designing systems, planning work, and coordinating coding agents. Two show-of-hands polls set the scene: few in the room still code the old way, and Claude and GitHub Copilot are used in roughly equal measure.

In the old model, progress depended on one developer's time. Now implementation can be delegated, so attention moves to where it matters most. But more output is only useful with guardrails; otherwise it becomes "AI slop." His proposal is a CLI-first workflow (CLI means command-line interface): start and delegate in the terminal, keep the editor for fine adjustments, and scale the work through GitHub.

He layers three guardrails. `AGENTS.md` captures repository intent and constraints. **Skills** package a repeatable recipe in a folder with a `SKILL.md`. **Custom agents** add a role, reasoning, and a restricted tools list. Work is then delegated remotely with `/delegate` or by assigning an issue to an agent, returning as a draft pull request a human reviews. His closing claim is conditional: 20× more code could be 20× more slop without the right guardrails.

## Key Takeaways

- **The bottleneck moves.** Once implementation can be delegated, a developer's attention is the scarce resource.
- **Pick a surface per job.** CLI to start and delegate, editor to refine, GitHub UI to scale.
- **Starting in the terminal is fine.** It need not mean becoming a shell expert — just describing work before opening an editor.
- **Guardrail 1 — `AGENTS.md`.** Repository intent, architecture, and constraints. It is project context, not a security boundary.
- **Guardrail 2 — Skills.** A folder with a `SKILL.md` (name and description front matter) supplies a repeatable recipe.
- **Guardrail 3 — Custom agents.** A role with reasoning and a restricted `tools` list; they orchestrate where a skill only prescribes.
- **Delegate, then review.** `/delegate` or an assigned issue returns a draft PR (pull request) for a human to assess.
- **20× is conditional.** Without guardrails, 20× more code can be 20× more slop.

## Detailed Notes

### When every feature depends on your own time · 00:15
- Two polls: few are coding exactly as before, and Claude versus Copilot usage is roughly split, perhaps leaning Claude.
- In the old model the developer wrote all the code, so progress was linear and limited by one person's time.
- The bottleneck moves when implementation can be delegated — developers still code, but become selective about what they write.
- A quality trap: early AI-generated code could add more work than it saved.
- Producing more code only helps when the surrounding workflow constrains what gets produced.

### Start in the CLI, continue in the editor · 02:44
- He presents his workflow as a proposal, not a requirement; adopting some AI tool is the part he treats as unavoidable.
- Available surfaces now extend beyond the IDE (integrated development environment): Claude Code and Claude Desktop, Copilot's CLI, editor integrations, and the GitHub web UI.
- His sequence starts in the terminal — not requiring shell expertise, just a place to describe work before opening an editor.
- The editor comes second: after roughly 19.5 of his 20 industry years there, it is still home.
- The CLI and GitHub UI become the points for delegation and scaling.
- A first useful output can be a draft app or MVP (minimum viable product). He notes businesses seeing working demos instead of PowerPoint decks.
- The CLI also handles ordinary repository work — issues, PRs, and status checks. He closed fifteen issues without opening VS Code.

### The editor becomes a control board · 05:54
- Instead of writing every Java, JavaScript, or Python change, he runs several terminals on different requests: build an app, fix an issue, add a feature, build another app.
- The meaningful change is **concurrent implementation**; letting it run while he drinks coffee is the garnish.
- Moving into the editor then means configuring guardrails, inspecting work, and making selected changes.
- He imagines 15–20 agents doing implementation, so continuous manual authoring shrinks as a share of the day.
- Integrations can surface activity from terminals, the editor, and GitHub repositories, so the editor becomes a **control board** for work happening in several places.
- He still values the editor's familiarity and sense of control — but without rules, that concurrency multiplies unwanted behaviour.

### Give the repository explicit rules · 08:04
- `AGENTS.md` is the bare-minimum guardrail: high-level guidance describing repository intent, architecture, responsibilities, and permitted changes.
- His example is a finance app using React and Tailwind, with an explicit constraint against changing the architecture unless asked.
- These instructions give the agent project context; they are **not an enforced security boundary**.
- In the demo he opens a finance tracker built with AI help, shows its longer `AGENTS.md`, and asks Copilot what the file does.
- He also suggests copying guidance from another project into a fresh repo and asking the agent to adapt the codebase to it.

### Turn recurring tasks into skills · 09:36
- When an agent performs a recurring task almost right, or veers off, a **skill** supplies a repeatable recipe: what to do, which tools to use, and in what order.
- Noring calls it a strict contract — a discipline against improvisation. Instructions alone do not guarantee deterministic execution.
- A skill is invoked by an agent and packaged as a self-contained folder.
- Each skill has its own directory with a `SKILL.md`: YAML front matter containing a `name` and `description`, followed by Markdown instructions.
- He names `.claude/skills`; a slide also lists `.github/skills` and `.agents/skills`. Shared conventions matter because developers do not always choose their organisation's vendor.
- The description helps the agent recognise when the recipe applies; the body supplies the procedure to follow.
- In the demo he asks Copilot to improve an existing skill's instructions, and shows invoking skills via `/skills` or by typing a prompt resembling the skill's name.

### Use custom agents for roles and orchestration · 12:50
- When a task needs planning, reasoning, or orchestration, he moves up to a **custom agent**.
- A custom agent has a distinct role (security specialist, back-end or front-end developer) and can use multiple skills and MCP servers (MCP is a standard way for agents to call tools).
- Skills versus agents: a skill is a task recipe invoked by an agent; an agent is a broader role that plans and coordinates.
- Definitions also use front matter plus instructions; he places them in `.github/agents` with a `.agent` filename. The documented VS Code format today is `<name>.agent.md`.
- Compared with a skill, the agent definition adds argument hints and a `tools` field.
- Tool access limits what the agent can do rather than merely describing it: his researcher can read, search the web, and keep to-do lists, with file creation and editing withheld.
- He selects the researcher in Copilot chat and asks about Microsoft Agent Framework — demonstrating a role with reasoning and tools, not one fixed recipe.

### Delegate from the CLI or an issue · 16:16
- The workflow returns to the terminals and scales outward: he imagines six sessions on features, MCP work, and fixes.
- He calls agents "toddlers" — sometimes brilliant, sometimes baffling — which is why the harness comes before broad delegation.
- His 10×, 20×, and 100× references express ambition, not measured productivity.
- CLI route: `/delegate` sends the session to GitHub. His example asks for a finance app in HTML, CSS, and JavaScript (separate from the earlier React and Tailwind example).
- Handoff: start from a real GitHub repository (delegation complains otherwise), send it with `/delegate`, let the remote job run and create a draft PR, then review before shipping.
- GitHub UI route: create an issue (his example adds dark mode), click **Assign agent**, and optionally add further instructions.
- A Copilot indicator and a clickable work-in-progress item show status; the result is again a draft PR for the human.
- This is remote delegation, distinct from local autopilot. He describes agents as sandboxed but does not promise universal containment; the concrete review boundary is the draft PR.
- He mentions Playwright MCP and GitHub's MCP server, and associates the GitHub-side flow with GitHub Enterprise — that remark is not a general plan requirement.

### Keep ownership of what ships · 20:45
- He compares the change to moving from axes to chainsaws: developers keep adopting tools that raise how much they can do.
- The human role stays consequential because engineers build the system that directs the agents.
- They set the harness, place approval gates, add more where needed, review proposed changes, and can remain the people who merge.
- Division of attention: CLI to start apps and concurrent tasks, editor for fine adjustments when its interface is better, CLI or GitHub UI to delegate.
- His 20× closing claim is conditional rhetoric: 20× more code could also mean 20× more slop without the right guardrails.
- `AGENTS.md`, skills, and custom agents are the mechanisms he proposes for steering that extra output.
- The intended outcome is to scale engineers, not replace them — deciding what to build and whether it belongs still needs an engineering mind.

## Resources

- [AGENTS.md](https://agents.md/) — repository instruction format for project context, conventions, and constraints (mentioned).
- Agent Skills specification — defines skill folders and `SKILL.md` metadata (related).
- Use Agent Skills in VS Code — current guidance for creating and using reusable skills with Copilot (update).
- [Custom agents in VS Code](https://code.visualstudio.com/docs/agent-customization/custom-agents) — the current `.agent.md` format, locations, and tool restrictions (update).
- [Delegating tasks to Copilot](https://docs.github.com/en/copilot/how-tos/copilot-cli/use-copilot-cli/delegate-tasks-to-cca) — sending CLI tasks to GitHub and following the result (update).
- [Playwright MCP](https://github.com/microsoft/playwright-mcp) — Microsoft's Playwright MCP server (mentioned).
- [GitHub MCP Server](https://github.com/github/github-mcp-server) — repository, issue, and PR tools (mentioned).
