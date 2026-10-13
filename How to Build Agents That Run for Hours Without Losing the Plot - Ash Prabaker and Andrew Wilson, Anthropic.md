# How to Build Agents That Run for Hours Without Losing the Plot

| Field | Details |
| --- | --- |
| **Speaker** | Ash Prabaker, Andrew Wilson |
| **Affiliation** | Anthropic |
| **Type** | Workshop |
| **Date** | 2026-05-18 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=mR-WAvEPRwE) |
| **Tags** | `evals` `coding-and-developer-tools` `rag-context-and-search` `agent-engineering` `architecture` `apis-mcp-and-protocols` |

## Summary

Demos show an agent building a whole app from a single prompt, but they hide the
**harness** — the machinery that keeps the agent moving, preserves its progress, and
checks whether the result actually works. Anthropic Applied AI engineers Andrew
Wilson and Ash Prabaker unpack that machinery, first tracing how Claude Code and the
models evolved together.

They describe three ways long runs break down: **context** (amnesia, drift, and
"context anxiety" near the limit), **planning** (attempting too much at once), and
**self-judgment** (accepting a half-built feature). The rest of the session covers
experimental harness patterns: a planner that sets a high-level spec, a generator
that builds, and a separate evaluator that actually uses the app.

A recurring theme is that the harness is a moving target. As models improve,
scaffolding that was once essential — like resetting context between sessions —
should be removed and tested again, not kept out of habit.

## Key Takeaways

- **Demos hide the harness.** The interesting engineering is the machinery around the model, not the model call.
- **Name the three failure modes:** context, planning, and self-judgment.
- **Self-evaluation is a trap.** An agent asked to review its own work tends to approve it.
- **Use an adversarial evaluator with a separate context.** Tuning a critic to be demanding is easier than making a builder self-critical.
- **Give the evaluator a different job.** Have it open the live app with Playwright, click around, and critique — not just read diffs.
- **Write down "good" as a rubric.** The team grades design, originality, craft, and functionality, weighted toward design and originality.
- **Negotiate the definition of done.** Generator and evaluator agree on testable acceptance criteria before any code is written.
- **Remove scaffolding when behavior improves** — and read the traces to decide what to keep or delete.

## Detailed Notes

### Why long-running agents fail · 00:15
- A company one-shots a browser over five or six hours; the impressive part is the harness, which is rarely shown.
- Wilson recalls Boris Cherny's anniversary note: early Claude Code struggled with Bash and string escaping and ran about 20 minutes; much of Claude Code is now written by Claude Code over days.
- Three failure buckets:
  - **Context:** a fresh session forgets; within a session, coherence degrades ("context rot"); near the limit, the model may rush to finish ("context anxiety").
  - **Planning:** the agent tries too much, leaves features half-built, or runs out of context before a usable stop.
  - **Self-judgment:** it accepts an incomplete result — a button looks finished even though no backend exists behind it.

### Models and harnesses improve together · 04:14
- Two routes to improvement: more capability in the model weights, or better scaffolding around the model.
- METR's task-completion time horizons rise from about 1 hour (Sonnet 3.7) to 12 hours (Opus 4.6) at 50% success with a minimal scaffold — a measure of task difficulty, not agent runtime.
- The **Claude Agent SDK** supplies the core loop: Claude decides, calls tools, receives results, and continues — around it sit MCP connections, sub-agents, project instructions (`CLAUDE.md`), skills, slash commands, and permissions.
- History is co-evolution: Sonnet 3.5 plus artifacts made coding and visual iteration useful; computer use added screenshots and clicks; MCP standardized tool connections.
- February 2025: Sonnet 3.7 with strong SWE-bench positioning, and Claude Code in research preview — partly to learn how developers work.
- May: Opus 4 and Sonnet 4 improved context management and task completion; Claude Code went GA and its SDK exposed the harness.

### Loops, context policy, and exits · 07:55
- Geoffrey Huntley's **Ralph** technique (July 2025, widely noticed around December) repeatedly feeds a prompt to a coding agent.
- The useful version includes planning, feature breakdown, and one task per fresh session; predictable failure is inspectable.
- The Claude Code Ralph plugin differs in its design:
  - Fresh-session Ralph loop → new context for the selected task, then launch another session.
  - Claude Code plugin → one session with compaction, with a stop hook intercepting termination.
- The plugin uses a maximum iteration count and a completion promise as exit conditions.
- Repetition, context renewal, and stopping are separate design decisions, even when packaged as one loop.

### Persistent state turns a vague request into recoverable progress · 12:05
- The November pattern starts from a vague request (build a browser, a Slack clone, a Salesforce clone).
- An **initializer** creates durable artifacts: `feature_list.json`, a progress file, a Git repo, an init script, and pass/fail flags.
- JSON was preferred over Markdown because the models were less likely to overwrite it (though the format itself does not prevent destructive edits).
- Each coding session follows a bounded procedure:
  1. Establish the working directory and read the progress record.
  2. Run the init script and a smoke test so the server starts from a known working state.
  3. Select exactly one feature whose tests do not pass.
  4. Implement it and exercise the application with Puppeteer.
  5. If verification succeeds, commit and mark the feature as passing.
  6. If features remain, begin another session with fresh context.
- A feature record can make a pending browser check explicit (send a message, reload, verify it persists) with `"passes": false`; a rendered message box alone would not satisfy the checks.

