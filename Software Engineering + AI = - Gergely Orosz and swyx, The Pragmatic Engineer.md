# Software Engineering + AI = ?

| Field | Details |
| --- | --- |
| **Speaker** | Gergely Orosz, swyx |
| **Affiliation** | The Pragmatic Engineer |
| **Type** | Keynote |
| **Date** | 2026-04-21 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=CS5Cmz5FssI) |
| **Tags** | `enterprise` `coding-and-developer-tools` `leadership` |

## Summary

This closing keynote is a fireside chat between Gergely Orosz, who writes The Pragmatic Engineer, and swyx, who hosts it. They ask whether AI coding tools are actually making software engineering faster, and they start with a strange new workplace habit: "token maxing," or burning AI usage just to make the numbers look good.

Orosz talks to engineers and managers across large tech companies. He argues that many adoption pushes came from leadership fear rather than demonstrated value. Once usage becomes a visible metric, it can turn into a performance signal that rewards busywork instead of useful output.

The conversation then widens. swyx presses on whether the push is still worth it, and the pair separate individual speed from organisational speed. They also cover how the engineer's role keeps growing, why directing agents is not the same as managing people, and the large internal AI infrastructure big companies are quietly building.

It ends on Orosz's own story: leaving Uber, starting a newsletter as an experiment, and turning it into a business without working every vacation.

## Key Takeaways

- **Token maxing is a warning sign.** When AI usage becomes a leaderboard or a spend target, people optimise the number instead of the outcome.
- **Goodhart's law applies.** A measure that becomes a target stops being a good measure; lines of code were the earlier version of this mistake.
- **Leaders pushed adoption from fear,** not proof. Seeing competitors use AI created pressure to mandate it, even when the tools were only mildly useful on existing code.
- **Feeling faster is not finishing faster.** A METR study found developers felt around 20% more productive but actually took longer on the same tasks.
- **Pick the right unit of analysis.** Helping a non-engineer colleague make progress on their own can create value that an individual coding-speed study never sees.
- **Tool intuition takes practice.** Understanding how models work does not tell you when to delegate or what context to give; only repeated use does.
- **Orchestrating agents is not people management.** You get the coordination without the human drama, conflict, and slow feedback loops.
- **Much of the work is internal.** Large companies are rebuilding platforms, gateways, and review tools before customers see new features.

## Detailed Notes

### Token maxing becomes a performance proxy · 00:28
- Orosz heard about "token maxing" only about a week before the talk, and his messages filled up with stories from big companies.
- **Token maxing** means spending or generating AI tokens mainly to push a visible usage number higher.
- At Salesforce, he says employees can look up how many dollars peers spent on tokens; at Meta, managers described usage as one input among many in performance reviews.
- Layoff anxiety makes a fuzzy metric feel decisive. A low-impact engineer with low usage can be framed as "not even trying."
- The behaviour gets concrete: instead of reading docs, ask an agent to summarise them, then keep asking follow-ups to raise the count.
- At Microsoft, he heard complaints about autonomous agents building junk just to inflate usage.
- A Meta leaderboard was removed after an article made it look bad, yet the behaviour continued — the fear of being measured stayed.
- He tentatively recalls a Salesforce minimum spend target of about $175 per month, which nudges people to "max" early in the month.

### Why leaders pushed adoption · 04:48
- This is a classic **Goodhart's law** problem: once a measure becomes a target, people adapt to the measure.
- Orosz recalls a CTO dinner in Amsterdam about six months earlier. One CTO complained that engineers had AI subscriptions but barely used them.
- That was before **Opus 4.5** and similar models. On mature codebases, tools like Cursor were only mildly useful for refactoring or finding bugs.
- The Dutch National Bank's engineers were motivated for a different reason: their job was to regulate the technology, so they had to understand it.
- Leaders also saw Anthropic writing lots of code with Claude Code while revenue grew, and drew a causal-sounding conclusion from it.
- The Coinbase example: Brian Armstrong demanded adoption within a week and later fired an engineer. Base salaries in the $300k–$400k range made the message land hard.
- Orosz's point: the message spread through consequences, not through a demonstrated improvement to engineers' own workflows.

### Compliance, output, and the right unit of productivity · 08:01
- Orosz compares token targets to LeetCode-style interviews: both select for people willing to satisfy an irrelevant requirement.
- He expects some engineers to turn the pressure into genuinely useful projects anyway.
- Startups behave differently, he says: they care about what got built and what it cost, not token counts.
- On whether AI speeds things up, he sees gains for the individual but is unsure about gains across teams. Anthropic is an exception, not the pattern.
- The **METR study** is the clear illustration. Participants felt roughly 20% more productive but were measured as slower — the original report found about 19% longer completion times.
- The sample was small (the published study had 16 developers and 246 tasks, though Orosz remembered 30 on stage), and one strong outlier skewed the average.
- swyx changes the unit of analysis. Even if the engineer's own speed barely moves, enabling non-technical colleagues to make progress directly removes a dependency on that engineer.
- His joke about "serverless developers" points to a real organisational effect: a pull-request-level study would never capture it.

