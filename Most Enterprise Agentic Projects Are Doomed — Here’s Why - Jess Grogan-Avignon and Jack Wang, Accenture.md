# Most Enterprise Agentic Projects Are Doomed — Here’s Why

| Field | Details |
| --- | --- |
| **Speaker** | Jess Grogan-Avignon, Jack Wang |
| **Affiliation** | Accenture |
| **Type** | Talk |
| **Date** | 2026-05-28 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=AGkzpxMdPn8) |
| **Tags** | `coding-and-developer-tools` `safety-and-governance` `enterprise` `leadership` `finance` |

## Summary

Jess Grogan-Avignon and Jack Wang, from Accenture, work with enormous enterprises — telecoms, national utilities, government, healthcare, and consumer goods — where a bad deployment can disrupt critical infrastructure. Control, process, and layers of governance exist for good reasons. But they were built for human speed, and the speakers argue that this mismatch, not model quality, is why most enterprise agentic projects stall.

They cite Accenture's AI-maturity research, where only 12% of companies qualified as "AI Achievers" and the rest are described as stuck. One project they delivered took about two weeks to build but twelve months to reach production, because infrastructure, security, AI-gateway, data-governance, and application teams all had to align. Giving developers coding agents makes this worse: more people become builders, so the supply of code needing review and release explodes.

Their prescription runs through five tensions — speed, value, delivery, trust, and moat. Treat AI investment like a venture portfolio instead of demanding certain payback, deliver in small hypothesis-driven loops, add agent autonomy only as outcomes earn it, and use customer feedback as the durable advantage. They summarize it as: bet like a VC, upgrade for machine speed, and engineer for trust from day one.

## Key Takeaways

- **The bottleneck moved.** Faster code generation doesn't help if approvals, security reviews, and deployment still run at human pace.
- **CI/CD underinvestment is real technical debt.** Turning repeatable human steps into executable code is the fix, and AI can help build that faster and cheaper.
- **Business cases assume the wrong things.** They require solution, value, and cost/time upfront — yet with AI, the work is how you discover them.
- **Fund a portfolio, not single bets.** A CFO should think like a venture investor: many bets, a few compounding wins, applied specifically to agentic transformation.
- **Deliver around hypotheses.** Models are non-deterministic, so build, evaluate, and iterate in small loops to produce evidence and confidence.
- **Progressive autonomy beats a big-bang switch-on.** Shadow → advisory → controlled autonomy → wider autonomy, gated by evidence about outcomes.
- **Trust is the durable asset.** Individual features come and go; accumulated trust decides whether people adopt what you build next.
- **Living memory is the moat.** Transactional records (CRM, ERP, SOPs) are a floor, not a fortress — the advantage is what you learn from real customer interactions.

## Detailed Notes

### When a bad deployment can take down infrastructure · 00:16
- The speakers work with enormous enterprises: telecoms, utilities serving whole nations, government, healthcare, and consumer goods.
- At that scale mistakes have real consequences, so these organizations built layers of control, process, repeatability, and governance — and grew successfully, but at human pace.
- Wang cites research where 12% of companies reached "AI Achiever" status; the other 88% are described as stuck piloting, spending heavily with limited return.
- He contrasts teams that ship Friday and roll back Saturday with enterprises where an afternoon's decision can take six months or more. Octopus, Klarna, and Shein are named as competitors willing to change the game.
- Their stance: staying through delivery (rather than leaving after the workshops) is where the useful knowledge comes from.

### Two weeks to build, twelve months to production · 03:10
- Executive conviction has arrived — leaders no longer need persuading that AI matters — yet enterprise speed hasn't shifted.
- The block isn't code quality or an unsolved context problem; it's a "human operating system" for delivery. Data access, security reviews, and deployment lean on stakeholder meetings rather than engineering automation.
- One project integrated with a company's central AI gateway and was handed a templated test config. Every config change needed manual review, which the team eventually automated.
- The application took about two weeks to build and another twelve months to reach production, because infrastructure, security, AI-gateway, data-governance, and application teams all had to align.
- Analogy: think of Google Search with an enterprise release process — three teams review results, legal signs off, then a quarter-end change freeze adds two more weeks.
- Coding agents raise the pressure: PMs, designers, and domain experts become builders, so the supply of deployable code jumps (they cite GitHub figures of one billion commits in 2025 and roughly 275 million per week in 2026).
- The deeper debt is years of underinvestment in CI/CD and engineering automation. Their prescription: every repeatable human process should become adaptable, executable code.

