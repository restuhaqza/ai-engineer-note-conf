# Comprehend First, Code Later

| Field | Details |
| --- | --- |
| **Speaker** | Priscila Andre de Oliveira |
| **Affiliation** | Sentry |
| **Type** | Expo session |
| **Date** | 2026-05-27 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=li0SaBt9RDM) |
| **Tags** | `developer-workflows-and-testing` `coding-and-developer-tools` `reasoning-and-models` |

## Summary

Priscila Andre de Oliveira is a senior software engineer at Sentry who now spends her days directing AI agents rather than writing code by hand. Her talk makes a counter-intuitive claim: the biggest everyday win from AI is understanding code, not generating it.

She describes Sentry's large, fast-moving codebase — more than 15 years of code, roughly 100 merged pull requests a day, and many contributors — where simply recovering context is a constant task. She cites a study estimate that a large share of developer time goes into reading and navigating code, and argues AI is well suited to speeding that up.

Her own tracked usage backs this up. After asking Claude to classify her recent sessions, she found that 67% of her prompts were about comprehension and only 2% about generating code. That pattern led her to build a reusable skill called **Catch Me Up**.

Her main recommendation is to put a human comprehension step between an agent's research and its plan. Understanding the agent's findings lets you catch a wrong direction before it turns into code.

## Key Takeaways

- **Comprehension beats generation.** In her own tracked usage, 67% of prompts were about understanding code and only 2% about writing it.
- **The fastest AI win in a big codebase is recovering context** you would otherwise dig up manually or wait on a colleague for.
- **Turn repeated questions into a reusable skill.** Catch Me Up is just a Markdown prompt with clear goals and a preferred output format.
- **Ask for visual answers** — diagrams, flows, or tables — if that is how you think.
- **Add a comprehension checkpoint between research and planning** so you can spot a mistaken premise before implementation.
- **Check the agent's findings yourself.** A plausible explanation can carry a bad assumption straight into the code.
- **AI is a tireless teammate** for the questions you might hesitate to ask another person.
- **Don't ship code you don't understand** into the codebase that pays your salary.

## Detailed Notes

### Agent manager, not just an engineer · 00:28
- Priscila is a senior software engineer at Sentry, based in Vienna; she is also a maintainer of Verdaccio (an open-source npm registry) and a ViennaJS co-organizer.
- She jokes she promoted herself to "agent manager" — no pay raise, but the agents never complain.
- Since December 2025 she has worked mainly by prompting rather than typing code; even the slide deck was built by a skill.
- Her merged work with Claude covers bug fixes, features, refactors, and changes across repositories.
- Generating code is now part of the job, but she still owns responsibility for whatever lands in the codebase.

### The codebase that pays the salary · 02:52
- Sentry started as error and performance monitoring and grew into a full observability platform (metrics, profiling, and tools for monitoring agent applications).
- The codebase is large and long-lived: founded in 2010, 15+ years of code, around 400 employees worldwide, and 100k organizations depending on it.
- Because her salary depends on it working, she has a personal reason to care about what agents produce.
- A company hackathon produced internal AI tools now used daily:
  - **Abacos** — tracks internal AI usage.
  - **Warden** — an agent that reviews code in pull requests.
  - **Junior** — a Slack bot that reads a thread, investigates a reported bug or UI change, and opens a corrective pull request.
- The AI SDK integration test repository predates the hackathon; her team asked her to contribute only by prompting until the result was good enough.

### Keeping up with a moving codebase · 05:48
- Sentry ran a three-month "quality quarter" to cut technical debt: removing TypeScript `any` types, clearing stale TODOs, simplifying code, and deleting unused feature flags.
- The codebase keeps changing: about 100 merged PRs a day, four offices, outside contributors (Sentry is fair-source), deprecated components, and new lint rules.
- After six-plus years she can return from vacation to a conflict-filled PR — and fixing it needs more than merging lines; she must catch up on decisions and conventions that changed while she was away.
- She cites a study estimate that around 70% of developer time goes to reading and navigating code. The exact figure depends on the study, but the point holds: understanding is already a big part of the work.
- AI's role here is to make that understanding faster.

