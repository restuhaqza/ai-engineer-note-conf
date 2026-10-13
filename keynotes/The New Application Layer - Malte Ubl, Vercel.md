# The New Application Layer

| Field | Details |
| --- | --- |
| **Speaker** | Malte Ubl |
| **Affiliation** | Vercel |
| **Type** | Keynote |
| **Date** | 2026-04-20 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=XKup1pj-34M) |
| **Tags** | `infrastructure-and-deployment` `coding-and-developer-tools` `apis-mcp-and-protocols` `enterprise` |

## Summary

Malte Ubl, CTO of Vercel, opens AI Engineer Europe with an argument about where the field is heading: **AI engineering is the successor to web development**, and software is increasingly made by agents, used by agents, and shipped as agents.

His central economic claim is that agents make software practical that was previously too expensive to build. That fills the gap of things that should exist but never got written. It also changes who — or what — uses the software: Ubl reports that over 60% of recent vercel.com page views came from AI agents. He argues cheaper software does not shrink demand for engineers; it expands the amount of software worth making.

From there he lays out the application-layer work: connect agents to messaging apps, give them a place to run, and focus on three archetypes — always-on work, research that speeds up a human decision, and surfacing information that already exists but is hard to use. He cites a 90% deflection rate for Vercel's in-house support agent and a sharp rise in support-team satisfaction.

He closes on infrastructure and strategy: separate the agent harness from the code it executes, build a stable layer above fast-changing models, and bet that model commoditization leaves room for application-layer innovation — much of which, he notes, is happening in Europe.

## Key Takeaways

- **Agents are a new kind of software.** They unlock automation that traditional development would have cost too much to build.
- **AI engineering is not a fad but a discipline.** Ubl sees it succeeding web development as the mainstream engineering field of the next decade.
- **More software, not fewer engineers.** Cheaper production tests how elastic demand for software really is; he sees demand rising with no sign of an S-curve yet.
- **Start with three agent archetypes:** round-the-clock work, research compression before a human decision, and making hidden information usable.
- **Keep the human decision explicit.** The abuse-review example returns a "pending-human-decision" packet and performs no enforcement, preserving the original risk profile.
- **Design for machine users.** When someone shows him a feature UI, Ubl asks for the CLI and the automation path instead.
- **Separate the harness from the code it runs.** He calls the common harness architecture fundamentally wrong and expects this to become a paradigm shift.
- **Europe can lead at the application layer.** AI SDK, Pi, and OpenClaw show consequential work without leading model development.

## Detailed Notes

### Connect an agent, then give it somewhere to work · 01:02
- Ubl opens with two pieces of his own "vibe coding" stack.
- **Chat SDK** connects agents to messaging apps such as Slack, Telegram, and WhatsApp.
- **just-bash** is a Bash interpreter written in TypeScript, giving agents a sandbox-like environment with near-instant startup — useful because agents already favor Bash.
- These are small building blocks of a larger shift: software built by agents, used by agents, and implemented as agents.

### A profession changing on both sides · 01:37
- Ubl used to run JSConf EU in Berlin, which ended its ten-year run in summer 2019 — fortunate timing before COVID.
- He sees AI engineering as web development's mainstream successor, shaping the next decade of software.
- The disruption has two sides: **how we build** (with AI) and **what we build** (AI applications and agents).
- Asked whether engineers still have a place, he answers yes. His self-deprecating proof: 25 years of experience would not make him good at TikTok videos, because people raised inside a medium develop instincts outsiders lack. Juniors formed in an AI-first world may be better at the new discipline.
- The second reason for optimism: agents expand what software can economically do.

### The software that was too expensive to write · 04:09
- Picture a circle of all software that should exist. Traditional development fills only part of it, because some automation requires too much scattered knowledge and exception handling to justify the cost.
- Agents make part of that previously uneconomic work affordable; Ubl expects we will find even more software worth building.
- More companies now choose to **make** rather than **buy** software, though he thinks SaaS companies will survive.
- More companies making more software means more engineering work even when each task is faster.
- He frames this as an economics experiment: the cheaper software becomes, the more of it we make, which pushes engineer demand up. An S-curve could come eventually, but he sees no sign of it yet.

