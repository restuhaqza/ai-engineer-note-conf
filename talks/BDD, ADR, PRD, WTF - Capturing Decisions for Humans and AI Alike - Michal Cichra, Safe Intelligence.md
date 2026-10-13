# BDD, ADR, PRD, WTF: Capturing Decisions for Humans and AI Alike

| Field | Details |
| --- | --- |
| **Speaker** | Michal Cichra |
| **Affiliation** | Safe Intelligence |
| **Type** | Talk |
| **Date** | 2026-06-03 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=504PvfXou5Y) |
| **Tags** | `architecture` `agent-engineering` `developer-workflows-and-testing` |

## Summary

Products accumulate rules and features whose original reasons get forgotten. People leave, memories fade, and an LLM's context gets compacted, so an agent cannot rely on remembering earlier work either. Eventually someone asks why a flow exists or why the code is shaped a certain way, and the founding engineer may not be around to answer.

Michal Cichra, a principal engineer at Safe Intelligence, argues that the rationale behind a product must be recoverable outside the person or session that produced it. He lays out four pieces that capture different kinds of intent: architectural decision records (why a technical choice exists), product requirements documents (why a feature exists), executable behavior scenarios (does the product actually behave as described), and a design system (a shared visual and usage vocabulary).

The key move is turning those documents into an enforcement loop. A coding agent trying to deliver a pull request must use Git, so Git hooks and CI checks catch rule violations and link the agent back to the decision document that explains the fix. Mechanical rules get automated so human review can focus on higher-level judgment.

## Key Takeaways

- **Context is lost on both sides.** People forget and leave; LLM context gets compacted and agents have no durable memory of earlier work.
- **Record intent, not just rules.** An **ADR** (Architecture Decision Record) captures why a technical decision exists and how to enforce it.
- **Keep PRDs lightweight.** A **PRD** (Product Requirements Document) only needs the problem, the goal, and the user journey that connects them.
- **Make specs executable.** Markdown describes intended behavior but cannot test it; **BDD** (Behavior-Driven Development) with Cucumber adds a readable, runnable layer.
- **Reuse a design system.** Document components, their appearance, and their usage rules, then compose larger patterns from smaller ones.
- **Enforce through the PR loop.** Git hooks plus CI run the same checks, so a skipped local hook still gets caught; local hooks alone can be bypassed.
- **Prevent bad dependencies entirely.** Import linting and database isolation stop whole classes of problems, like N+1 queries, rather than catching them case by case.
- **It is context-heavy but works.** Cichra reports using roughly half the context window for initial research yet surviving twenty to fifty compactions (a personal report, not a measured result).

## Detailed Notes

### Why do we still have this rule? · 00:22
- Cichra draws on time at Microsoft, Red Hat, and a decade on one product; his team at Safe Intelligence had just released an agent-testing product.
- He opens with the five-monkeys story: monkeys are punished when one climbs for bananas, and the prohibition persists through replacements who never felt the punishment. He himself questions whether the story is an urban legend — its value is as an analogy.
- A rule can outlive its rationale. Humans forget and leave; LLM context compacts and agents have no memory.
- Later the team asks why a flow exists, what a feature solves, or where a change belongs — and the founding engineer may be gone.
- AI can make this familiar problem arrive sooner. The rationale needs to stay recoverable outside the person or session that created it.

### Record the architectural reason and the enforcement path · 02:03
- An **ADR** (Architecture Decision Record) records why a choice exists and how the team intends to enforce it, with references and code snippets.
- Example: code is split into layers to prevent N+1 queries (repeated database calls). Import linting restricts which modules may depend on which others.
- Database reads return plain data shapes instead of ORM objects (ORM = a library mapping code objects to database rows), so downstream code cannot quietly trigger more queries.
- The boundary controls both where database access happens and what crosses into the rest of the app; Cichra ties this to avoiding duplication.
- There is no mandatory ADR format. A text document cannot enforce itself — a separate tool must detect the violation and explain which rule broke, why it exists, and how to fix it.
- Recording the affected files, folders, or language scope helps connect a failure to the right decision; Cichra says roughly fifty more ADRs define the product's architecture.

### Keep the product purpose lightweight · 03:26
- A **PRD** (Product Requirements Document) preserves a different kind of intent: why a feature exists, the problem it solves, and how the user moves through the app to use it.
- It can be short. The useful link is between the problem, the desired outcome, and the journey connecting them.
- It helps an agent implementing the feature and equally helps the person returning to it six weeks later.

