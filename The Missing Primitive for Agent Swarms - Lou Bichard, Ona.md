# The Missing Primitive for Agent Swarms

| Field | Details |
| --- | --- |
| **Speaker** | Lou Bichard |
| **Affiliation** | Ona |
| **Type** | Talk |
| **Date** | 2026-05-23 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=5Sui_OnSRlY) |
| **Tags** | `apis-mcp-and-protocols` `infrastructure-and-deployment` `agent-engineering` `developer-workflows-and-testing` `coding-and-developer-tools` |

## Summary

Lou Bichard, field CTO at Ona, asks what it would take for coding agents to carry work from development into production without a person driving every change. His answer is the "software factory": a system that slowly takes the human out of actively operating each stage of the software development life cycle (SDLC). He is careful to say this is an early ambition, not routine practice.

The talk maps how agents get organized — swarms (one intent fans out and converges on a single task or pull request), fleets (agents spread across many repositories), and events (an external signal starts the work). It then looks at real internal systems like Stripe's Minions and Ramp's Inspect, and at Ona's own development-environment platform. Bichard's framing of harness engineering is that your repository stores the guidance and feedback agents need to get further on the next run.

His main argument is that execution is mostly solved but coordination is not. Runtimes, orchestration, and triggers already exist; what is missing is a way for an agent to prove it finished a stage and be granted permission to advance. He proposes state machines, durable execution, gates, and a small CLI that a local coding agent can call.

Why it matters: this names the layer most agent infrastructure skips. Tools built for human work tracking, like GitHub pull requests or Linear tickets, turn into a noisy queue once many agents run at once.

## Key Takeaways

- **Software factory = removing the human from operating the SDLC**, not one engineer juggling many agents at the same time.
- **Three organizing patterns:** swarm (fan out and converge on one PR), fleet (spread across repositories), and events (webhooks that start work).
- **Harness engineering** means encoding process, guidance, and feedback in the repository — for example `AGENTS.md`, skills, and tests — so later runs get further.
- **Runtime, orchestration, and triggers are largely solved; coordination is the missing primitive.**
- **Ona runs agents in VMs**, citing security isolation and noisy-neighbor compute contention as reasons containers are not enough.
- **Context rot** makes agents drift and skip steps, which leaves a human facing an overwhelming review queue.
- **Make advancement a checked operation:** a gate decides whether work may move from one SDLC stage to the next, instead of the agent declaring itself done.
- **No agreed protocol yet.** The prototype uses neither ACP nor A2A; Bichard cares more about co-designing a standard than shipping one implementation.

## Detailed Notes

### Who is still driving the work? · 00:21
- Bichard introduces himself as Ona's field CTO, with a background in principal engineering, platform engineering, and product management.
- His definition of a software factory: incrementally move the human out of the loop so work flows from development into production automatically.
- He distinguishes this from one engineer running many parallel agents — in that setup, the human still drives every change.
- He qualifies the ambition: many people he met were not yet thinking in these terms, and this is an early stage.

### Swarms, fleets, and events · 01:50
- **Swarm:** a single intent fans out to several agents, then converges back onto one PR or task — similar to the familiar parent-and-subagent pattern.
- **Fleet:** agents fan out across many repositories, useful for organization-wide changes. Bichard says this already exists in Ona.
- **Events:** an external signal activates agents, e.g. a PR is raised or a Linear ticket is created. Much of this is existing webhook infrastructure.
- Together these answer different questions: how to split a task, how to spread a change, and when to start work.

