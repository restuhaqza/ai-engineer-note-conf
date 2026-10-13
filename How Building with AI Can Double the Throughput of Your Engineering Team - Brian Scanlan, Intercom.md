# How Building with AI Can Double the Throughput of Your Engineering Team

| Field | Details |
| --- | --- |
| **Speaker** | Brian Scanlan |
| **Affiliation** | Intercom |
| **Type** | Talk |
| **Date** | 2026-05-15 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=4_VQBbs2iQA) |
| **Tags** | `agent-engineering` `developer-workflows-and-testing` `coding-and-developer-tools` `leadership` |

## Summary

Brian Scanlan, a senior principal engineer at Intercom, describes "2X" — a program
to double engineering throughput in a year without doubling the team. Intercom is a
15-year-old B2B SaaS company that pivoted to AI, and its customer-support agent,
Fin, reports more than 8,000 customers.

Early AI adoption (GitHub Copilot, then Cursor and Augment) produced only marginal
gains by mid-2025. So leadership made AI use an explicit expectation, written into
job descriptions for engineers, designers and product managers, and staffed a
full-time team to help hundreds of people adopt. They also standardized on Claude
Code instead of spreading effort across many tools.

The core investment is durable knowledge: small, testable skills that capture how
Intercom does work, backtested against historical code, incidents and finished
projects. A security incident showed the payoff. Scanlan gave Claude Code a problem
rather than a task, and it found and used an internal skill he did not know existed.

Intercom reports doubling PR throughput in under a year. That moved the bottleneck
to code review, so the team built a tested suite of automatic approvers (including
Codex) and validated them against historical examples. Scanlan closes by noting the
shift is spreading beyond engineering into product and design roles.

## Key Takeaways

- **Make a measurable bet.** "2X" targeted doubling engineering throughput in a year without doubling headcount, tracked mainly as code changes per R&D person.
- **Mandate plus support.** AI adoption was written into expectations for engineers, designers and PMs, and backed by a full-time team, hackathons and visible examples.
- **Pick one platform and go deep.** Intercom standardized on Claude Code, comparing tool sprawl to running multi-cloud.
- **Treat agent knowledge as the product.** Small, durable, testable skills capture institutional know-how and are backtested on real work.
- **Give the agent a problem, not a task.** Let it choose which skills to invoke; Scanlan's security-incident story is the proof.
- **Throughput moves the bottleneck.** After PRs doubled, code review became the constraint.
- **Automate review with evidence.** Intercom backtested approvers against history and had humans label outputs; the dashboard shows a 17.6% automatic-approval rate.
- **Measure behavior, not vibes.** Skill-invocation events go to Honeycomb; full session transcripts go to S3 for mining and skill assessment.

## Detailed Notes

### An AI business still has to reinvent how it builds · 00:17
- Intercom is a 15-year-old privately held Irish-American B2B SaaS company (~1,400 people across six cities) that pivoted to AI the weekend ChatGPT arrived.
- Its revenue growth is a U-shaped recovery against a downward trend among public SaaS peers.
- Being an "AI company" means more than wrappers: Fin, its support agent, has 8,000+ customers and reports industry-leading resolution rates.
- Fin revenue is described as approaching 100 million (currency and period not specified). Intercom says its own model now serves all English text Fin conversations, cheaper and faster than frontier models.
- It reports roughly 2 million resolutions a week and offers direct access to its model suite.
- These are company-reported deployment and performance claims; the talk does not give the evaluation conditions behind the model comparison.

### Give the change a measurable target · 04:15
- Intercom's platform group owns uptime, performance, security, cost, observability and internal developer productivity, largely on Ruby on Rails monoliths.
- Fast, iterative shipping was already core; the company's essay is titled "Shipping is your company's heartbeat."
- Early AI tools (Copilot, Cursor, Augment) gave marginal improvements by mid-2025, but did not change how the org built software.
- The response: **2X — double engineering throughput within a year without doubling the team.**
- The primary measure is code changes per R&D person, alongside developer surveys and DX.
- Scanlan acknowledges every measure is imperfect, but argues that a large throughput increase should be expected if the new ways of working are real.
- The program coincided with a big jump in model and harness capability over the Christmas break, which he credits as a major contributor.

### Make adoption an organizational responsibility · 05:54
- Leadership made the expectation explicit: not adopting AI means you are not meeting expectations — for engineers, designers and PMs.
- The message had to be repeated constantly across many forums; one announcement would not change habits.
- Supporting work combined reinforcement, practice and staffing:
  - Visible examples: automated Slack announcements of skill updates, and teams celebrating useful work and sharing techniques.
  - Time to learn: hackathons and AI immersion days.
  - Dedicated support: a growing full-time 2X team.
