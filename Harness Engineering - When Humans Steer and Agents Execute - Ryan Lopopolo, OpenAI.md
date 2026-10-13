# Harness Engineering: When Humans Steer and Agents Execute

| Field | Details |
| --- | --- |
| **Speaker** | Ryan Lopopolo |
| **Affiliation** | OpenAI |
| **Type** | Talk |
| **Date** | 2026-04-17 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=am_oeAoUhew) |
| **Tags** | `developer-workflows-and-testing` `safety-and-governance` `rag-context-and-search` `agent-engineering` |

## Summary

Ryan Lopopolo, a member of technical staff at OpenAI, argues that implementation
is no longer the bottleneck in software engineering. When agents can write,
refactor, maintain, and delete code, the scarce resources become human time,
human and model attention, and the model's context window. The engineer's job
shifts from typing code to designing systems and delegating work.

His name for this shift is **harness engineering**: building the documentation,
guardrails, repository structure, and feedback loops that let agents produce
acceptable software reliably. Humans steer; agents execute. The session is part
keynote and part on-stage interview, so it mixes principles with concrete
mechanics.

Two ideas anchor the practice. First, quality standards must be made legible to
agents — written down as documentation, lint rules, structural tests, and reviewer
agents — instead of living as tacit knowledge. Second, context should arrive just
in time: front-loading every rule overwhelms the agent, so requirements are
surfaced through review comments, test failures, and skills as the work unfolds.

He also treats the repository as part of the prompt. Package boundaries,
consistent patterns, and shared utilities reduce the context an agent must load
and cut merge conflicts. His closing image is the LLM as a "fuzzy compiler": the
surrounding structure decides which generated code is acceptable, so durable value
sits in the human-defined constraints, not in any single generated implementation.

## Key Takeaways

- **Implementation is no longer scarce.** Human time, attention, and the context
  window are the real constraints; organize around delegation.
- **Write down what "good" means.** Documentation, ADRs (architecture decision
  records), tickets, and persona notes turn tacit standards into agent-readable
  context.
- **Turn repeated feedback into guardrails.** Recurring review complaints become
  lint rules, structural tests, and specialized reviewer agents.
- **Deliver context just in time.** Don't front-load every rule; surface
  requirements through review, lint, and test feedback as work progresses.
- **Treat repository layout as a prompt.** Clear package boundaries and consistent
  patterns save context and reduce merge conflicts.
- **Start with tests and bottlenecks.** Have agents test existing behavior, then
  automate the places where humans wait.
- **Review for acceptance, not perfection.** Requiring every comment to be
  addressed can bully the agent and chase minutiae instead of shipping.
- **Every manual "continue" is a harness failure.** If the agent needs a human to
  click yes, the context or task setup is incomplete.

## Detailed Notes

### The scarce resource is no longer implementation · 01:54
- After nine months of building software exclusively with agents, Lopopolo says
  the way we build has changed fundamentally.
- Code is "free": abundant, cheap to refactor, and no longer the constraint.
  Producing, maintaining, and deleting code no longer forces how teams allocate
  effort.
- The new scarce resources are human time, human and model attention, and the
  model context window.
- Each engineer effectively gets access to many agents' worth of capacity, limited
  mainly by GPU capacity and token budgets.
- Work that used to be permanently deprioritized can now be started several times
  in parallel, and the best result kept.
- Example: internal tools can ship with good localization and internationalization
  from day one, without trading against other work.
- The role becomes staff-engineer-like: decide which problems matter, define
  acceptable software, and design structures that let people and agents execute.

### Make quality legible and enforceable · 06:56
- Agents cannot reproduce standards that exist only in people's heads.
- The context that shaped a codebase — docs, ADRs, historical tickets, code
  reviews, descriptions of good work — should be written where agents can read it.
- A single patch involves hundreds of small, under-specified non-functional
  decisions; the job is to specify the ones that matter.