### Organizational workloads and harness engineering · 03:11
- Stripe's Minions plugs coding agents into existing infrastructure and reportedly drives thousands of pull requests; Ramp's Inspect runs background agents internally.
- These are examples of companies building execution infrastructure around agents, not treating an agent as one isolated chat.
- Ona has run development-environment infrastructure for about six years, with deeper agent integration in the last year or two.
- Its fleet feature launches agents on schedules or triggers for work like CVE remediation, raising test coverage, or enforcing a common change.
- The hard part is rarely the single task, which current LLMs can often do; it is applying it across thousands of teams or repositories.
- **Harness engineering** (Bichard credits OpenAI's article by Ryan Lopopolo) is an extension of context engineering: skills, instructions, and unit tests all give the agent guidance or feedback.
- The loop: let the agent try, find where it gets lost, encode that knowledge back into the repository, then let the next run go further.

### Execution infrastructure and isolation · 05:40
- Four capabilities are needed: **runtime** (somewhere to execute), **orchestration** (scale up and down), **triggers** (start on an event), and **coordination** (agents picking up and handing off work).
- Bichard regards the runtime as largely solved. Execution options include separate threads, Git worktrees, containers, VMs, or microVMs; Ona calls its units development environments.
- He argues for VM isolation for serious development tasks, on two grounds: security and resource contention.
- In his view a container is not a strong enough security boundary, and bursty workloads in Kubernetes pods can compete for compute (noisy neighbors).
- These are Ona's reasons for choosing VMs, not a guarantee that a VM alone solves every security or capacity problem.

### Two kinds of subagent · 07:17
- The demo (recorded in advance to avoid venue Wi-Fi) uses OpenAI's Symphony, which has a detailed repository specification for the agent to build against.
- Task one: **process-based subagents** — one VM holds the parent agent and the child processes it starts.
- Task two: a **fleet of separate VMs**, where adding a worker also means provisioning another environment; scale is bounded only by spending and cloud capacity.
- In the VM variant, the parent is the controlling agent; children get small slices of context and one task each, then message back to the parent.
- Both demos showed seven sub-items; the running view shows delegation in progress, not a finished implementation.
- A UX question emerges: as the task tree grows, how do you inspect and control the workers without losing sight of the overall task?
- In the process-level version, subagents appear as a stack at the bottom of one agent window; clicking a worker opens its own chat context while execution stays in the same VM.

### The SDLC hides many decisions · 10:20
- Bichard tried to build a project without touching any code. He concludes the technology can support that, but doing so exposes complexity a tidy lifecycle diagram hides.
- The familiar five-stage SDLC is a useful human summary but too coarse for an agent to follow.
- Planning alone contains many smaller steps: ticket breakdown, dependencies, and scope, each with its own gate.
- An agent needs the work inside a stage broken into actionable steps, not just the stage's name.
- The engineering problem is making agents follow those steps reliably and making progression deterministic. Starting a worker does not prove its prerequisites were met.

### Context loss and noisy coordination · 11:43
- As the context window fills, agents lose direction and become less effective — Bichard calls this **context rot** — and they tend to skip required steps.
- He links skipped steps to sycophancy: an agent asked to write tests may cut corners to please the user and finish.
- Once runtimes and orchestration exist, these failures become coordination problems.
- GitHub can hold a pull request, but a human may still need to review it, resolve a merge conflict, and repair CI; across many agents those signals become overwhelming.
- Bichard argues GitHub is a poor coordination layer for agents, and offers a similar criticism of using Linear beneath Symphony.
- The issue is not whether these tools can record tasks, but whether they give agents and supervisors a precise way to manage progress.

### Make progression explicit, and the open contract · 12:59
- His proposal combines established ideas: **state machines** to model the workflow, **durable execution** to keep a process running over time, **gates and compliance checks** to decide whether work may proceed, and **CLI packaging** so the machinery works locally and in CI.
- This is a proposed direction, not a finished product. Its purpose is to put the lifecycle rules somewhere an agent can invoke, rather than rely on its current conversation to remember them.
- The intended human role is **on the loop**: able to supervise and intervene without driving every change.
- He names context management as the hardest part, sees runtime/orchestration/triggers as solved, and flags security as a separate obstacle to more automation.
- The concrete shape of the gateway: a local agent (say Claude Code) calls a CLI that answers two questions — has work achieved the current SDLC stage, and may it proceed to the next?
- He sketches a small TypeScript gate where a work item stays in the `test` stage unless passing test evidence exists; the agent's request alone cannot satisfy the gate.
- He points to interim landscape pieces such as OpenClaw's `acpx` (built on ACP) and the nascent Fabro project, without claiming his prototype uses them.
- Asked about protocols, he says the prototype uses neither ACP nor A2A and is not built on an agreed standard yet; he is more interested in collaborating on the standard than the implementation.
- The unresolved contract is how an agent shows that development work reached a valid state and gets permission to continue — the missing primitive the talk leaves open.

## Resources

- [Background Agents (Bichard's visualization site)](https://backgroundagents.com)
- [Stripe's Minions](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents)
- [Harness engineering (OpenAI, Ryan Lopopolo)](https://openai.com/index/harness-engineering/)
- [OpenAI Symphony](https://github.com/openai/symphony) and its [specification](https://github.com/openai/symphony/blob/main/SPEC.md)
- [OpenClaw acpx](https://github.com/openclaw/acpx)
- [Fabro](https://github.com/fabro-sh/fabro)
