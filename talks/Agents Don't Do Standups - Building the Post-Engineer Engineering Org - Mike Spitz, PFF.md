# Agents Don't Do Standups: Building the Post-Engineer Engineering Org

| Field | Details |
| --- | --- |
| **Speaker** | Mike Spitz |
| **Affiliation** | PFF |
| **Type** | Talk |
| **Date** | 2026-05-15 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=VMemhtlsoNk) |
| **Tags** | `architecture` `evals` `enterprise` `developer-workflows-and-testing` `coding-and-developer-tools` |

## Summary

PFF is a sports-data company that serves NFL and NCAA teams and runs a consumer
side of fantasy football, betting and draft tools. Its CTO, Mike Spitz, presents a
three-month case study (January–March) staffed by just two engineers. The goal was
unusual: instead of making each engineer produce more, the team asked how to make
the *agents* move faster.

The reported numbers are strong — 25× more deployments, about 10× output per
engineer, and a feature set finished in under two months against a four-month
estimate. Spitz stresses these are team delivery rates, not individual
productivity, and that a smaller team already moves faster. The more interesting
claim is compounding: because one engineer was freed in under a month, the next
piece of work could start while the other finished.

The bigger change was to process. Scrum ceremonies did not survive. The
project-manager role disappeared and coordination moved into a specification and a
lightweight design document (LDD). Ticket status updates itself from pull-request
events, so standups lost their reporting role. A QA agent checks staged work
against acceptance criteria; an automatic repair loop is planned but not yet
built.

Spitz frames this as a speed problem, not only a tooling one. Humans stay on
intent, design, security and product feel, while repeatable and verifiable work
goes to agents. He recommends a phased rollout led by the strongest engineers and
warns that the advantage compounds: falling a few months behind can become a year.

## Key Takeaways

- **Optimize for agent speed, not engineer output.** The framing shifted from "help engineers produce more" to "help agents move faster."
- **Reported results:** 25× deployment frequency, ~10× output per engineer, and completion in under two months against a four-month estimate.
- **Freeing one engineer early compounds.** Subsequent work can start before the current project ends.
- **Customer quality rose** to a reported 8.6/10, against a pre-AI average of about 7–7.5/10.
- **Scrum was replaced, not just trimmed.** Every-other-day huddles and a spec → LDD → tickets → PRs flow took over from standups and sprint planning.
- **Ticket status follows PR events** (opened, in review, merged), making status meetings redundant.
- **Start with your strongest, most system-literate engineers** and expand in phases; test on non-critical systems first.
- **Automate verifiable work, keep design and security human.** The self-healing repair loop was still only a plan.

## Detailed Notes

### What if the workflow optimized for agents? · 00:18
- PFF supplies data to NFL and NCAA teams and runs a consumer side: fantasy football, betting and draft tools.
- Scale: ~200 employees and ~20 engineers, distributed across India, Spain and the US; 100M annual page views and 9M annual drafts.
- The team was busy but falling behind competitors because attention had drifted to betting while users wanted more from the draft product.
- Spitz's personal experiments (around November) grew into a pilot with the strongest frontend engineer and a strong full-stack engineer.
- He reframes the goal: not "help engineers output more" but "help the agents be quicker."
- He places Agile, software craftsmanship and even engineering perks in a history of optimizing around scarce human capacity. If that bottleneck moves, the process must move with it.

### More deployments, and a harder question about output · 02:30
- The two-engineer pilot deployed about five times a day; a ~10-engineer team deployed roughly once every five days — a 25× team-level difference.
- Caveat Spitz states plainly: small teams are always faster, so part of that gap is team size.
- The pilot still had to coordinate its deploys with the larger team.
- Output was measured by blending ticket counts with code complexity, giving ~10× per engineer; the weighting formula is not shown.
- The feature set finished in under two months versus a four-month estimate. One engineer was unblocked in under a month, against three months blocked under the old plan.
- Customer surveys averaged 8.6/10, versus a pre-AI estimate of 7–7.5/10.

