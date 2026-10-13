# Fear of Missing Agent Time

| Field | Details |
| --- | --- |
| **Speaker** | Michael Richman |
| **Affiliation** | Bitly |
| **Type** | Talk |
| **Date** | 2026-05-24 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=W-SX_srBa3Y) |
| **Tags** | `apis-mcp-and-protocols` `coding-and-developer-tools` `observability-and-reliability` `architecture` |

## Summary

Michael Richman, an engineering leader at Bitly, names a common frustration: FOMAT, or fear of missing agent time. You step away from a coding agent and it keeps working — until it needs a decision, a clarification, or permission. That interruption is unpredictable, so you cannot schedule when to come back. Richman notes coding tasks at the time ran five to 45 minutes, but as they stretch to hours or days, "check back in a bit" stops working.

His answer is Cmd+Ctrl, a system he built to hold all of his coding agents in one place, reachable from a phone, the web, or a watch. It lets him monitor sessions, reply to them, and start new ones away from his development machine. The demos show a session starting in the terminal and being continued from a phone, and the reverse: a session started on the phone, then resumed in the CLI.

The architecture separates where agents run from how you supervise them. A daemon beside each agent platform reports lifecycle changes — including blocked states — to a control plane; the UI talks to that API and notifies the user. Richman argues the deeper shift is in how development feels: a new kind of "flow" is agent choreography, moving between parallel agents to unblock one and redirect another.

Why it matters: it treats agent supervision as a real interface problem, not just a matter of more autonomy. It also warns that always-available agents make time away more valuable, not less necessary.

## Key Takeaways

- **FOMAT = fear of missing agent time.** An agent may stop two minutes after you leave and wait, blocked, the whole time you are gone.
- **You cannot schedule around handoffs.** Agents need input unpredictably, so the system has to reach you wherever you are.
- **One place for all sessions.** Cmd+Ctrl gathers Claude Code, Codex, Gemini, Cursor, OpenCode, and more into a single interface.
- **Changing devices preserves the conversation.** A session started in the terminal appears on the phone and vice versa, because it is the same session.
- **Group sessions by attention, not just time.** Subscribed (push alerts), on my radar (keep visible), recent (last 24 hours), and the rest.
- **A daemon-and-control-plane split** lets the control plane aggregate agents across machines and frameworks.
- **Flow becomes agent choreography.** The new satisfaction is coordinating parallel agents and unblocking them elegantly.
- **Time away matters more, not less.** Managing many sessions is exhausting, and breaks are where ideas arrive — so the system should let you act on those ideas.

## Detailed Notes

### An agent working—or waiting? · 00:16
- Richman leads several engineering teams at Bitly, the link shortener.
- The opening image: you are on a walk with an idea to hand to an agent, but you have to wait until you are back at the dev machine.
- Or you leave an agent running, come back, and find it stopped after two minutes to ask a question and has been blocked since.
- Agents feel low-touch and high-autonomy, but in practice the work is back-and-forth and babysitting.
- At the time of the talk a coding task took roughly five to 45 minutes, with relatively little agent idle time — a window that is only getting longer.
- The requirement: know when an agent needs you and when it is done, wherever and whenever that happens.

### One place for scattered sessions · 02:19
- Richman is a hands-on leader who still writes code, co-leads Bitly's AI coding tools strategy, and says he co-wrote the Bitly MCP server.
- He built Command and Control (Cmd+Ctrl) because he needed to work with coding agents outside the terminal or IDE.
- The space was already moving: Anthropic released remote control and a "teleportation" mechanism, and Cursor shipped a solution about two days earlier.
- His requirement is broader than reaching one tool remotely: all coding agents in one interface, from mobile or anywhere.
- His desktop shows the problem — multiple terminal windows with multiple tabs, Claude Code, Codex, and Gemini sessions beside the Cursor IDE, each at a different stage.
- He can track two or three sessions; at four or five he loses track of what another session is doing and which one needs attention.
- Three questions the missing interface must answer: which agent is waiting for a decision, which needs a mid-task correction, and how to start a session while away.
- He credits AgentCraft, demoed earlier at the conference, as a gaming-style take on the same need.

