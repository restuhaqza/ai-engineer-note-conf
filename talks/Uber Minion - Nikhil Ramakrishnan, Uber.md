# Uber Minion: Agent-Ready Dev Platform Behind 11% of Merged PRs

| Field | Details |
| --- | --- |
| **Speaker** | Nikhil Ramakrishnan |
| **Affiliation** | Uber |
| **Type** | Talk |
| **Date** | 2026-05-07 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=o7G5EHDqQ1g) |
| **Tags** | `uber` `minion` `devpod` `shepherd` `background-agents` `developer-platform` `agentic-dev` `toil-automation` `migrations` `platform-engineering` |

## Summary

Nikhil Ramakrishnan, a software engineer on Uber's developer platform team, explains how Uber built one of the most mature internal AI platforms in the industry through deliberate, compounding infrastructure bets made over many years -- long before the current AI boom. The headline result: Uber's background agent platform, **Minion**, now generates **over 11% of all merged PRs** across the entire company.

The conversation traces the lineage of Uber's platform thinking back to a real engineering pain point: full-stack engineers who needed to fluidly switch between Android, iOS, backend, and frontend work required **environment isolation** as a hard prerequisite. This drove the creation of **DevBox / DevPod** -- a remote, containerized development environment that turned out to be the foundational primitive not just for onboarding human engineers, but for onboarding **agents** as well.

Minion itself is framed around a sharp philosophical insight: engineers are not blocked by the *complexity* of most tasks, but by the **toil** surrounding them -- validation loops, CVE remediation, small mechanical changes -- where the fix is trivial but the surrounding ceremony (test, validate, ship) consumes the bulk of the time. Minion's job is not to write big features, but to **stack-rank toil into the background** so that by the time an engineer reaches the validation step, 50-70% of the work is already done.

Alongside Minion, Uber runs **Shepherd**, a large-scale migration management system operating across hundreds of millions of lines of code, treating "large-scale change" itself as a first-class primitive. Throughout, Nikhil stresses that adoption was driven by **developer satisfaction data**, not mandates -- engineers are "builders" who must be given tools they can mold, room to fail safely, and proof that the platform genuinely removes pain rather than overpromising capability.

## Detailed Notes

### Uber's Long-Compounding Platform Bets
- **Behind Uber the app** sits an engineering organization that has been building one of the most mature internal AI platforms in the industry through deliberate, compounding infrastructure investments over a long time horizon.
- **Key systems mentioned:**
    - **Minion** -- background agent platform responsible for >11% of merged PRs company-wide.
    - **Shepherd** -- large-scale migration management across hundreds of millions of lines of code.
    - Custom internal AI tools for test generation, code review, and more, all compounding on top of each other.
- **Speaker background:** Nikhil has been at Uber for many years ("since [a long time ago] and just never left"). He's an early adopter of cloud-based dev tooling inside Uber and was a key force in building the primitives that underpin Uber's agentic development platform.

### The Origin: Environment Isolation Was Not a Soft Problem
- Uber reached a point where engineers were no longer specialists. A typical engineer might build Android, iOS, backend, and frontend code -- effectively operating as **fully product-focused engineers** wearing multiple hats.
- Switching between these stacks required **environment isolation**, which was treated as a hard, non-negotiable problem rather than a soft convenience.
- Nikhil built a **rudimentary container for local development** for Android engineers. Within a year, it evolved into a prototype called **DevBox**.
- **DevBox's value proposition:** instead of a fragile, manual local setup, engineers get a remote "box" that hosts the full environment. The long-term vision was that engineers wouldn't have to interface with local tooling if they didn't want to -- e.g., "if I don't want Android tooling on my machine, I use a DevBox; if I want to build Go code, I use a DevBox."
- **The bridge to agents:** the same primitives built to onboard human engineers turned out to be exactly what's needed to onboard agents. The question becomes: *what happens if we give agents their own computers and let them work for longer periods of time?*

### Giving Agents Their Own Computers: New Working Model
- Handing agents dedicated compute and long time horizons is a fundamentally new working model that introduces three categories of challenges:
    1. **Security** -- agents operating with real access in real environments.
    2. **Infrastructure** -- running and supporting agent workloads at company scale.
    3. **LLM wrangling** -- getting models to do longer-horizon work reliably.