### Replace ceremonies with a shorter feedback loop · 04:53
- The project-manager role was removed, cutting the "game of telephone" between requirements and feedback.
- Replacement: huddles every other day, roughly 30–60 minutes, with engineers, product and data science reviewing what was built recently.
- The team shipped minimum viable versions to production fast to collect feedback early.
- The workflow: the agent interviews the humans to build a spec → spec feedback → agent-written LDD → engineering feedback → automatic tickets → pull requests.
- The LDD (lightweight design document) is not Claude-specific: a skill produces it by drawing on past LDDs, so new work matches the org's engineering approach.

### Let work events carry the status · 06:51
- Sprint planning disappeared: an hour spent estimating tickets no longer changed what the team did.
- A future constraint Spitz expects: estimating token cost per task once token subsidies end. It is not in use yet.
- Ticket status is driven by PR events instead of status meetings:
  - PR opened → ticket in progress
  - PR in review → ticket in review
  - PR merged → ticket closed
- Sprint refinement moved into the spec and LDD. Generated tickets are structured to avoid blocking each other, with real dependencies flagged explicitly.
- Retrospectives were replaced by customer satisfaction surveys, normal delivery metrics (like deployment frequency) and immediate escalation of issues by engineers.

### Start with people who know where the system bends · 08:28
- Begin with the engineers everyone already asks when stuck — they hold the most system knowledge.
- Roll access out in phases rather than giving everyone coding assistance at once.
- Spitz prepared for about two months by pushing small, non-critical proof-of-concept features to production.
- Those experiments got little traffic, so mistakes had limited consequences before the approach touched the heavily used draft product.
- Analogy: not everyone can drive a sports car. The curious engineer who investigates unfamiliar code will thrive.
- Engineers who need a fully prescriptive spec will find the shift harder, because the workflow asks them to help define intent.

### Automate verifiable work; preserve design judgment · 10:09
- Agent work needs verifiable, deterministic tasks — tests, but also checks specific to the product.
- PFF examples: feature flags for trunk-based development, generating interactive elements, and generating analytics for them.
- **Split code review by responsibility:**
  - Agents handle variable naming, style and the opinionated nitpicks engineers dislike hearing about.
  - Engineers handle system design and the larger implications of a change.
- This removes some emotional friction and frees people for the big picture.
- Humans stay heavy on the spec and LDD, and on the product boundary: generated code must still feel like the company's other products.
- Generating a feature does not decide whether it belongs in the product.

### Compose the workflow, then test the result · 12:00
- Think of the lifecycle as a factory of small composable steps (building a door vs fitting a steering wheel).
- Repeated operations — branch naming, feature flags, building an API in the house pattern — become skills.
- Warning: third-party skills may carry design opinions that conflict with your org, so composed skills must agree with each other.
- The spec → LDD → ticket → PR path runs largely autonomously, with human feedback built into the design stages.
- QA sequence: merge → auto-deploy to staging → a QA agent reads the tickets and acceptance criteria → the agent checks the staged work and flags failures.
- Checks run against the deployed result, not just the agent's description of its code.
- **Planned but not built:** an agent that reads failed acceptance criteria and auto-creates corrective PRs, closing a self-healing loop.

### Make guardrails work, then move at a pace engineers can sustain · 14:09
- Agents still take shortcuts, so security, product feel and the right scale of an implementation stay human jobs.
- A sufficiently prescriptive LDD prevents sprawling, over-engineered code before it is written, instead of unwinding it in review.
- Start adoption with boring, repetitive work engineers dislike — removing it wins immediate buy-in.
- Scrutinize process like code: ask what a long-standing meeting actually helps the team accomplish.
- Encode the org's engineering culture in skills; PFF's example is the service–repository pattern for APIs.
- Functional guardrails must be in place before autonomy, and the strongest engineers should build the first workflow.
- Do not onboard everyone at once with a hackathon. Every org's style differs, so expect a phased transition.
- Avoid being too conservative: the compounding advantage means a few months behind can become six, then twelve.

## Resources

- Extend Claude with skills (Claude documentation)
- DORA's software delivery performance metrics (documentation)
- [Manifesto for Agile Software Development](https://agilemanifesto.org/)
- "The AI Playbook That Works, and Why Yours Doesn't" — companion talk abstract from Mike Spitz