### Recover context before directing a change · 08:01
- Telling an agent to "explore, then change" leaves a gap: the agent may have misunderstood what it found, and you need enough understanding to steer it.
- Two before-and-after examples from her workflow:
  - "Where did this regression start?" — before: open GitHub and follow `git blame`; now: just ask for the regression's history.
  - "Why did this product behavior change?" — before: message a colleague (possibly in another time zone, with an answer the next day); now: ask for the decision context.
- Context that used to take manual digging or waiting now comes back in seconds.
- The productivity gain she emphasizes is reaching an informed understanding sooner, not writing code faster.

### Turn recurring questions into a skill · 09:37
- She had Claude analyze a local cache of 116 sessions and classify them into six buckets: comprehension, modification, process, review, generation, and other.
- The result: 67% comprehension and 2% generation. (These are her classified prompts, not a share of working time.)
- The repeated questions became **Catch Me Up**, a skill stored as a plain Markdown file containing a detailed natural-language prompt.
- It organizes exploration into six modes: Architecture, Convention, Feature trace, Syntax / API, Testing, and History.
- It is invoked with `/catch-me-up`. Because she is visually oriented, the skill asks for a diagram, flow, or table instead of only paragraphs.

### Does the test simulate an envelope? · 12:05
- The demo uses the unfamiliar AI SDK testing repository, where she is a new contributor, with Claude Opus.
- Her prompt: "I am a new contributor. Catch me up on how this repository works, and clarify whether it simulates a Sentry envelope and intercepts it during tests."
- The answer returns a summary and a flow diagram — and corrects her premise: the tests intercept real envelopes rather than simulate them (it spawns a collector).
- This changed what she understood before contributing; without it she would have had to establish the same facts herself.
- She draws a line between experimenting on a vibe-coded side project and shipping work code she does not understand.
- She also uses the skill during PR review, when her existing context is not enough to approve a colleague's change.

### Put comprehension before planning · 14:01
- She references Jake Nations' "Vibe Coding Our Way to Disaster," which proposes research → planning → implementation (similar to Claude Code's planning mode).
- She agrees but adds a missing human step: understand the research before you authorize the plan.
- Her four-step loop:
  1. **Research** — the agent explores the relevant code and context.
  2. **Comprehend** — read the findings, build your own understanding, and redirect or ask for more investigation where context is missing.
  3. **Plan** — once understanding is sound, ask the agent to propose the change.
  4. **Implement** — proceed with the agreed direction.
- The comprehension step can send work back to research; its purpose is to catch a wrong direction before it becomes code.
- Armin Ronacher's "The Final Bottleneck" raises the same worry — engineers who no longer know what is in their own codebase — which she treats as a real risk.
- Closing advice: track your own AI usage to spot reusable skills, align your mental model before prompting, and "ship kino code" rather than slop. She notes the talk is sponsored by Sentry.

## Resources

- [Vibe Coding Our Way to Disaster](https://www.arthropod.software/p/vibe-coding-our-way-to-disaster) — Jake Nations on deliberate research and implementation.
- [The Final Bottleneck](https://lucumr.pocoo.org/2026/2/13/the-final-bottleneck/) — Armin Ronacher on review bottlenecks and codebase understanding.
- "Measuring Program Comprehension: A Large-Scale Field Study with Professionals" — the paper behind the comprehension-time estimate.
- [Sentry AI SDK Integration Testing](https://github.com/getsentry/testing-ai-sdk-integrations) — Sentry's public test repository used in the demo.
- [Warden](https://github.com/getsentry/warden) — agent-based code review for pull requests.
- [Junior](https://github.com/getsentry/junior) — a Slack bot for investigating issues and taking action.