- Uber has begun experimenting aggressively in this space, with Minion as the central platform.

### The Core Insight Behind Minion: Toil, Not Complexity
- **The toil task archetype:** an engineer is working on a product feature when a ticket arrives -- say, "fix this massive vulnerability." The actual fix is minor, but:
    - They must reshuffle their schedule.
    - They must make the fix, test it, and validate it in production.
    - They're the only person who can do it because they own the fix.
    - **The validation loop, not the fix itself, is what consumes time.**
- As Uber's stack grew, this kind of work became *common engineering work*.
- **Minion's trade-off and bet:**
    1. Bet on a specific tool that can do a specific task.
    2. Bet that the tool can do that task *well*.
- The bet is explicitly **not** "the agent writes big features." Nikhil says he *cannot score* an agent on "write me a big feature." He *can* score it on: "I told you to change a line of code -- can you figure out whether you also need to run tests? Can you reason about the tree of dependencies that might be affected without me explicitly telling you?"
- **The win:** by backgrounding toil, when engineers reach the validation step, **50-70% of the required work has already been completed by the background agent.**

### How Minion Works in Practice
- Nikhil wrote a short internal doc proposing the idea, which led to founding a platform that:
    - Hosts the agent (Minion) plus a number of **specialist agents** inside Uber's **CI infrastructure**.
    - Accepts **prompts** as inputs describing the work.
    - Routes work through Uber's standard **CI checks** -- tests, quality checks, diff description, and all required metadata.
- **End goal:** remove the toil from the engineer entirely.

### "The Engineer Is a Sensitive Creature" -- Don't Promise the World
- A core principle for rolling out agentic platforms internally:
    - **Don't promise engineers the world.** Give them a *valuable use case*.
    - **Go to where the pain lives.** Prove the core value proposition on a real, painful category of toil.
    - Once engineers experience the value, they naturally understand that AI is not there to take away all their work -- just the parts that are tedious and require **rigor and tedium to execute**.

### Use Case Identification and Rollout: Satisfaction-Driven, Not Mandated
- Identifying the right use cases to roll out is one of the biggest challenges.
- **Two competing forces:**
    - The **industry's documentation** of AI product use cases.
    - The **internal conflict**: not everything applies, but a lot of it can, in different ways.
- **Common internal feedback:** *"I don't know what to choose. I don't even want to choose."*
- Uber treats this as a strong signal: **if it comes down to internal teams making a choice, the platform team defaults strictly to developer satisfaction and data.**
    - Satisfaction data covers areas like onboarding experience.
    - Signal examples: "toil work is taking away a lot of my time," "I barely get any time to do deep work," "code reviews are taking a lot of time."
- **Iterative co-design (not "Apple-style" dictate):**
    - Uber can't just ship a tool and say "use it this way."
    - Engineers push back: "No, we use it *this* way. You need to listen to what I'm saying."
    - There's an ongoing **negotiation** between platform team and engineers.
- **Bigger swings become possible once trust is established:**
    - Example: "code reviews are a bottleneck → let's build high-quality agentic reviews."
    - Example: "feature flag cleanup / small changes are a bottleneck → let's give you the ability to get out of that zone."
    - Then **measure directly**: is this actually helping?
- **Quote-worthy principle:** *"Most organizations working in this space are looking for people with ingenuity to act on boring feedback."*

### Feeling of Productivity vs. Actual Productivity
- Every quarter, the same pain points resurface: documentation is a pain point, toil is a pain point.
- **Key insight:** the *feeling* of productivity is as important as *actual* productivity. The two go hand-in-hand in building a productive engineering workforce.

### Shepherd: Large-Scale Change as a Primitive
- Beyond Minion, Uber invests heavily in **Shepherd**, a system for applying agents at scale to migrations, upgrades, and other mass changes.
- **The classic problem:** languages and frameworks (e.g., Java) need constant upgrading across many services, many of which are not actively developed.
- **Shepherd's philosophy:** treat the real work of "keep the codebase evergreen" as a first-class engineering concern -- **"evergreen engineering."**
- **The core capability:** give engineers a platform to **disseminate changes** at scale and provide the **mental model of a large change**.
- **Large-scale change is treated as a primitive**, supported by the platform.
- **Minion vs. Shepherd, contrasted:**
    - **Minion** = a primitive for **backgrounding toil**.
    - **Shepherd** = a primitive for **mass changes** -- including managing merge conflicts, keeping PRs up to date, and providing a **centralized view** of mass changes across the org.
