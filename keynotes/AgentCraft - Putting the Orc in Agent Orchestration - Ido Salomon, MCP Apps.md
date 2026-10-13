# AgentCraft: Putting the Orc in Agent Orchestration

| Field | Details |
| --- | --- |
| **Speaker** | Ido Salomon |
| **Affiliation** | MCP Apps |
| **Type** | Keynote |
| **Date** | 2026-04-25 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=kR64LOqBBCU) |
| **Tags** | `developer-workflows-and-testing` `architecture` `coding-and-developer-tools` `agent-engineering` `vision-and-video` |

## Summary

Ido Salomon — creator of AgentCraft, creator of MCP-UI, and creator and co-maintainer of MCP Apps —
opens with a simple observation: if one coding agent is useful, why not run 10, 20, or 100? Spinning them up is
easy. Steering them is not. The human becomes the orchestration bottleneck, and managing dozens of
unpredictable workers is a very different job from writing software.

His inspiration comes from real-time strategy (RTS) games, where players already manage many units,
split attention, and react to changing conditions. AgentCraft is a free, still-experimental
orchestrator that transfers those interaction patterns to agent work.

The walkthrough shows what that means in practice: an agent unit is a real live coding session; the
map is a projection of the filesystem, so you can watch which file each agent touches; a change list
records who changed what and when; and RTS-style muscle memory lets you cycle through agents that
need approval. Then quests and campaigns push task discovery and supervision into the agents
themselves, review bundles make the resulting pull requests easier to assess, and shared workspaces
let teammates and their agents see each other's work.

Salomon's closing point: the skills needed to direct agents are not new. Games are one place people
already practiced distributing attention across many units. AgentCraft's direction is to raise the
ceiling of human–agent collaboration through visibility, agent autonomy, and collaboration between
people and agents.

## Key Takeaways

- **Spawning agents is easy; steering them is the hard part.** The human, not the model, is the
  bottleneck.
- **A unit is a live session.** AgentCraft detects or spawns real coding sessions (Cursor, Claude
  Code, Codex, OpenClaw) and lets you prompt them with text, voice, or images.
- **The map is the filesystem.** Directories sit on the map and files become rooms, so an agent's
  location shows which file it is working on.
- **Lineage answers "what changed."** The change list records which agent made each change and when,
  opening the door to collision detection and a heat map.
- **Faster switching is not less attention.** Cycling between blocked agents let him run more in
  parallel, but only briefly; the mental load stays.
- **Quests and campaigns shift supervision.** Agents propose missions, and a campaign hands ongoing
  supervision to a campaign orchestrator inside a container.
- **Review bundles compress the review queue.** Changes come with rationale, tasks, screenshots, and
  videos so you can judge results instead of reconstructing activity.
- **Workspaces make agents shared.** Teammates see each other's agents, can prompt them directly, and
  coordinate in a shared human–agent chat.

## Detailed Notes

### Spawning agents is the easy part · 00:30
- If one agent is amazing, the obvious move is to launch 10, 20, or 100 and be 100 times more
  amazing. A screen full of agents looks glorious, but it doesn't actually work.
- The reason is not the spinning up; it's us. The human becomes the bottleneck in orchestrating all
  those agents.
- Managing dozens of unpredictable workers is not the job most engineers signed up for, so new skills
  are needed.
- Luckily those skills are not brand new — they've been hiding in unexpected places. Anyone who has
  played **RTS** games (real-time strategy, like StarCraft-style titles) recognizes the feeling of
  managing dozens of units at once.
- AgentCraft is the attempt to transfer those gaming interaction patterns into productive work.

### A game unit is a live coding session · 02:01
- On the AgentCraft map, an agent is not a metaphor: it is a real running coding session, such as
  **Cursor**, **Claude Code**, **Codex**, or **OpenClaw**.
- AgentCraft can detect sessions already running on the machine and visualize them, or spawn a
  session directly from the interface — with a game-like "at your command" acknowledgment.
- You prompt a unit much as you would from a CLI. Inputs include voice, text, and images; in the demo
  he asks an agent to develop a feature and it starts working.
- The game presentation changes how the session is reached, not the familiar prompt-and-work
  interaction underneath.
- Surrounding buildings expose supporting functionality: one manages skills and plugins, and
  integrated terminal and Git tools keep the workflow in the same environment.
- The map is an entry point into real development tools, not just a display of animated workers.

### See where the work is happening · 03:34
- Once several agents are running, the next problem is understanding their activity quickly.
- A side panel shows high-level mission status and summaries of what each agent is doing.
- The map offers a spatial view: it is a projection of the filesystem, with directories placed on the
  map and files represented as rooms.
- Because an agent appears at a location, you can see visually which file it is working on.
- A change list shows everything that happened in a file, and because AgentCraft is orchestrating, it
  also records **which agent** made each change and when — full lineage of the work.
- From there he proposes a **heat map** to visualize collisions and potentially prevent them
  proactively. The walkthrough describes the visibility, not a demonstrated prevention algorithm.
