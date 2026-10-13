# Remobi: take your terminal workflow onto your phone

| Field | Details |
| --- | --- |
| **Speaker** | Connor Adams |
| **Affiliation** | Independent |
| **Type** | Talk |
| **Date** | 2026-07-12 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=5192csoTkVo) |
| **Tags** | `infrastructure-and-deployment` `enterprise` `coding-and-developer-tools` `developer-workflows-and-testing` |

## Summary

Connor Adams presents **Remobi**, an open-source mobile interface for checking on and steering coding agents from a phone. The talk starts from a familiar itch: you are out with friends or family, but your agents are still running, and you want to know if they need help. Adams is torn about whether making that easier is a good idea, but he builds it anyway.

His core requirement is that mobile access should not force a new workflow. He already runs coding agents inside **tmux** (a terminal workspace manager that splits a screen into panes and windows) on a remote machine he reaches over SSH. Remobi is designed to open that exact workspace on a phone rather than replacing it.

Remobi is a **progressive web app** (a website that behaves like an installed mobile app), so it runs on both iOS and Android, backed by a small server on his machine. Custom tmux shortcuts become tap controls for agent actions, Git review, and diff inspection. Adams is candid that it looks rough and is only minimally functional, not polished.

The last part covers access and safety. Remobi talks to the machine over the network, and Adams argues the security of that link is entirely up to how you configure it. He uses **Tailscale** (a private network tool) and warns that exposing the service directly to the public internet is a serious mistake.

## Key Takeaways

- **Remobi keeps your existing terminal setup.** It targets people who already use tmux, rather than asking them to adopt a new app's way of working.
- **The problem is the phone check-in habit.** Agents keep running while you are away, and the urge to see if they need steering is the motivation.
- **Other options each add friction.** Agent-specific apps, manual session handoffs, and general SSH terminals all fell short for Adams.
- **Custom controls carry over.** Existing tmux key bindings are turned into touchscreen buttons for agent modes, Git, and diffs.
- **The UI is deliberately modest.** Adams calls it minimally functional, not designed for polish; pane gestures are the practical part.
- **Installation is guided and optional.** A shell script walks you through setup and can install a helper skill for tmux; you can also install the skill and npm package yourself.
- **Remote access is your responsibility.** How you expose the server decides the security; Tailscale is the default, with Cloudflare Tunnels and ngrok as alternatives.
- **Never expose it publicly.** Putting the interface straight on the internet means you have effectively taken over your own machine.

## Detailed Notes

### Do the agents need you? · 00:28
- Adams asks the audience who checks on their agents from a phone, then who wants to but does not.
- He describes the pull honestly: you are out with friends or family, and you still feel pulled to see whether the agents are progressing or need steering.
- He jokes about "AI psychosis" driving the urge to build yet another app, but argues the tool he wanted did not exist.
- His objections: a popular mobile client (Happy) is tied to one agent and uses a relay server he does not personally trust; built-in handoff features need a manual transfer and do not cover other agents; and a plain phone terminal means managing SSH keys and touch controls.
- The deciding requirement: he already has a tmux-based workflow, so mobile access should preserve it.

### Build the workspace in the terminal · 02:16
- Adams moved from VS Code to the terminal, partly for portability. His setup lives on a remote dev machine he can SSH into, so the same tools follow him.
- He shows four coding agents sharing one terminal in separate panes. tmux acts like a window manager for the terminal: panes divide a window, and windows give separate views with tabs along the bottom.
- The status bar is customizable too — he uses it to show CPU usage and other machine info.
- Custom commands extend this: he opens a launcher, chooses how many agents to start (offering sixteen, picking four), and the workspace is created. This demonstrates configurability, not a speed benchmark.
- He says he let AI generate much of the configuration because he did not know how to implement it: describe the interaction you want, then have the agent write the tmux config.
- He acknowledges apps like Conductor but prefers to own his workflow while still experimenting.

### Keep review and maintenance close to the agents · 04:05
- After Claude Code finishes a change, the next job is to inspect it. Adams presses a shortcut to open **Lazygit** (a terminal UI for Git) in a window and scrolls through changes.
- He also opens **Critique**, another scrollable diff viewer, to make the point that code review happens before committing. The agent's output has somewhere to go before it becomes a commit.
- Custom tools also handle small local chores. When a dev server will not start because a port is taken, his utility finds the agent browser process holding it and lets him kill it.
- Frequently needed operations become terminal shortcuts alongside agent management and Git review.
- He punctures the productivity narrative with a joke: all this customizing can still leave you shipping nothing users actually use.

### Open the same workspace on a phone · 05:19
- Adams switches to his phone. Remobi is a progressive web app, so it works on iOS and Android, backed by a server running on his machine.
- He mentions a Pi version, then picks the machine from the desktop demo. The phone opens the workspace exactly where he left it.
- One rough edge appears right away: a scrollback shrinking behavior does not work as expected, which he attributes to switching views.
- The touch controls matter most: a Shift+Tab button switches the agent into plan mode, a button opens the Git interface, and another opens Critique with gesture-based scrolling.
- These controls let him trigger terminal actions without typing every key combination on the phone keyboard.
- He is blunt that it will not win design awards. The gestures are the useful part: double-click to zoom into a pane, scroll within a pane, then zoom back out — keeping the multi-pane layout while making one pane readable on a small screen.

### Adapt the touchscreen controls to your setup · 06:50
- Remobi is open source. Adams introduces the install script with sarcasm about the wisdom of pasting arbitrary shell commands into a terminal.
- The guided setup walks you through installation and installs a skill that helps configure tmux if you do not have it yet.
- If you already use tmux, the setup converts your key bindings into controls for Remobi's touchscreen interface. So it accommodates both a fresh workspace and an established, customized one.
- The guided script is optional; you can instead install the skill and the npm package directly.
- He closes by asking for GitHub stars, even from people who will not use the app.

### Session access and the security boundary · 07:54
- An audience question asks whether remote control is itself a tmux feature. Adams separates the roles: tmux provides the panes and workspace, while Remobi calls tmux and drops you straight into your session.
- The demonstrated setup hides that session-entry step rather than making the phone user manage it each time.
- A second question covers how the phone talks to the machine. Remobi is a website reached over the network. Adams uses Tailscale, and names Cloudflare Tunnels and ngrok as alternatives.
- His key qualification: security depends entirely on how that access is configured. Giving a route to a terminal does not by itself decide who is allowed to use it.
- He warns that placing the service directly on the public internet means, in his words, that you have taken over your own computer.
- When an audience member asks whether Tailscale is the default (and says they would otherwise be worried), Adams confirms that it is.

## Resources

- [Remobi](https://github.com/connorads/remobi#readme) — source, install instructions, and configuration for the mobile terminal PWA.
- [tmux documentation](https://github.com/tmux/tmux/wiki) — guides for persistent sessions and terminal multiplexing.
- [Lazygit](https://github.com/jesseduffield/lazygit) — terminal UI for inspecting and managing Git changes.
- [Critique](https://github.com/remorses/critique) — terminal diff viewer for reviewing changes.
- [Happy](https://github.com/slopus/happy) — mobile and web client for Claude Code and Codex.
- [Tailscale Serve](https://tailscale.com/docs/features/tailscale-serve) — share a local service within a private network over HTTPS.
- [Cloudflare Access](https://developers.cloudflare.com/cloudflare-one/access-controls/applications/http-apps/self-hosted-public-app/) — add authentication and access rules to a self-hosted app.
