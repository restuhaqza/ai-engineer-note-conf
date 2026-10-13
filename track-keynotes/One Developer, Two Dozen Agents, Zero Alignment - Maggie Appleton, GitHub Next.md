# One Developer, Two Dozen Agents, Zero Alignment

| Field | Details |
| --- | --- |
| **Speaker** | Maggie Appleton |
| **Affiliation** | GitHub Next |
| **Type** | Track keynote |
| **Date** | 2026-04-26 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=ClWD8OEYgp8) |
| **Tags** | `coding-and-developer-tools` `safety-and-governance` `developer-workflows-and-testing` `rag-context-and-search` |

## Summary

Maggie Appleton, a designer and staff research engineer at GitHub Next (GitHub's
experimental research team), starts from a picture many people find exciting: one
developer running two dozen coding agents in parallel and doing the work of a whole
team. She calls this the "one man, two dozen Claudes" theory of the future, and she
argues it misses something important. Software is a team sport. Scaling one
person's output does not create agreement about *what* to build.

Her core point is that implementation is becoming fast and cheap, so the real
bottleneck moves to alignment — deciding together whether something should be built
at all. But the tools teams use to coordinate (GitHub, Slack, Jira, Linear) were
built for a slower rhythm where there was time to discuss, catch mistakes, and
course-correct during implementation. Agents compress that window. Issues turn
into pull requests in minutes, plans stay local and unshared, and all the
alignment pressure lands on the pull request at the very end.

The result is wasted work and **coordination debt** (cleanup from agents editing
the same files, messy merge conflicts, and people duplicating each other). Her
answer is a shared workspace. She demos **Ace**, the Agent Collaboration
Environment: sessions that are both a multiplayer chat and a sandboxed cloud
computer (a microVM) on its own Git branch, where teammates and agents work
together in real time.

The talk closes with the bigger opportunity: reclaim the time implementation used
to consume and spend it on research, design, and architecture. In a world of fast,
cheap software, she argues, craftsmanship and quality become the differentiator —
but craft still costs time, and buying it means agreeing to do fewer things better.

## Key Takeaways

- **More agents don't fix alignment.** Single-player tools scale one person's output, not a team's agreement on what to build.
- **Implementation is becoming a solved problem.** The hard question shifts from "how do we build it?" to "should we build it?"
- **The implementation window is collapsing.** When coding is cheap, teams skip planning and the pull request becomes the first place decisions are seen.
- **Two failure modes:** wasted work (building the wrong thing) and coordination debt (merge conflicts, duplicate work, giant PR stacks).
- **Most needed context isn't in the code.** Budget, decision authority, product vision, user research, and org history live in people's heads.
- **Ace pairs chat with a computer.** Each session is a multiplayer conversation plus a sandboxed cloud microVM on its own branch.
- **Cloud sessions change collaboration.** Teammates join in one click, see the full prompt history, and pick up work without stashing local changes.
- **Reclaim time for craft.** Faster production only helps if teams use the freed capacity for deeper thinking and fewer, better features.

## Detailed Notes

### One developer, two dozen agents needs a shared direction · 00:25
- The opening image: a wall of terminal-based coding agents running in parallel on a single machine.
- The promise is that one person with a fleet of agents can do a whole team's implementation work.
- Appleton's objection: that assumes software is made by one person. These are single-player interfaces.
- She compares the belief to the "nine women make a baby in one month" fallacy — some things don't scale with more workers.
- Implementation is getting faster, cheaper, and higher quality; the bottleneck becomes **agreement about what to build**.
- When production is cheap, **opportunity cost** is the real cost: choosing one feature spends the effort that another would have used.

### The implementation window collapses · 02:42
- Coordination tools (GitHub, Slack, Jira, Linear) were built for an older rhythm; Appleton notes even people inside GitHub are exploring what comes next.
- In the old planning–building–review flow, implementation was slow enough to allow Slack chats, meeting comments, and draft PR reviews along the way.
- Agent implementation compresses that interval, and cheap code tempts teams to skip planning — so early alignment touch points disappear.
- Generated code actually takes *longer* to review, moving coordination to the wrong side of implementation.
- The time from logging an issue to an agent opening a PR is now a couple of minutes.
- Local plan modes make it worse: a plan can be produced and never shared with the team.
- The pull request inherits the burden of explaining intent, supplying context, and negotiating scope — after the code already exists.