- The views answer different questions: the mission panel says what an agent is trying to do, the map
  says where it is working, and the change list says what changed and who changed it.

### Faster reactions still consume attention · 04:44
- Visibility only helps if you can react. AgentCraft leans on another RTS mechanism: using **muscle
  memory** to cycle quickly between units that need help.
- Here the interruptions are requests to approve a plan or answer a question. Instead of searching
  for the next blocked session, you move through agents that need you.
- Salomon reports this let him use more agents in parallel — but only for a short time.
- The limit is human: there is a ceiling on how many ideas he can hold at once before getting tired.
- His takeaway is that reducing the cost of each intervention is not the same as reducing the number
  of interventions. Faster switching doesn't remove the mental cost.

### Delegate task discovery and supervision · 05:34
- To cut the burden of inventing the next task, he tells the agents to find missions. These appear as
  **quests** he can launch with a button — refactoring, testing, and other work he doesn't want to do
  himself.
- That reduces one cost but leaves the babysitting cost: someone still has to cycle through the
  agents and supervise them.
- A **campaign** moves more of that responsibility into the agent system. The user supplies a broad
  goal, and AgentCraft spins up a **container** (an isolated environment) in which the agents work.
- Inside it, the agents decompose the task, develop a plan, and present it. A **campaign
  orchestrator** then takes over ongoing supervision of the decomposed work.
- Container isolation is his stated reason for tolerating more exploratory activity inside a campaign
  — not a demonstrated guarantee that any action is safe.
- The central change is organizational: the human's effort moves toward planning and review instead
  of continuous babysitting.

### From supplying ideas to reviewing implementations · 06:42
- Once agents can run a campaign, even the starting idea need not come from the user. He describes a
  daily `cron` job (a scheduled task) that scans Twitter for interesting ideas, implements them, and
  leaves him to decide which results he wants. He says this is how he implemented channels quickly.
- The immediate consequence is a new queue: many pull requests waiting for review.
- **Review bundles** make that queue easier to assess. They bring together the changes, the reasons
  behind them, and the tasks involved, plus screenshots and videos.
- A reviewer can inspect both what the agents changed and visual evidence of the result, instead of
  reconstructing every campaign from its activity history.
- This shifts the balance between planning and selection. He asks how much time a detailed plan
  deserves if agents can produce ten implementations and you pick the best fit. Ten is a proposed
  exploration strategy, not a measured productivity result.
- The practical dependency is clear: shifting effort away from planning works only if the
  alternatives are cheap enough to inspect — which is why review evidence follows campaign autonomy.

### Share the workspace with people and their agents · 07:43
- Autonomy does not remove the need for other people; agents still have limits, so AgentCraft extends
  the workspace to human teammates.
- In his example, a product designer works with an agent running on the designer's own computer. Each
  participant can see the other's agents and understand what they are doing.
- The designer starts a new page; the engineer can wait for it to finish or hand it off immediately
  to agents on the engineering side and continue from there.
- He corrects his own wording from "my agents" to "our agents": the workspace makes the agents part
  of a shared effort rather than assistants hidden on each machine.
- Collaboration takes two forms: **direct prompts**, where a participant instructs their own agents
  or a teammate's agents, and a **shared conversation**, where humans and agents exchange messages in
  the same chat.
- In the demo an agent announces it is starting work, and a human replies that they are also working
  on it; on the next action the agent can account for the overlap.
- He also mentions lighter coordination: awareness of which files each person is changing. That
  supports soft coordination — it is not an exclusive lock or a guarantee against conflicts.

### Skills for directing the work · 09:24
- The walkthrough leaves features unexplored, but closes on the human transition behind the product:
  moving from writing code to directing others to write it can feel like entering unfamiliar work.
- Salomon's answer is that some of the necessary skills already exist outside the workplace. Games
  are one place people practiced managing units, distributing attention, and coordinating activity.
- AgentCraft's direction is to apply those skills through better visibility, more autonomy for the
  agents, and collaboration between people and agents.
- The product is offered as a free download, with a website and QR code shown at the end of the
  recording.
- He describes it as new and experimental, with changes still needed, and invites users into its
  Discord community to give feedback — the invitation is to test whether this style of orchestration
  improves real work, and to help shape it where it doesn't.

## Resources

- AgentCraft — project homepage for the RTS-inspired agent orchestrator, with setup and community
  links.
- Install AgentCraft — current prerequisites and guided setup for discovering and launching coding
  sessions.
- Campaigns — the campaign workflow covering planning, task worktrees, and verification evidence.
- Alliance Hall — shared rooms, remote-agent visibility, team messages, and file-intent awareness.
- Isolated agent containers — container setup and documented network, filesystem, and browser limits.
- MCP-UI — SDKs and examples for interactive tool interfaces using the MCP Apps standard.
- MCP Apps specification and SDK — the official standard for interfaces embedded in AI tools.