### Three archetypes for useful agents · 06:47
- **Always running:** look for a job where moving from nine-to-five to round-the-clock service changes the value delivered, since agents do not need to sleep. Customer support is the obvious start; Sierra and Decagon exemplify the emerging agent-as-a-service category.
- **Compress the research:** many processes look like *event → research → human decision*. An agent can take just the research step. His example cuts a task from 30 minutes to five across 100,000 repetitions a year — without changing the process or increasing risk.
- At Vercel, Contact Sales runs this way: roughly 75% of submissions turn out to be support requests, and the rest are researched (LinkedIn, company size, routing) before a human checks. Abuse reports work the same, but with a more consequential decision at the end.
- In code, the abuse flow returns a **review packet** with status `"pending-human-decision"` and performs no enforcement — automating preparation without silently taking on decision authority.
- **Surface hidden information:** issue trackers go stale while the real updates sit in Slack or a meeting recording. An agent can gather that evidence and prepare the status a manager would otherwise request.

### Ask what people hate about their job · 11:36
- A direct way to find the next useful agent: ask employees what they hate most about their work.
- Vercel's in-house support agent reports a **90% deflection rate**, meaning it helps users in real time without escalating them (no reporting window or method given).
- The organizational result was a sharp rise in support-team job satisfaction: routine questions like a rejected credit card leave the queue, leaving more time for interesting cases and people who need deeper help.
- Ubl treats eliminating toil as a worthwhile mission in itself.

### Design for agents using the software · 12:37
- Agents are becoming the **users** of ordinary software, and developer tools are seeing it early.
- He reports that over the past seven days, more than 60% of vercel.com page views came from AI agents (Vercel-specific, no classification method described). Platform usage is also shifting from dashboard clicks to APIs and CLIs.
- This changes feature review: when someone shows him a UI, he asks where the CLI is, how it is automated, and how an agent uses it.
- As UI creation gets cheap, the programmatic interface becomes the thing that makes a product usable by its machine audience — so automation access belongs in the design, not bolted on at the end.

### Run the software without coupling it to the harness · 13:42
- If you did not write the code, you have fewer preferences about how it runs — but you still expect it to run, with near-prompt-level simplicity.
- New execution facilities follow: everyone is shipping sandboxes, and just-bash explores giving an agent a computer without a full computer.
- On security he sounds a 1999-style alarm: powerful systems are being deployed before anyone has learned to secure them reliably.
- His concrete architectural objection: most popular agent harnesses combine the environment running the harness with the environment executing generated code. **Separate the two.**
- He points to an unnamed Anthropic agent product, seen the day before, that does keep them separate — evidence the problem is solvable. Expect more paradigm shifts, since today's designs are not final.

### A stable layer above fast-changing models · 15:59
- The application layer persists even as the leading model changes; engineers build stable software on top.
- That means AI innovation need not happen at a model lab. Ubl argues Europe can lead in AI engineering: Vercel's **AI SDK** gets over ten million downloads a week and is led by Lars Grammel in Berlin; **Pi**, the coding agent, was made in Austria; **OpenClaw** is another example.
- His forecast: Europe will not lead on models, but it does not need to.
- He sketches two futures. In one, a model lab wins decisively and AI stays expensive, with application engineers reduced to a forward-deployed workforce for OpenAI, Anthropic, or Google.
- He considers the other more likely: model competition commoditizes the layer. He praises Claude and Codex, expects Google to catch up, and notes Google's cheapest infrastructure should keep prices low.
- In that world, agents create the business value and the application layer is where the next paradigm gets invented — which is where OpenClaw came from.

## Resources

- [Chat SDK brings agents to your users](https://vercel.com/blog/chat-sdk-brings-agents-to-your-users) — hooking agents into messaging apps
- [just-bash](https://github.com/vercel-labs/just-bash) — TypeScript Bash environment with an in-memory filesystem
- [AI SDK 5](https://vercel.com/blog/ai-sdk-5) — unified provider access, typed chat interfaces, and agent-loop controls
- [Pi agent toolkit](https://github.com/earendil-works/pi) — coding-agent toolkit and command-line interface
- [OpenClaw](https://github.com/openclaw/openclaw) — open-source personal AI assistant
- Decoupling the agent harness from execution — Anthropic's write-up on separating Managed Agents from generated code
- Vercel's earlier AI support case study — 2024 account of integrating AI into support workflows
