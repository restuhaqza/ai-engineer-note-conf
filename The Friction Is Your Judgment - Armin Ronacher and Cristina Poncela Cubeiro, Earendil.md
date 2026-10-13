# The Friction Is Your Judgment

| Field | Details |
| --- | --- |
| **Speaker** | Armin Ronacher, Cristina Poncela Cubeiro |
| **Affiliation** | Earendil |
| **Type** | Keynote |
| **Date** | 2026-04-18 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=_Zcw_sVF6hU) |
| **Tags** | `architecture` `coding-and-developer-tools` `rag-context-and-search` |

## Summary

Armin Ronacher (creator of the Flask Python framework) and Cristina Poncela
Cubeiro work together at Earendil. They argue that AI coding tools are genuinely
useful but can quietly erode human judgment. When code is cheap to generate,
teams ship more than they can carefully review, and rare human decisions get
skipped.

The talk separates two problems: a *psychological* one and an *engineering* one.
Psychologically, the next prompt is addictive and a stream of output feels like
progress. Engineering-wise, agents are optimized to make code run and unblock
themselves, so they hide failures and can make a codebase harder to understand.

Their answer is not to reject agents. It is to design the codebase so agents can
work in it well, add mechanical checks that catch bad patterns, and deliberately
keep the few points of resistance where human judgment actually matters.
Ronacher's closing image is physical: steering needs friction.

## Key Takeaways

- **Frictionless becomes the baseline.** Time saved by AI tools gets refilled
  with more expected output, leaving less room for review.
- **Agents hide failures to make progress.** A config loader that falls back to
  defaults can let the app run on settings nobody intended.
- **Generation outpaces review.** Every engineer produces more code than they can
  inspect, so pull requests pile up and reviews get rubber-stamped.
- **Libraries are easier for agents than products.** A small, bounded API surface
  fits in context; a whole product with permissions, flags, and billing does not.
- **Design the codebase as agent infrastructure.** One SQL interface, one UI
  primitives library, no dynamic imports, unique function names.
- **Make rules mechanical.** Linting rules catch bad patterns without relying on
  a reviewer's memory.
- **Not all findings are equal.** Route mechanical bugs back to the agent; flag
  migrations and permission changes for a human.
- **Keep useful friction.** SLOs (service-level objectives) and error budgets are
  deliberate resistance that matters more, not less, with fast agents.

## Detailed Notes

### Shipping without friction · 00:23
- The talk opens on a security incident where an accidentally deployed config
  change caused an outage. The company's own tagline under the headline read
  "ship without friction."
- The question they pose: which obstacles should disappear, and which make us
  stop before a costly mistake?
- Ronacher has 20 years in software and created Flask. Poncela Cubeiro calls
  herself a "native AI engineer" because the tools existed before her career.
- They present an evolving practice, not a settled solution.

### From free time to production pressure · 03:29
- Early AI coding felt fun and productive: extra time and better work.
- As adoption spread, that advantage became an expectation. Everyone is assumed
  to use the tools and to ship faster.
- The trap has two parts. First, the next prompt has an uncertain payoff, so you
  keep trying. Second, a high volume of visible output feels like efficiency.
- **The human must keep the agency to interrupt the loop.** The agent will happily
  keep reading files it never needed.

### Creation outpaces review · 05:29
- When writing code was the bottleneck, writing and review capacity were roughly
  matched. Agents break that balance.
- More people now contribute code: marketing staff, and former engineers who are
  now CEOs. Responsibility does not spread the same way.
- Engineering teams still own the result. Machines cannot be responsible for a
  change.
- A hypothetical 5,000-line pull request shows the mismatch: it needs the most
  judgment exactly when it makes disengagement most tempting.

### Runnable code can hide a broken system · 07:13
- Agents optimize for immediate progress: write code, pass tests, unblock.
- A config reader that catches a failed read and returns defaults looks like
  recovery, but the app may now run on unintended settings.
- Ronacher's example: a loader that preserves the failure makes the missing or
  invalid config visible before any writes happen.
- **Local workarounds compound.** Each one adds complexity, until the agent can
  no longer navigate the codebase and starts re-implementing things.
- Humans feel discomfort when writing this kind of code; agents add no emotional
  brake. That concern must enter through people and explicit constraints.

### Make the codebase legible to agents · 09:11
- Libraries tend to work well with agents: a defined problem, a constrained API
  surface, and a simple extensible core.
- Products combine interacting concerns (UI, API, permissions, feature flags,
  billing) that exceed the agent's context window.
- An edit can be reasonable in one file and incoherent across the whole app.
- Modular components help, but so does modularizing the *execution flow*. In an
  assistant refactor, three steps were clear: receive the message, run the agent
  loop, handle the output.
- Trouble accumulates *between* those steps, as the agent adds type conversions
  and stuffing into shared state.
- Follow established patterns (models learned them), keep the core simple, and
  avoid abstractions that hide intent, such as ORMs versus raw SQL.

### Turn design rules into mechanical checks · 12:32
- At Earendil, most of this discipline is enforced with linting rules, not
  reviewer memory. One rule bans bare catch-alls.
- **One SQL query interface** so the agent does not miss a site that also needs
  changing.
- **One UI primitives library** so styling and behavior stay consistent.
- **No dynamic imports.**
- **Unique function names**, so a grep returns one unambiguous result and the
  agent spends less context disambiguating.
- They are also trying TypeScript's `erasableSyntaxOnly` mode: code stays
  JavaScript plus erasable type annotations, so there is less transpile-time
  indirection when locating errors.

### Separate fixable bugs from human decisions · 14:11
- They built a review extension for the Pi coding agent that sorts findings into
  two groups.
- Mechanical bugs and clear `AGENTS.md` violations can go back to the agent to
  fix automatically.
- Other changes need an explicit human call-out:
  - **Database migrations** depend on locks and production data size.
  - **Permission changes** often rely on under-documented authorization rules.
- The current human call-outs are non-blocking attention signals, not an enforced
  approval gate. A new dependency raises questions a machine cannot answer.

### Where speed helps — and where to keep friction · 16:00
- Good uses of agents: turning a customer report into a reproduction case, and
  exploring product directions (if you own the resulting code).
- Weaker uses: system architecture and reliability, where interactions and
  consequences matter more than a first working version.
- Ronacher warns that months of technical debt can pile up in days or weeks. He
  found broken production code and realized he had committed it himself with an
  agent's help.
- **Some friction is deliberate.** SLOs (service-level objectives) and their
  error-budget policies force teams to ask whether a service is reliable and
  staffed enough to operate.
- Steering requires friction. Useful friction is where human judgment enters.

## Resources

- [Flask documentation](https://flask.palletsprojects.com/en/stable/)
- [TypeScript's `erasableSyntaxOnly` option](https://www.typescriptlang.org/tsconfig/erasableSyntaxOnly.html)
- [Pi review extension](https://github.com/mitsuhiko/agent-stuff/blob/main/extensions/review.ts)
- [Pi coding agent](https://pi.dev/)
