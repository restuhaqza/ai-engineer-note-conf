# Cooking with Agents in VS Code

| Field | Details |
| --- | --- |
| **Speaker** | Liam Hampton |
| **Affiliation** | Microsoft |
| **Type** | Talk |
| **Date** | 2026-05-21 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=dyHpnnlkTc8) |
| **Tags** | `coding-and-developer-tools` `agent-engineering` `leadership` |

## Summary

Liam Hampton, a cloud advocate at Microsoft, frames coding agents as a question of **how much involvement each task needs**. His starting point is the myth of the one-shot prompt: no single prompt builds a real application. Meanwhile agents keep multiplying across terminals, CLIs, chat windows, and editors, adding choices and cognitive load right when businesses are asking about return on investment and token spend.

His answer is to match three execution environments to three kinds of work. **Local** agents run beside you in VS Code — he uses them where he wants to stay hands-on, like writing tests. **Background** agents run through GitHub Copilot CLI and keep their changes in a separate Git worktree (an isolated working directory tied to the same repository) — he uses them for a front end, a task that becomes tedious to babysit. **Cloud** agents run on remote GitHub infrastructure — he hands them documentation.

The demonstration runs all three at once on one Python CRUD (create, read, update, delete) application with a requested front-end issue. Hampton shows the points where control returns to the developer: a background agent told to stop before opening a pull request, tests inspected as they are written, and a worktree-specific command needed to actually run the new UI. He closes on reusable guidance (instructions, custom agents, prompt files, and skills) and on MCP servers that extend what agents can reach.

## Key Takeaways

- **There is no one-shot prompt.** Expecting a single prompt to build and fix an app is the wrong mental model.
- **Three environments, three jobs:** local for hands-on work, background for tedious delegation, cloud for long handoffs.
- **"Local" describes interaction, not location.** A local agent can use a remote or locally hosted model.
- **Isolate background work with a Git worktree.** Each worktree gets its own directory and branch, so changes stay out of your checkout.
- **Delegation includes a stopping point.** Hampton tells the agent to stop before a pull request so he can test locally first.
- **Passing tests are a checkpoint, not the end.** Inspection surfaced unfriendly route errors, so tests and error handling were updated together.
- **Run from the right directory.** The front-end changes live in the worktree, so running the original checkout shows nothing new.
- **Reusable guidance has building blocks:** custom instructions, custom agents, prompt files, and agent skills — plus MCP servers for external tools and context.

## Detailed Notes

### What should an agent actually take off your hands? · 00:40
- Hampton opens with the misconception that one prompt can build an entire application or solve every issue.
- Agents now show up in terminals, CLIs, chat windows, and editors, which adds choices before it adds value.
- Businesses follow up with harder questions: where are the productivity gains, what is the ROI, and when does AI spending pay off.
- **Using an agent deliberately starts with understanding the work you are handing over.**
- Token spend is a separate worry. He mentions (without any measurement) a viral LinkedIn repo claiming that pirate-style chatbot speech reduces token usage — an example of how actively people are experimenting with cost.
- These questions apply beyond GitHub Copilot.

### Separate execution environment from developer involvement · 02:28
- **Local agent:** works alongside you inside VS Code, with close, ongoing participation. "Local" is about the workspace and interaction, not necessarily where inference runs — the model can be remote or locally hosted.
- **Background agent:** provided by GitHub Copilot CLI and reachable in VS Code, it works more independently. Its changes live in a **Git worktree** (a separate working directory attached to the same repo, here with its own branch).
- **Cloud agent:** runs on remote infrastructure using GitHub services, for work he wants to supervise least, such as documentation.
- The three are different ways to participate, not three levels of model intelligence.
- Task mapping:
  - **Local** — write tests; he wants to inspect the code and iterate.
  - **Background** — build a front end; repetitive to-and-fro gets arduous.
  - **Cloud** — prepare documentation and open-source files; a longer handoff.
- VS Code acts as one entry point for all three, including third-party agents.

### Give the front-end agent a stopping point · 05:32
- The demo uses a plain Python product store doing create, read, update, and delete. It works, but the presentation is rough, and GitHub issue 25 asks for a better front end.
- He picks a CLI background agent and asks it to summarize and plan a solution to issue 25 — planning before implementation.
- He enables **Autopilot** (a preview feature at the time) so the agent stops asking for approval on every tool call, including MCP calls. He warns this is dangerous.
- Once the plan is ready, he authorizes implementation with a boundary: stop before creating a pull request and let him test locally.
- **The useful delegation includes both the assignment and the point where control returns to the developer.**