### The generator–evaluator split · 17:50
- Problem: ask an agent to review its own pull request and it often approves work that needs another pass.
- The analogy is GANs (generative adversarial networks): a generator builds while a separate evaluator applies pressure (the loop borrows the roles, not the training).
- Generator and evaluator have separate contexts, system prompts, and responsibilities.
- The evaluator does more than read diffs: it opens the live application with Playwright, navigates, clicks controls, tries workflows, and sends back a critique.
- Why would another LLM be less generous? It still has biases, but tuning a dedicated critic to be demanding is tractable, while making a builder reliably self-critical is hard.
- The harness exploits the gap between being able to criticize and being able to construct.

### Define quality and negotiate done · 20:42
- Make desired qualities explicit so a critic can apply them.
- The rubric has four dimensions: **design, originality, craft, and functionality**, weighted toward design and originality when functionality is already solid.
- Reference sites and short examples calibrate taste, including what counts against generic purple gradients.
- Cycle: generator builds a page → evaluator launches Playwright, navigates, screenshots, scores, critiques → generator revises.
- HTML/CSS-only examples ran for about four hours over 5–15 rounds.
- If originality stays low, the builder can discard the design and restart — unlike a single generation or a loop patching the same approach.
- A **planner** expands a one-line request into high-level workflows and sprints, deliberately not a detailed technical blueprint, since an early technical mistake could cascade through every later sprint.
- Before implementation, builder and evaluator negotiate an **acceptance contract**:
  1. Generator proposes the feature and the tests that show success.
  2. Evaluator challenges excessive scope, weak tests, and missing edge cases.
  3. They exchange Markdown files until they agree.
  4. Generator builds; evaluator grades against the agreement.
- This converts a broad user story into testable assertions without over-specifying upfront, and gives the plan an opposing reader (the Ralph loop's fixed `plan.md` had none).

### A game maker that looks finished — and one that plays · 26:46
- Prompt "Build a retro game maker" run with the same model solo versus with the full harness; Prabaker calls the harness an expensive experiment, not an efficient default.
- Solo: the sprite editor looks plausible, but in play mode arrow keys and space do nothing — the agent knew the *look* of a game maker without making a playable game.
- Harness run: about 6 hours and about $200. It named itself **Retro Forge** and added a new-project dialog and a fuller canvas — product choices that came from the planner, not the prompt.
- A vague planner line about AI features became a full in-app level assistant.
- Play mode had a debug HUD, a running physics loop, a moving player, and collisions — because the evaluator launched the game and tried to play it.
- The evaluator caught ordinary integration bugs: FastAPI route ordering that passed unit tests but could break in production, and a Boolean-logic error in the delete key.
- The level-editor sprint reached 27 acceptance criteria; granular criteria make findings actionable ("fix this exact line"), while vague criteria yield vague critiques.
- Honesty note: out of the box Claude was a poor general QA agent — it found bugs and deferred them. The real work was reading traces and tuning prompts where the agent's judgment diverged from a human's.

### Simplify the harness as models improve · 34:14
- Harness design starts from a model's uneven capabilities: identify the gap, fill it in scaffolding, improve the model, then remove the scaffolding.
- Context anxiety in Opus 4.5 disappeared in the team's 4.6 experiments, so they dropped session resets.
- Simplified arrangement: one continuous session with compaction; longer continuous builds instead of explicit sprints; evaluation after the build rather than after each sprint; shared state still on disk.
- The planner–generator–evaluator core survived; the surrounding scheduling machinery shrank.
- A simplified run produced a digital audio workstation at roughly half the earlier cost. The evaluator operated it (tempo, key, melody, drums) and it looked well developed — but the music was poor, because Claude cannot hear.
- Available primitives: auto mode (safer than routinely using `--dangerously-skip-permissions`), custom sub-agents with a harsh QA prompt, Playwright MCP or Claude for Chrome MCP, computer use for native apps, and skills to package grading rubrics.
- Compaction produces lossy summaries that can drift; explicit criteria, structured shared state, and clean role boundaries still matter.
- The adversarial pattern is reusable: a synthetic-dataset generator can have its own QA agent, then pass output to an integrator with a different QA agent, each stage with its own critic.
- **Reading traces is the primary debugging loop.** Read the whole thing, not a summary, and empathize with the model (the Claude for Chrome team imagined navigating a page with eyes closed). Turn findings into durable instructions (`CLAUDE.md`, skills, auto memory).
- Long-lived products need more than a finished first run: persist a timestamped work log (what was tried, the bug found, the fix, the result) and living project docs, so a later session can reconstruct what happened.
- Scope: the full pattern suits greenfield work; brownfield is narrower (for example, autonomous monitoring → issue or feature request → implementation agent → pull request → review before merge). The team explores autonomy rather than claiming to use it everywhere.

## Resources

- [Harness design for long-running application development](https://www.anthropic.com/engineering/harness-design-long-running-apps) — the companion account (mentioned).
- [Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents) — the earlier initializer-and-coding-agent pattern (mentioned).
- [METR task-completion time horizons](https://metr.org/time-horizons/) — interactive measurements and methodology (mentioned).
- [Geoffrey Huntley's Ralph technique](https://ghuntley.com/ralph/) — repeated coding-agent loops (mentioned).
- Claude Code Ralph Wiggum plugin — session-local iteration with completion promises and max iterations (repository, mentioned).
- [Playwright MCP](https://github.com/microsoft/playwright-mcp) — Microsoft's browser-automation MCP server (mentioned).
- Claude Agent SDK — current documentation for building with Claude's agent loop (mentioned).
- Claude Code permission modes — current implementation guidance on permission behavior (update).