- **Ultimate goal of Shepherd:** keep code healthy, keep code evergreen, and give people the primitive to do it.

### Culture: Builders, Not Mandates
- Organizations where engineers identify as **builders** and are **outcome-oriented** are moving faster toward agentic / AI-driven development.
- **Mandating AI use did not work** at Uber. People had to realize the value themselves; the tooling had to reach a sufficient level of capability before adoption followed.
- **What this tells us about engineers:** they are all builders. The reason Uber has 90% of the platform primitives that pre-date AI is precisely because **builders need to keep building and need to move fast**.
- **Examples:**
    - DevPods = a function of being able to **parallelize manually**.
    - DevContainers / remote dev environments = **parallelism without agents** -- a demand that existed well before AI.
- **Implication for the platform team:** platform engineers must consider whether they can "wear some of those hats" for builders -- i.e., proactively absorb the work that builders would otherwise scrutinize as a waste of time.

### How to Make Tools Stick: Safe Failure
- **Two failure modes for rollout:**
    1. *"Use this or else"* -- does not work.
    2. *"This tool can do X, Y, Z"* -- but in production it only delivers a quarter of X, a tenth of Y, a fiftieth of Z. Also fails.
- **What works:** engineers must feel that the platform team **sees the same pain they see**. That's what makes a builder "hungry."
- **The builder mindset:** builders want to take the tool and **mold it to their outcome** -- not be forced into a rigid expectation.
- **Why agents fit Uber's culture:** Uber's engineers are builders who need to finely configure and control what they do. Agents are a fit *because* they're moldable, not because they're forced.

### Parting Wisdom: The Mission Hasn't Changed, Only the Means
- **It's not too late.** Most people still don't have this figured out. There is room and runway.
- **The primitives have always been the same:**
    - Can I move my engineers faster?
    - Can I take away the pain they see daily so they can focus on building high-quality features?
- **The platform mission is unchanged.** Only the *means* used to solve the problem have shifted.
- **All investments may or may not pay off** -- prioritization matters -- but the primitives of **high-quality software delivery** and **rapid iteration** remain essential.
- Most tech companies will want rapid iteration. The job is to build the fundamentals and **give engineers a safe space to fail**.
- **Why safe failure matters:** when engineers fail expensively, it **erodes trust**. A platform must protect people from toil, give them room to fail safely, and let them focus on what they've always tried to do: build high-quality products and features for the business.

## Key Technical Concepts

- **Minion:** Uber's background agent platform. Accepts prompts, runs specialist agents inside Uber's CI, and is responsible for **>11% of all merged PRs** across the company. Targeted specifically at **toil tasks**, not feature authorship.
- **DevBox / DevPod:** Uber's containerized remote development environment. Originally built for environment isolation across Android/iOS/backend/frontend stacks. Became the foundational primitive for **onboarding both humans and agents** to the codebase.
- **Shepherd:** Uber's large-scale migration management system operating across hundreds of millions of lines of code. Manages mass changes, merge conflicts, PR freshness, and provides a centralized view of org-wide migrations. Treats **"large-scale change" as a first-class primitive**.
- **Toil Tasks:** Engineering work where the actual change is minor but the surrounding ceremony (testing, validation, deployment, ownership) consumes the bulk of the time. The canonical example is **CVE / vulnerability remediation**. This is Minion's primary target.
- **Stack-Ranked Backgrounding:** Minion's working model -- toil is pushed into the background so that **50-70% of the work is done by the time the engineer reaches validation**.
- **Satisfaction-Driven Platform Investment:** Use cases are chosen based on **developer satisfaction data and pain-point feedback**, not on industry hype. Engineers negotiate the direction of the platform rather than being told how to use it.
- **Evergreen Engineering:** Shepherd's framing -- the discipline of keeping code healthy, up-to-date, and continuously migrated, treated as a permanent fact of engineering rather than a one-off project.
- **Safe Failure:** A platform property that allows engineers to try tools and approaches without expensive failures that would erode trust in the platform.
- **Primitive-Oriented Platform Design:** Uber exposes capabilities (large-scale change, background toil, remote dev environments) as **primitives** that engineers compose, rather than as rigid end-to-end products.