### Discover the business case by building · 06:52
- Business cases aren't wrong — they add oversight and force ROI questions — but they assume three things are knowable upfront: the solution and scope, the value, and the cost and time to deliver.
- With AI that's often backwards: doing the work is how you learn the answer.
- As prototyping and experimentation costs fall toward near zero, the question shifts from efficiency to unlocking new capabilities — products, services, and experiences that were previously uneconomic.
- They cite AI Achievers growing revenue about 50% faster than peers and link it to doing new things. This is a pre-pandemic observational association, not evidence that AI alone caused the growth.
- Emergent examples: Cursor's vibe-coder audience wasn't planned, and Claude Code wasn't mapped out far in advance. Enterprise examples include Walmart's trend-to-product design and an unnamed JPMorgan internal tool that became a revenue stream.
- The better question than "can we justify this?" is "what does it cost us not to explore this?"

### Fund a portfolio of bets · 09:31
- Wang says the CFO should think like a venture investor, at least when it comes to agentic transformation.
- A VC doesn't demand guaranteed three-year payback from one project; they back a portfolio, expecting many bets to fail while a few compound.
- That changes the unit of evaluation: not "does each project justify itself," but "are we placing enough well-chosen bets to find the important opportunities?"
- He explicitly scopes this to agentic transformation, not every corporate expenditure.
- If finance can't support that uncertainty, transformation should start there, because the downstream experiments depend on its funding decisions.

### Organize delivery around hypotheses · 10:32
- Data scientists and ML engineers already work with hypotheses, experiments, and statistical confidence; Wang jokes that enterprises often sideline them like the IT Crowd while treating Jira boards and PI planning as the real delivery system.
- Agentic work reverses that: models are non-deterministic and behavior emerges from interactions that weren't fully specified, so fixed feature plans and fixed milestones don't fit.
- Delivery effort gets consumed reconciling real system behavior with stakeholder expectations, polishing idealized upfront designs, debating guarantees nobody can yet substantiate, and writing status updates for decisions that never get made.
- The missing work is usually evidence that lets the organization decide. The loop: frame a hypothesis, build enough to test it, examine what happened, iterate.
- Teams need people comfortable with ambiguity who can translate statistical results into stakeholder confidence — a skill to hire, train, and reward, not an afterthought.

### Trust survives the feature · 13:10
- Grogan-Avignon notes we're learning to trust AI the way we trust search results, but in large enterprises the trust gap is still wide.
- Shipping one feature may matter less than the trust it builds; that trust covers content quality, accuracy, responsible use, and privacy.
- Frame delivery as deposits into (or withdrawals from) a trust account held with stakeholders, leadership, and customers.
- Features change and disappear, but accumulated trust shapes whether people adopt what comes next.
- The engineering question becomes: how do you build trust deliberately, quickly, and with evidence?

### Increase autonomy only as outcomes justify it · 14:36
- Traditional automation is seen as test, deploy, switch on — but agents' emergent behavior can't be exhaustively tested upfront.
- An evaluation suite is still necessary and must measure the right behaviors, but it doesn't replace a deliberate path to production exposure.
- **Progressive autonomy**, an exposure ladder:
  - **Shadow** — runs alongside humans and can't affect outcomes; compare human decisions with agent output.
  - **Advisory** — recommends in the live workflow; track human approvals, rejections, and corrections.
  - **Controlled autonomy** — acts in narrow, low-risk scenarios with explicit limits and kill switches.
  - **Wider autonomy** — broader scope, once confidence in the target behaviors is earned.
- The gate between stages is evidence about outcomes, not completing project activities or passing a fixed test set.

### Living memory and the learning organization · 16:48
- Wang asks what stays unique when AI can clone a shipped feature. Existing records (CRM, ERP, standard operating procedures) are "transactional memory," but competitors have their own — it's a floor, not a fortress.
- The real moat is "living memory": edge cases, corrections, emotional intent, and actual behavior at your specific scale and context — signals that belong only to you.
- Shipping is the start, not the finish. Every feature should either generate useful feedback or deliver something earlier feedback taught you; a feature that does neither is easy to copy.
- The five tensions — speed, value, delivery, trust, moat — require changing who owns the work: teams run hypothesis-driven loops, finance becomes a transformation partner, and governance speed becomes a CTO-level engineering problem.
- Closing argument: the advantage isn't being the earliest adopter but learning fastest, and neither trust nor living memory can be bought or copied.

## Resources

- [Accenture's AI maturity research](https://newsroom.accenture.com/news/2022/more-than-60-percent-of-companies-are-only-experimenting-with-ai-creating-significant-opportunities-for-value-on-their-journey-to-ai-maturity-accenture-research-finds) — explains AI Achievers and the four maturity levels.
- [Walmart's Trend-to-Product](https://corporate.walmart.com/news/2025/04/09/in-an-ever-changing-environment-walmart-uses-genai-to-create-cool-for-customers) — trend analysis plus generative design, with human designers and merchants.
- Building Claude Code with Boris Cherny — the tool's origins as a side project (article).
- GitHub's plan for Agents — interview with COO Kyle Daigle on agent-driven activity and revenue (article).