- Advice for medium or large orgs: put your best engineers on this full time, or every employee solves the same adoption problem alone.

### Choose a platform, then onboard the agent · 07:22
- Intercom standardized on Claude Code instead of letting people pick any editor or agent.
- Analogy: it is like choosing one cloud provider — spreading investment prevents compounding platform benefits. Exceptions need specific, impactful reasons.
- Goal: make Claude act like a senior Intercom engineer on any technical task. Anything an engineer can do on a laptop, the agent should be able to do — within existing permissions, controls and audits.
- Access is not enough; the agent must learn Rails conventions, architecture, React patterns, testing standards and security rules.
- The working loop:
  1. Use the agent for technical work.
  2. Notice where it takes a wrong path or misses a convention.
  3. Update shared guidance.
  4. Capture reusable knowledge in skills, and use hooks to enforce required behavior.
- Delivering the platform is real ops work: Intercom pushes internal Claude plugins to laptops, and debugging installs across hundreds of machines is like managing Python environments.

### Put durable knowledge into small, testable skills · 11:57
- Intercom is technically conservative: use a few tools extremely well, rather than everyone building custom multi-agent orchestrators.
- The preferred unit of investment is a small, durable, testable skill.
- Historical code changes, incidents and completed work let the team backtest a skill: does it handle known work to a high standard?
- Continuous improvement includes making skills able to update themselves.
- Don't get trapped behind custom infrastructure; stay able to adopt new capabilities as the provider ships them.
- The durable asset is accumulated working knowledge, not a commitment to any one harness.

### Give the agent a problem it can investigate · 13:38
- Asking an agent to run a named skill still helps, and Scanlan does it — but the target interface is a problem description.
- Let the agent infer intent and pick the relevant capabilities, so the requester does not need to know every internal procedure.
- Security-incident example: Snowflake table metadata was accidentally published to a public GitHub repo.
- Scanlan opened Claude Code, asked it to join the incident's Slack channel and investigate.
- An internal skill he did not know about already captured the company's data-breach policy, criteria and procedure.
- Claude downloaded the files, analyzed the exposure, concluded it was innocuous and gave next steps — about two minutes versus an estimated 20 minutes manually.

### Higher throughput moves the bottleneck to review · 15:55
- The platform decision was made in December and rolled out in January; Intercom reports doubling PR throughput in under a year.
- A separate dashboard figure for Claude Code pull requests is described only as "in the nineties," too imprecise to quote exactly.
- Code review became the bottleneck. The dashboard showed a 17.6% automatic-approval rate.
- Getting there took real work: backtesting approvers against historical changes, human labeling, and shaping PRs toward safe, simple changes.
- Intercom worked with auditors on SOC 2, ISO 27001 and HIPAA. Scanlan's view: human approval is not inherently required for every change, but you must understand the process and keep audit controls.
- The approval suite is tested and includes Codex for review — a deliberate exception to single-platform consolidation.
- Observability: basic skill-invocation events from hooks go to Honeycomb; full session transcripts go to S3 for mining and assessing skill effectiveness.

### Build the skill by doing the work, then widen the scope · 19:10
- The shared plugin system has hundreds of contributors and thousands of lines of code; base plugins handle transcripts, session syncing and safety hooks.
- Example skill: flaky-spec repair. Intercom has hundreds of thousands of tests, and heavy shipping encouraged pushing past intermittent failures.
- Scanlan did not enumerate every cause up front. He gave the agent a goal, guided it through real repairs, and let the procedure emerge.
- The result organizes knowledge into lookup tables, shortcuts and progressive disclosure.
- He says the skill produces fixes that would impress him coming from senior Rails engineers.
- Defects had been rising but are now closed faster than ever, partly from deliberate backlog-zero work and partly from faster day-to-day repair.
- More throughput also strained infrastructure: CI "melted" and had to be fixed.
- Claude Code spread beyond engineering, and Intercom experimented with single-person product teams. Scanlan used internal skills to do product-management work himself.

## Resources

- [Fin's original announcement](https://www.intercom.com/blog/announcing-intercoms-new-ai-chatbot/)
- [Shipping is your company's heartbeat](https://www.intercom.com/blog/shipping-is-your-companys-heartbeat/)
- "How we use Claude Code today at Intercom" — article by Brian Scanlan
- "2× – nine months later: We did it" — follow-up on Intercom's per-person PR throughput measure
- "AI is approving our pull requests" — Intercom's automated-review design
- "The age of vertical models is here" — Fin's specialized-model announcement
- [brian.scanlan.ie](https://brian.scanlan.ie/)