## Practical Applications

- **Toil Automation at Scale:** Identify categories of work where the fix is trivial but the validation loop is expensive (CVEs, dependency bumps, small mechanical changes), and background them with Minion-style agents.
- **Agent Onboarding via Dev Environments:** Reuse existing remote dev environment infrastructure (DevPods/DevContainers) as the onboarding substrate for agents -- the same primitives that serve human onboarding.
- **Large-Scale Migrations:** Use a Shepherd-style system to manage framework/language upgrades across hundreds of services, including auto-resolving merge conflicts and keeping PRs fresh.
- **Satisfaction-Driven Roadmapping:** When internal users say "I don't know what to choose," default to **developer satisfaction data** to prioritize which agentic capabilities to invest in next.
- **Building Engineer Trust:** Ship narrow, valuable use cases rather than overpromising; let engineers mold tools to their outcomes; prove the platform sees the same pain they do.
- **Platform Primitive Design:** Expose each capability (toil backgrounding, mass change, isolated dev environments) as a composable primitive, not a closed product.

## Notable Quotes

> "Uber's background agent platform called Minion is responsible for generating over 11% of all merged PRs across Uber."

> "The actual fix is minor. It's actually the validation loop that takes a long time, and that's what I'm required for."

> "When they get to the validation, 50% to 70% of the work required has been completed by a background agent."

> "Don't promise an engineer the world when you give them a platform, give them a valuable use case because you have to go to where the pain lives."

> "Most organizations working in this space are really looking for people with ingenuity to act on boring feedback."

> "The feeling of productivity is as important as actual productivity."

> "Mandating the use of AI didn't really get an option where it needed to be. People had to realize for themselves. It had to reach a point of capability before people adopted it."

> "What I need is the engineer to be able to see that I know the pain they see. That's what gets a builder really hungry."

> "You want to build a platform that lets people have the space to fail and protect people from toil."

## Timestamps

| Time | Topic |
|------|-------|
| 0:00 | Introduction: Uber's compounding platform bets and the 11% PR stat |
| 0:55 | The origin of DevBox: environment isolation for multi-stack engineers |
| 2:00 | DevBox → DevPod: from onboarding humans to onboarding agents |
| 2:40 | Giving agents their own computers: security, infra, and LLM challenges |
| 3:10 | The inception of Minion and the toil-task framing |
| 4:10 | Why "write me a big feature" is the wrong scoreable unit |
| 5:00 | Minion architecture: agents inside CI, prompts in, diffs out |
| 5:30 | "The engineer is a sensitive creature" -- don't promise the world |
| 6:20 | Use case identification and the "I don't know what to choose" signal |
| 7:30 | Satisfaction-driven investment and the Apple contrast |
| 8:30 | Bigger swings: agentic code review and toil backgrounding |
| 9:10 | Feeling of productivity vs. actual productivity |
| 9:30 | Shepherd: large-scale migrations and "evergreen engineering" |
| 10:30 | Large-scale change as a first-class primitive; Minion vs. Shepherd |
| 11:20 | Builder culture and why mandates don't drive AI adoption |
| 12:10 | DevPods = manual parallelism; the pre-AI demand for primitives |
| 12:50 | Making tools stick: safe failure and shared pain |
| 13:30 | Parting wisdom: the mission is unchanged, only the means have shifted |

## Takeaways

- Uber's AI success is a **compounding payoff** of pre-AI platform primitives (DevPod, CI, Shepherd) -- not a sudden AI retrofit.
- **Toil, not complexity**, is the right target for background agents. Score on the scoreable (small fixes + correct dependency awareness), not on "write me a feature."
- By the time an engineer touches a task, **50-70% of the work should already be done** by the background agent.
- Expose capabilities as **primitives** (background toil, mass change, isolated dev environments) that engineers can compose, rather than as closed products.
- **Satisfaction data beats mandates.** Engineers adopt AI when capability meets real pain -- not when leadership forces it.
- The platform mission is unchanged: **move engineers faster, remove daily pain, protect space to fail safely.** Only the means (agents) are new.

## Speaker Links

- Talk source (YouTube): https://www.youtube.com/watch?v=o7G5EHDqQ1g
- Channel: Ona (https://www.youtube.com/@ona_hq)