### Make the behavior specification executable · 04:07
- A written spec leaves one question open: how do you know the running product behaves as described? Markdown describes behavior but does not test it.
- Reviewing AI-written tests is not a comfortable substitute — Cichra finds them even harder to read than AI-written code.
- **BDD** (Behavior-Driven Development) supplies an intermediate layer describing behavior in human language. Cucumber makes that layer executable.
- A Gherkin scenario can express a profile-saving journey — signed in, change a display name, save, reopen, and expect the new name — without exposing test plumbing.
- Each step still needs an executable implementation; the scenario alone does not run the test.
- Scenarios can connect to PRDs and critical user journeys, and may refer back to the documents explaining why the behavior exists.

### Give agents a visual vocabulary they can reuse · 06:00
- Consistent interfaces need the same treatment as code. A **design system** and pattern library are the shared vocabulary — as true now as before agents.
- Separate appearance from usage. "A primary button is blue, this shape, this size" is one decision; "only one primary button visible per page" is another. Both must be explicit.
- Turn decisions into reusable components: document colors, variants, and states, and provide previews and snippets that humans and agents can inspect.
- Review previews against the visual principles before reuse.
- Build larger patterns by composing smaller pieces, as with code — otherwise each new screen invents another competing convention.

### Put feedback on the path to a pull request · 07:18
- Documents and components still need an operational answer: how do people and agents follow them? Cichra's answer is a repeating work-and-feedback loop.
- The harness combines Git hooks, skills, CI, linters, and other checks. An agent tasked with delivering a pull request must use Git, giving a natural trigger point.
- Hooks run predefined tasks locally; CI runs the same tasks remotely. That duplication matters because local hooks can be bypassed, for example with `--no-verify`.
- Checks cover linting, formatting, type checking, code duplication, architectural boundaries, and document linting.
- Automate mechanical rules so review can focus on higher-level decisions. Tabs, spaces, and settled conventions no longer deserve discussion in every review.

### Prevent forbidden dependencies, then explain the repair · 08:43
- Architecture checks make abstract layering concrete by controlling module imports.
- In the end-to-end BDD suite, tests cannot access the database or import any module that can — forcing the suite to exercise the app through browser behavior rather than reaching behind the interface.
- In the product, rendering templates cannot talk to the database. Combined with plain data instead of ORM objects, this removes a route to per-item extra queries.
- Cichra describes the architecture as preventing N+1 queries entirely; the generalizable mechanism is narrower — blocking database access from rendering blocks that class of implicit query.
- The aim is to make an unwanted dependency impossible rather than finding its consequences repeatedly in review.
- The repair path becomes concrete: the change is rejected, the failure links to the decision document, the agent reads the rationale, corrects the change, and tries again.

### Keep the loop, change its focus · 09:50
- The cycle of work, feedback, and revision is the same across product, UI, and backend tasks. What changes is which information and checks are most useful.
- **Skills** supply that focus:
  - **ADR and PRD work:** guides the agent to decision records and the code they affect.
  - **UI work:** skips some checks to prioritize quick browser feedback.
  - **Testing:** uses code coverage and changed files to select only the relevant tests instead of the whole suite.
  - **Goal execution:** retains the model's decisions so they can be reviewed later.
- These skills specialize the work inside the loop without replacing its structure.

### Make recovery possible after context compaction · 11:04
- The approach has a cost: retrieving and understanding documents consumes context. Cichra reports initial research can use roughly half the context window.
- He still describes it as having worked over the preceding half year, reporting twenty or fifty compactions in his own sessions without losing the ability to continue — personal experience, not measured reliability.
- The target is a multi-hour session where an agent pursues a clear goal autonomously within explicit rules.
- That needs more than keeping every detail in the current conversation; it needs a way to recover the details that matter once they are gone.
- Decision records preserve why architectural choices were made; product documents preserve why features exist; executable scenarios tie intent to behavior; design systems extend the discipline to interfaces (including rules like no inline styles outside the system). The harness connects all of it to the work.

## Resources

- [Cucumber step definitions](https://cucumber.io/docs/cucumber/step-definitions/) — how readable Gherkin steps map to executable code.
- [Git hooks reference](https://git-scm.com/docs/githooks) — checks triggered during commits and pushes, including rejection behavior.