### Continue a terminal conversation from a phone · 04:49
- Cmd+Ctrl is on iPhone, Android, web, and watch, and can monitor, interact with, and launch sessions.
- The first demo uses a deliberately ordinary Claude Code prompt, because the point is conversation continuity, not code generation.
- Sessions are grouped by notification subscriptions, watched sessions, and recent activity; the new session appears as 27 seconds old.
- Opening it shows the same agent response as the terminal, because it is the same session.
- The notification test:
  1. Subscribe to the session's notifications.
  2. Send "Sleep for one second and say hello."
  3. Leave the app while the agent works.
  4. Open the push notification when it arrives and read the hello.
  5. Return to the terminal and resume the session.
- The resumed terminal shows both the phone-sent message and the agent's reply, so switching devices keeps the conversation intact.

### Start remotely, resume locally · 07:19
- The second demo reverses direction: create the session in Cmd+Ctrl, then pick it up in the terminal.
- The new-session screen offers the agents configured on his machine: Claude Code, Codex, GitHub Copilot, and Cursor.
- He selects Claude Code, switches to a testing directory, and asks what time it is — a trivial prompt that isolates the handoff.
- He resumes the mobile-created session in the CLI, which shows the question, a Bash `date` command, and the returned time and date.
- The point: the terminal already holds the conversation begun elsewhere; there is no need to restate the task.
- From the terminal he then asks for tomorrow's date, and Cmd+Ctrl shows the same conversation continuing.
- He often starts his day by sending prompts from bed, launching several sessions, then continuing them in the CLI or on his phone.

### Decide what deserves attention · 09:24
- Remote access is only half the problem; with many sessions in one place, the interface must show which ones matter now.
- **Subscribed:** receive push notifications for the session.
- **On my radar:** keep a session visible without a notification for every message.
- **Recent:** sessions from the last 24 hours.
- **Remaining sessions:** access to the rest of the history.
- The displayed history holds thousands of sessions — not thousands of running agents — grouped so active attention is separate from historical access.
- The **overview dashboard** adds a stand-up-style summary of recent sessions, using each session's last several messages to describe its activity.
- Together the groups and summaries answer two needs: where to look, and enough recent context to re-enter the work.

### Daemons feed a shared control plane · 11:32
- The architecture separates where agents execute from the interface used to supervise them.
- A Cmd+Ctrl **daemon** runs beside each agent platform — Claude Code, Cursor, Codex, Gemini, and OpenCode are named.
- The daemon watches the agent lifecycle and reports changes (such as a blocked state or a request for help) up to a **control plane**.
- The UI talks to that API layer and notifies the user, turning an event in a development environment into something you can respond to elsewhere.
- The control plane aggregates agents across platforms and machines: execution can stay on a dev machine, run in a cloud VM, or span both.
- His concrete example: Claude Code on his Mac and Codex CLI in a cloud VM, with both sets of sessions in one UI.
- The system is designed to be coding-tool agnostic, and the daemon layer is open source, so other agent frameworks can integrate and expose their sessions through the same UI.
- The guiding idea: whatever your agent, you can reach it and it can reach you.

### Flow becomes agent choreography · 13:06
- Richman pushes back on the idea that coding agents are fully autonomous, especially if you can only reach them inside their native environments.
- Traditional flow meant deep focus on one problem — you and your code.
- The new flow is **agent choreography**: several agents working in parallel while you move between them, unblocking one and redirecting another.
- The satisfaction comes from the elegance of that coordination and the results on the other side.
- There is a human limit: always-available agents make time away more valuable, because managing multiple sessions carries a high cognitive load and is exhausting.
- He connects this to Matt Pocock's remarks the previous day and a Simon Willison interview on Lenny's Podcast the previous week.
- Breaks are also where good ideas arrive, and being able to send one to an agent closes the loop with the opening walk.
- He closes by inviting people to try Cmd+Ctrl and share their workflows and pain points, with product and LinkedIn details on the closing slide.

## Resources

- [Cmd+Ctrl](https://cmd-ctrl.ai/)
- [Bitly MCP Server quickstart](https://dev.bitly.com/bitly-mcp/overview/quickstart/)
- [Claude Code Remote Control](https://code.claude.com/docs/en/remote-control)
- [Claude Code cloud sessions and teleport](https://code.claude.com/docs/en/claude-code-on-the-web)
- [AgentCraft setup](https://www.getagentcraft.com/docs/getting-started)
- [Simon Willison on Lenny's Podcast (notes)](https://simonwillison.net/2026/Apr/2/lennys-podcast/)
