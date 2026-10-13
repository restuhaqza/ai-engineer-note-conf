# Your Attention Is the Bottleneck, Not Your Agents

| Field | Details |
| --- | --- |
| **Speaker** | Zack Proser |
| **Affiliation** | WorkOS |
| **Type** | Talk |
| **Date** | 2026-06-11 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=so9l_MwS2yg) |
| **Tags** | `speech-and-audio` `apis-mcp-and-protocols` `agent-engineering` `architecture` `developer-workflows-and-testing` `coding-and-developer-tools` |

## Summary

Zack Proser, who works on the Applied AI team at WorkOS, asks why producing more
software than ever still leaves developers fried by 11 a.m. His answer: agent
throughput is no longer the limit — human attention is. Agents can loop until they
meet the criteria you give them, but your judgment, taste, and ability to tell
whether something is actually solved degrade under load.

He makes it concrete with a Slack BlogBot bug. A sentence-case pass was mangling
acronyms like SCIM and SSO. Instead of cycling between the code, the ticket, and
Slack, he gave Claude Code read/write Slack access through MCP (a standard way for
agents to call external tools), told it to fix the bug and verify its own work, and
came back to a completed fix. Access to the real verification surface removed a
human round trip.

The rest of the talk is a stack for staying sustainable: **signal layers** that
filter incoming work, **voice-first** input, **remote control** so stepping away
from the desk does not stop execution, and a **self-improving system** that reviews
its own conversation history. Throughout, he stresses verification gates and,
unusually, the developer's own well-being — including an Oura Ring integration that
tells Claude when he slept badly.

## Key Takeaways

- **Attention, not agents, is the constraint.** Execution can scale faster than a person's ability to judge and review.
- **Verification closes the loop.** Giving an agent access to the surface it must verify removes a whole round trip with a human tester.
- **Filter your intake.** A signal layer that reads Slack and Linear for you avoids the feed that reliably steals your focus.
- **Voice changes throughput and posture.** Proser reports ~184 words per minute by voice versus a personal typing best of 90.
- **Stepping away need not stop work.** Remote control keeps the session running on the dev machine while you review PRs from your phone.
- **Independence requires verification.** Three gates — code checks, browser behavior, and a reviewing agent — are the condition for letting agents run.
- **Treat session history as material.** A scheduled pass over saved conversations finds missing skills and prevents next week's friction.
- **Never delegate what you cannot review.** Skill growth still comes from doing hard work by hand, then accelerating from a solid base.

## Detailed Notes

### Why more output still burns you out · 00:16
- Proser opens by asking who feels fried despite getting more done than ever — context switching was always expensive and is now worse.
- A BlogBot bug: a sentence-case pass was damaging acronyms like `SCIM` and `SSO`, usually forcing him across windows to check the workflow.
- Claude Code already had his Linear tickets; he added Slack read/write via MCP and told it to fix the bug, verify its own work, and not stop.
- The loop crossed the same boundary a human tester would: change the enforcer, submit a request through Slack, let BlogBot process it, then check the output.
- He returned to a completed fix instead of another "this part is still broken" handoff.
- The success was also unsettling — if the loop works, why not stack copies of it? He imagines fixing 150 bugs a day, but as a hypothetical, not reported throughput.

### Execution scales; judgment does not · 02:42
- Proser frames the asymmetry: agents scale, but attention lives in "meat space" and degrades under load.
- He cites Simon Willison's account of being wiped out by 11 a.m. after running four parallel agents, and argues each person must find their own limits.
- The developer still supplies judgment and taste: whether the result solves a problem and whether the criteria match human and business needs.
- Scaling task intake to agent capacity consumes that judgment faster than it replenishes.
- His response is four capabilities: signal layers, voice-first flows, remote control, and a system that improves itself.
- He has been coding by voice for about a year and a half; the other pieces extend that practice.

### Filter intake, and lower the cost of directing work · 05:59
- Opening Slack to check one thing exposes every other thread; Proser puts his own odds of distraction while combing Slack at 80%.
- A **signal layer** lets Claude Code inspect repeatedly: look for `@mentions`, DMs, and high-priority asks, then use Linear MCP access to deduplicate and find the real tickets.
- The developer sees actionable work through the environment already in use instead of entering a feed that can derail the day.
- **Voice** solves the opposite direction — getting intent into the tools.
- He reports a personal typing best of 90 words per minute and voice input regularly around 184; these are his observations, without an accuracy or correction comparison.
- The payoff is faster dispatch: speaking across three Cursor windows, Codex, and multiple Claude tabs while a typist is still entering one prompt.
- He expects small savings to compound over years.