### Coordination debt starts before the PR · 05:02
- **Wasted work:** features nobody asked for, changes that don't solve real problems, and finished work tossed after late feedback.
- **Coordination debt:** fuzzy merge conflicts from agents touching the same files, and developers independently doing the same task.
- Local success can still create costs elsewhere in the team, like giant review queues no one has context for.
- The fix is to align *before* agents start and keep aligning *during* implementation — planning and building become a cycle, not phases.
- Much of the needed context lives outside the repo: business constraints and money, decision authority and politics, product vision and user research, and organizational history.
- Code can't tell an agent these things, so teams need a low-friction way to add that context early.

### Ace: a session is a conversation and a computer · 06:40
- **Ace** stands for Agent Collaboration Environment — a research prototype from GitHub Next, unfinished and heading to technical preview.
- The interface looks familiar: a session list on the left and a multiplayer chat with both teammates and coding agents.
- Each session is backed by a **microVM** — a sandboxed cloud computer with its own Git branch.
- Sessions isolate parallel tasks; participants within a session share one working environment.
- Asking a colleague for help gets cheaper: they enter the session without stashing changes, pulling branches, or wrestling with worktrees.
- They also see the full prompting history, so the shared artifact is both the code and the conversation that produced it.

### Run the project and collaborate live · 08:07
- Standard terminal commands work: Appleton runs `bun install` and `bun dev`.
- Opening the port shows a live browser preview beside the chat; the demo app is a calmer Hacker News showing top stories from the last three months.
- She asks the agent to switch the theme to purple, and the preview updates because it is running the changed code; the agent also makes a commit with a message.
- With teammates present, she requests more themes and selects **Opus 4.6**.
- A summary block in the upper-right tracks recent changes from any participant, helping someone switching between parallel sessions stay oriented.

### Team conversation becomes agent context · 09:21
- Teammates can start the dev server, view the same preview, and run commands, seeing shared outputs — "works on my machine" stops being an issue.
- Collaborators add screenshots, questions, and alternative feature ideas; a teammate (Nate) prompts the agent directly even though Appleton started the session.
- The agent receives the **whole conversation** as input, so the team can discuss a change and then invoke `@Ace` to act on that shared context.
- The Slack-like interface lets designers, PMs, and support staff join while a feature is built.
- Appleton's reason for a dedicated tool: software-development primitives — terminals, diffs, the running project — are outside Slack's core business.

### Plan, pull request, and multiplayer editing · 11:02
- Once the team likes the changes, they create a PR directly inside Ace, inspect its preview, and follow the link to GitHub.
- The PR description links back to the Ace session, preserving a route to the conversation; only part of a team needs to adopt Ace.
- Direct editing is still available: Appleton opens the project in **VS Code** with real-time multiplayer editing because everyone is on one cloud computer.
- Closing her laptop doesn't end the session — teammates can keep editing and prompting the agent.
- For a bigger change (variable timeframes), she asks Ace to draft a plan, opens it as a shared document, and teammates edit it together with visible cursors.
- The decision (a dropdown instead of a segmented control) is folded into requirements before she asks `@Ace` to implement it — the intervention happens while it is still a plan.

### Recovering context, and spending time on craft · 13:07
- The dashboard addresses a Monday-morning problem: remembering what you left unfinished. Ace surfaces React hooks from an ongoing refactor.
- **Pick back up** reopens the session tied to an unmerged PR; recently completed PRs and issues show finished work.
- **Team pulse** summarizes coworkers' activity (one shipping a lobby channel, another fixing access-token issues), alongside a raw feed of issues and PRs.
- The volume matters: coworkers now ship five features a day instead of half of one — an illustration, not a measured result.
- The bigger hope is a **social information fabric**: agents that surface relevant decisions or pull the original author into a discussion about extending their feature.
- The closing argument is to **reclaim time**. Implementation used to consume the effort teams wanted for research, design, and architecture.
- In a world of fast, cheap software, quality becomes the differentiator — but craft still costs time, so buying it means doing fewer things better.
- The goal: agents that help teams think rigorously together, align faster, and build a few exceptional things rather than a thousand crappy ones.
- Ace was expected to reach early access within a couple of months, via a signup form.

## Resources

- [One Developer, Two Dozen Agents, Zero Alignment](https://maggieappleton.com/zero-alignment) — Appleton's companion page with notes and slides.
- [GitHub Next](https://githubnext.com/) — research projects from GitHub's experimental tools team.
- "Gas Town's Agent Patterns, Design Bottlenecks, and Vibecoding at Scale" — related article by Appleton.
- "Collaborative Workspaces" — a 2022 GitHub Next concept bringing code, design, and discussion together.