- Durable guardrails include security and reliability reviewer agents in CI
  (continuous integration — automated checks on every change), repo-specific lint
  rules, and source-level tests.
- Example rules: require retries and timeouts around network calls, keep files
  under 350 lines so they fit context better, and enforce package privacy and
  dependency edges.
- Error messages should state the expected pattern and the remediation, so both
  humans and agents know the next step.
- This can slow short-term delivery, but the goal is to eliminate a whole class of
  failure once, then step back.
- Each specialist on a team can encode their expertise one time, so every agent
  trajectory benefits from it.

### Deliver the right context at the right moment · 11:21
- A good harness is not a replacement for the coding agent; its core job is
  **just-in-time context delivery**.
- AGENTS.md (a repository file of agent instructions) and always-on rules have
  limits: auto-compaction and long sessions eventually page earlier instructions
  out of context.
- Skills and CI reviewers can refresh the relevant guidance throughout a task.
- React example: let the agent prototype the UI first, then use lint or tests to
  require smaller, more stateless components with local dependencies instead of
  prop drilling.
- The agent then revises its existing patch to satisfy the newly surfaced
  constraints — exploration first, standards enforced at the end.
- The workflow starts from a ticket plus a few skills that let Codex launch the
  app, start local observability, and attach browser tooling.
- He keeps leverage concentrated in roughly five to ten skills, improving them
  rather than spreading wide, so changing infrastructure stays hidden.
- He favors first-party harnesses such as Codex because their tools and invocation
  conventions are part of model post-training, while his team focuses on steering
  observable behavior.

### Design the repository and review process for agent throughput · 21:26
- His repo grew from a single-package app into a PNPM (Node package manager)
  workspace with about 750 packages, split by business domain and stack layer.
- The goal: concrete filesystem boundaries, enforceable package privacy, and
  changes that fit within a small subtree.
- Custom ESLint rules and structural checks enforce shared utilities, deduplicated
  Zod schemas (a data-validation library), and canonical helpers.
- Consistency matters because it reduces the fresh context an agent must load: one
  ORM, one language, one CI-script style, one way to add lint rules.
- Large-scale migrations become worthwhile because uniform patterns are easier for
  agents to predict and repeat across the repo.
- High throughput broke their review process: three to five PRs (pull requests)
  per engineer per day caused painful merge conflicts and long-open PRs.
- Fix: separate code more clearly, shorten PR lifetimes, and hold a weekly
  "garbage collection day" to convert recurring complaints into docs, checks, and
  reviewer agents.
- Persona-specific reviewers (frontend architect, reliability engineer,
  scalability) run on every push and surface serious issues.
- Review should not become an obstacle course: implementation agents may
  acknowledge, defer, or reject feedback, and the bias is toward accepting useful
  work, not chasing every nit.

### Start with confidence, then expand beyond code generation · 30:28
- Entry point one: have agents write tests for existing behavior. This raises
  confidence and makes the codebase easier for agents to navigate safely.
- Entry point two: find where human time goes — manual coding, slow CI, flaky
  tests, waiting for review — and automate those bottlenecks step by step.
- The high-leverage work is defining, prioritizing, sequencing, and delegating,
  not personally executing everything.
- His token use is roughly a third planning/tickets/docs, a third implementation,
  and a third CI.
- He is wary of unread plans: approving a plan you never read locks in
  instructions you may not want followed.
- If a plan governs execution, push it as its own PR for real human review;
  otherwise a well-specified ticket should let the agent proceed directly.
- QA (quality assurance) was initially weak because agents lacked tools and docs
  to download, launch, and inspect built artifacts; new lifecycle stages each
  start from zero.
- The vision extends to feedback triage, incident handling, preventing PII
  (personally identifiable information) leaks in logs, support runbooks, and
  critical user journeys.
- Mental model: the **LLM as a fuzzy compiler**. Specs, tests, and constraints
  decide which generated implementations are acceptable; swapping models is like
  changing a compiler backend.