### Leave the desk without moving the execution · 08:15
- The **shower principle**: focused IDE work is great for executing a clear blueprint, but that same focus narrows the alternatives you consider.
- Walking the dog, taking kids to the park, or showering loosens focus and lets a solution surface — Proser calls this diffuse thinking.
- Previously, stepping away meant stopping work; remote control changes that.
- With **Claude Code Remote Control**, enabled by a startup flag, command, or config, the session keeps running on the development machine with its filesystem and tools.
- Claude on his phone can reach that same session from another network, including cellular out on a trail; the phone is another interface, not a replacement environment.
- The sequence matters: establish the important work in a focused session, start the agents, confirm they are heading the right way, then walk.
- Benefits include less fixed desk posture and less repetitive strain; a new idea can be sent straight back to the session.

### Verification gates and a mobile working day · 11:09
- "Speed requires safety" is the condition for letting work proceed away from the desk; browser and computer-use tools expand what agents can check but need explicit obligations.
- **Gate one — code:** run lint, build, and unit tests automatically through hooks.
- **Gate two — behavior:** use the browser and click through the app to catch broken flows such as login.
- **Gate three — requirements:** have another agent review a written set of requirements and return feedback the implementing agent must address.
- He compares the third to Constitutional AI — an analogy for checking against written principles, not that training method.
- A working day with mobile feedback: begin focused (review the GitHub backlog, queue work in Codex, start priority features), confirm each agent's direction, then review PRs from the phone and send corrections into the existing loop.
- Natural-language comments in GitHub Mobile can drive integrations like Cursor Agent; he cites **Opus 4.6** as making this practical.

### Let the system improve itself · 13:22
- Fast execution alone can produce a disorganized week: productive Monday and Tuesday, interruptions midweek, a Friday that shipped work no one can reconstruct.
- A colleague pointed out that Claude Code saves conversations locally as `JSONL`; those sessions record both output and the effort required.
- A scheduled daily or weekly pass can scan that record for expensive reasoning, repeated clarification, and missing capabilities.
- The useful question is specific: which tool, MCP server, or skill would have prevented this particular struggle?
- Proser describes a skill that can create, evaluate, and improve other skills; Anthropic publishes a skill-creator for this purpose.
- He reports that a single Opus 4.6 pass can surface useful missing skills, making conversation history material for improving reliability instead of discarded output.

### Include the person — and build real skill first · 15:30
- Proser connects Oura Ring data to Claude via MCP; when his sleep is poor, Claude proposes doing only the first part of a project today.
- He jokes that he overrides it — but at least it makes him consider a break.
- Sleep and focus peaks belong alongside tickets, requests, and skills: delegation should preserve the person still responsible for quality and shipping.
- Without that intention, more powerful tools simply make burnout arrive faster.
- His adoption sequence: connect the highest-cost context switch (Slack or Linear), add a missing verification gate, then spend the recovered margin on a walk or time with the dog.
- On the first audience question, about early-career skill growth, his advice is not to delegate work you do not yet know how to do yourself.
- He uses agents for TypeScript, RAG, and AWS work because years of doing it directly let him catch hallucinations; he still recommends coding by hand and asking AI to test his understanding.
- On whole-stack changes: small bugs parallelize well, but features touching backend, database, and frontend need coordinated attention. He starts with **Git worktrees** so agents edit separate directories, then agent teams with clearly defined prompts and a shared spec.

## Resources

- [Claude Code Remote Control](https://code.claude.com/docs/en/remote-control) — connect a phone or browser to a session running on your machine.
- [Claude Code hooks reference](https://code.claude.com/docs/en/hooks) — configure lifecycle hooks that trigger checks.
- [Claude Code browser integration](https://code.claude.com/docs/en/chrome) — give the agent a browser to verify behavior.
- [Anthropic skill-creator](https://github.com/anthropics/skills/blob/main/skills/skill-creator/SKILL.md) — public skill for creating and improving other skills.
- [Ghost Pepper](https://github.com/matthartman/ghost-pepper/blob/main/README.md) — open-source, local dictation.
- [Constitutional AI](https://www.anthropic.com/research/constitutional-ai-harmlessness-from-ai-feedback) — the research analogy Proser references for written-principle review.
- [Zack Proser's video catalog](https://zackproser.com/videos) — includes the talk about working with AI in the woods.