### Learning new workflows takes time · 10:42
- Orosz recalls a 2024 conversation with Simon Willison, who was still changing his workflows after two years of experimentation. There was no manual.
- Understanding the theory of a model does not make you good at using it. Compilers are the contrast: understanding them genuinely helps you write better low-level code.
- The distinction is between knowledge and intuition about tool use. Knowing how attention works does not tell you when to delegate or what context to provide.
- A workflow that works for one person can break when brought into a team, forcing another round of learning.
- The teams getting the most value, he says, have low ego, are open to learning, and are willing to drop old assumptions — without throwing away hard-won engineering experience.

### The engineer's role keeps widening · 12:41
- This trend predates AI. Venture-funded startups had to move fast with small teams, so engineers took on more.
- Since the mid-2010s, testing and DevOps responsibilities folded into software engineering; more recently, product responsibility has followed. Companies hired "product engineers" as early as 2022.
- AI speeds up the expansion and raises expectations even for early-career engineers, who are now asked to plan, understand the business, and own outcomes.
- Teams are shrinking. Orosz relays a conversation with a VP of engineering at John Deere (a roughly two-century-old tractor maker) who said two-pizza teams are becoming one-pizza teams, partly because of these tools.

### Orchestration is not people management · 14:42
- swyx asks whether directing agents makes everyone an engineering manager. Orosz rejects the analogy outright.
- A quick show of hands suggests only around 15–20% of the room had management experience or wanted it.
- Real management means distance from the product and responsibility for interpersonal conflict and personal problems. Agents bring none of that.
- The better comparison is a tech lead or experienced engineer directing and reviewing work.
- Orosz cites DHH's "mech suit" image: agents extend what one engineer can do while keeping the engineer in control.
- Feedback differs too. A management decision may take six months to show results; agent work returns feedback much faster.
- Maximum concurrency is not the goal. He mentions Mitchell Hashimoto, who is happy with a small setup and one background agent.

### The hidden infrastructure behind product output · 17:25
- Outwardly, big AI spenders like Uber have not launched a wave of new customer features. Inwardly, a lot is changing.
- Buying Cursor or Claude Code is only part of it. Companies are building custom background coding agents wired into their monorepos.
- They are also building **MCP gateways** (a single entry point for agents to reach internal services and tools), integrated with service discovery.
- On-call tooling is being retooled, and internal code review systems are categorising changes by risk.
- Orosz names Airbnb, Intercom, Meta, Microsoft, and midsized companies doing similar work.
- He gives three reasons: internal tools are a low-risk way to learn; giant codebases do not fit in a context window, so custom retrieval (basic **RAG**) beats off-the-shelf tools; and anything labelled "AI" or "agent experience" is easier to fund.
- The result is a lot of duplicated effort. His joke: if you're at a large company and not building an MCP gateway, what are you even doing?

### Shopify's early bet, and building The Pragmatic Engineer · 20:39
- Orosz warns against dismissing big-company spending as waste: if an incumbent like Google executed well, it would be a serious competitive threat.
- Shopify's example, as he tells it: in 2021, Farhan Thawar heard GitHub was building Copilot and asked Thomas Dohmke for access.
- When told it wasn't for sale, Thawar offered a company-wide rollout in exchange for honest feedback from 3,000 users. Shopify got Copilot about a year early, with weak tooling and real churn.
- The tradeoff was deliberate: absorb cost and instability to be a few months — perhaps six — ahead of competitors. That is worth it for tech businesses, but less so for companies whose advantage is mainly physical.
- The bet is also about hiring: denying engineers the best tools makes it harder to recruit the best people.
- On his own business: after COVID-era Uber layoffs, low morale pushed him to take a break and write. He launched The Pragmatic Engineer a year after leaving Uber.
- His first (free) article explained Uber's platform/product split, which he says hadn't been written about publicly. Before publishing anything, 100 people prepaid $100; within six weeks he had 1,000 paying subscribers — enough to replace his old base salary.
- It became the number-one paid technology newsletter about four months after launch and held that spot for three years. He worked 50–60 hours even on vacation for the first two years, then hired a researcher and started a podcast to make the pace sustainable.

## Resources

- **The Pragmatic Engineer** — Orosz's software-engineering newsletter, blog, and books.
- **METR's early-2025 developer productivity study** — the original report on AI-assisted task completion.
- **METR's follow-up and experiment redesign** — why selection effects and concurrent agents complicated the first results.
- **DHH's new way of writing code** — Orosz's interview with David Heinemeier Hansson about agent-first development.
- **Uber's agent identity and MCP gateway architecture** — Uber engineers on agent identity propagation and their MCP gateway.