### Add documentation and tests while the UI builds · 07:00
- With the front end running, he opens another chat and starts a cloud agent to make the repository open-source friendly: a README, contribution guidelines, and related files.
- Then he turns to tests, which the project lacks. He selects a local session, chooses Claude Opus 4.6 with medium reasoning for speed, and asks it to write unit tests.
- A custom agent holds the guidance for testing this Python application.
- He can watch the local work proceed while the cloud agent prepares docs and the background agent builds the UI.
- The tests pass, but inspection finds a separate problem: the route errors are not user-friendly. He asks the agent to update both the route error handling and its tests.
- **Passing tests end one task and start the next.** They are a checkpoint from which he spots behavior the original request did not cover.

### Check progress, then run the right working directory · 08:55
- He checks the running assignments: code and tests are updated, the background assignment reports finished, and the local tests pass.
- VS Code keeps all sessions visible together, so he can move between closely watched and delegated work without leaving the editor.
- The pull request extension shows a documentation PR from an earlier run. The cloud chat started during this walkthrough is still running — the visible PR demonstrates the review path, not completion of the current job.
- The background agent respected the pause before a PR, but its testing instructions are wrong: Hampton reminds it that the app lives in a Git worktree.
- Local validation sequence:
  1. Find the separate worktree holding the generated front end.
  2. Enter that directory and run the Python app there.
  3. Resolve a port conflict that interrupts startup.
  4. Open the Product Store UI and inspect the change.
- **Running the original checkout would not show the agent's isolated changes.** Once startup succeeds, the preview shows a styled Product Store with name and description fields, an Add product button, and an empty list — plus the new error checking.
- Recap: local tests for close involvement, a background front end for independent implementation, cloud documentation for a longer handoff. The recording shows no final merge or production deployment.

### What runs behind the cloud session · 11:47
- Cloud agents run in isolated **GitHub Actions** environments. Hampton notes cost as a common question but explains execution and capabilities rather than pricing.
- MCP servers extend the agent's tools and context: GitHub access supports repository work, and Playwright supports screenshots and automated front-end testing.
- Safeguards he names: network allowlists and restrictions that stop the agent pushing directly to main.
- Later documentation sharpens these boundaries: built-in MCP defaults give GitHub repo-scoped read-only tools and Playwright localhost access; the firewall excludes MCP servers and setup-step processes; branch restrictions govern where the agent can push, not what it can read.
- **These safeguards constrain behavior; they are not blanket safety.** Human review and testing remain necessary.

### Make the workflow reusable · 12:41
- Four building blocks supply repeatable guidance, and each has counterparts in other agent tools:
  - **Custom instructions** — define how the agent operates.
  - **Custom agents** — specialize behavior for a task, such as fixing tests.
  - **Prompt files** — make useful prompts reusable.
  - **Agent skills** — package task-specific guidance and resources.
- Hampton compares skills to a newer version of `AGENTS.md`; treat that as an analogy. Agent Skills use `SKILL.md` directories, can include scripts and resources, and load on demand. They complement standing instructions rather than simply replacing them.
- In the VS Code Copilot chat pane, the cog opens a customization interface: his custom test agent plus built-in Ask, Explore, and Plan agents, supplied skills (address a PR comment, create a pull request), instructions, prompts, hooks, and MCP servers.
- The same interface exposes Claude customization — plugins, hooks, instructions, and skills — so it is a control plane for agent behavior, not just a chat launcher.

### Connect reusable guidance and external tools · 14:55
- **Awesome Copilot** is an open-source starting point for customizations. It targets Copilot, but Hampton encourages adapting its material for other AI tools.
- He also describes an MCP server for using the project from a workflow.
- **MCP** (Model Context Protocol) connects models and agent workflows to additional tools and services.
- Authenticated examples: Azure, GCP, and AWS resources. Unauthenticated examples: tools and documentation servers such as Playwright and Microsoft Learn. Authentication depends on the service being exposed.
- Closing picture: VS Code as a single entry point — first- and third-party plugins, chat customization, full MCP specification support, and connections to GitHub Copilot CLI sessions.
- He ends by inviting the audience to share their own agent workflows; the central design choice throughout is how much involvement each assignment needs.

## Resources

- [Awesome GitHub Copilot](https://github.com/github/awesome-copilot)
- [Git worktree manual](https://git-scm.com/docs/git-worktree)
- [Agent Skills in VS Code](https://code.visualstudio.com/docs/agent-customization/agent-skills)
- [Custom-agent configuration and built-in MCP tools](https://docs.github.com/en/copilot/reference/custom-agents-configuration)
- [Copilot cloud-agent firewall and limitations](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/customize-the-firewall)