# Replacing 12K LoC with a 200 LoC Skill

| Field | Details |
| --- | --- |
| **Speaker** | David Gomes |
| **Affiliation** | Cursor |
| **Type** | Keynote (track) |
| **Date** | 2026-04-30 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=WE_Gnowy3uw) |
| **Tags** | `developer-workflows-and-testing` `coding-and-developer-tools` `evals` `rag-context-and-search` |

## Summary

Cursor shipped parallel coding in Git worktrees, plus a "Best-of-N" feature that runs the same task through several models so you can compare results. David Gomes explains how the team later deleted around 15,000 lines of code (LoC) and re-implemented the same workflows as roughly 200 lines of Markdown, built from two existing primitives: agent skills (reusable instruction packs) and subagents (a parent agent that delegates).

The refactor trades maintenance burden and rigidity for flexibility. Cursor no longer maintains a large parallel-agents feature for a small slice of power users, and users gain new abilities: switching into a worktree mid-chat, working across multiple repositories, and asking the coordinator agent to combine pieces of different models' solutions.

The cost is a real one. The old implementation physically prevented an agent from touching files outside its worktree. The new version just tells the model to stay in its directory and trusts it. Over long sessions models drift, and weaker models drift more. Gomes describes Cursor's response: evaluations that separately score the intended work and the forbidden work, reinforcement learning (RL, training via reward signals) on prompts like these, and better system reminders.

## Key Takeaways

- **Markdown can replace a lot of app code.** A full feature became a short skill plus subagents, saving the team ongoing maintenance.
- **A Git worktree is a separate checkout** of the same repo, letting parallel agents work without overwriting each other's files.
- **The old feature did 5 jobs:** lifecycle/context, isolation and setup scripts, judging, harness tweaks, and cleanup.
- **Prompts handle orchestration now.** `/worktree` and `/best-of-n` are Cursor commands (prompt-like, loaded on demand), not hard-coded behavior.
- **Isolation became a suggestion.** The old code enforced the checkout boundary; the new code trusts the model to obey a directory instruction.
- **Evals need two scorers:** one for "did the model work in its worktree?" and one for "did it touch the primary checkout?" Both matter.
- **Model choice affects reliability.** In early tests Haiku often drifted into the primary checkout; Composer and Grok did better.
- **Native worktrees are coming back** for Cursor's agent-focused window. The skill stays and improves via evals, RL, and system reminders.

## Detailed Notes

### How much of a feature can become instructions? · 00:15
- A **Git worktree** is a separate checkout tied to the same repository. Different agents can edit the same project in separate folders without clashing. A file can change in one worktree while the primary checkout stays untouched.
- In Cursor's original feature, commands and lints were scoped to the agent's checkout. You could watch a grid of agents run in parallel, then ask one to open a pull request from its worktree.
- **Best-of-N** gave several models the same prompt to compare. For a front-end project you could preview the different interfaces before choosing.
- This shipped alongside Cursor 2.0 in October 2025. Gomes' goal now: keep the workflow but move its orchestration into instructions.

### The code behind parallel checkouts · 03:10
- The old feature needed real application machinery: creating and managing worktrees, and telling each agent which checkout it was in.
- It also handled isolation and setup: keeping agents inside their checkout and running user-configured setup scripts.
- **Judging**: a thumb-up recommendation comparing competing implementations.
- **Harness support**: added system reminders (reminders injected into the agent's context) to keep agents on track.
- **Cleanup**: removing abandoned worktrees before users running hundreds of them filled their disk.
- Gomes estimates his deletion PR removed about 15,000 lines. The replacement is roughly a 200-line skill. These are approximate descriptions of the refactor.

### A parent agent becomes the coordinator · 04:58
- The replacement combines two existing primitives: **agent skills** (reusable instructions) and **subagents** (a parent delegating work to children with their own context).
- Demo: invoke `/worktree` with a task like "fix a typo in the footer". The agent creates a separate checkout and works there.
- The worktree instructions are short: create the checkout, run configured setup scripts, do the work inside it, and keep edits out of the primary checkout.
- The last instruction is the hard one. Creating a directory is one action, but *staying* in it is a constraint the session must keep honoring.
- Best-of-N adds coordination: the parent starts one subagent per model, each makes its own worktree, the parent waits, then compares and helps the user choose.
- Gomes says the Best-of-N prompt is around 40 lines of Markdown, versus roughly 4,000 lines of code before.

### The command contract · 07:19
- The skills must handle the environment: Windows, Linux, and macOS directions, setup-script runs per worktree, and an emphatic "do not leave your checkout".
- The commands:

  | Command | Purpose |
  | --- | --- |
  | `/worktree` | Start work in a separate checkout. |
  | `/best-of-n` | Give the same task to multiple agents. |
  | `/apply-worktree` | Bring side-checkout changes into the primary checkout. |
  | `/delete-worktree` | Remove the side worktree. |

- He calls it a skill, but technically these are **Cursor commands**. Like skills, their prompts load on demand. Cursor chose commands so it can update prompts on its backend — an improved prompt reaches users without a client update.
- The Best-of-N demo gives one task to Kimi, Grok, Composer, GPT, and Opus: five subagents, five contexts, five worktrees. Opus finishes later, then the parent compares them.
- Bonus over the old UI: the parent can mix a preferred part of one implementation with a preferred part of another, instead of just picking a winner.

### Less maintenance, more flexible workflows · 09:58
- Worktrees are an **advanced feature**, used by power users who want parallel execution and grids of agents, not most Cursor users. Gomes argues it should not demand outsized maintenance.
- **Switch mid-chat:** you can discuss a task, then invoke `/worktree` to move it into a side checkout. The old feature did not allow this, and adding more prompt-UI settings was unattractive.
- **Multi-repo:** the old implementation disabled worktrees for multi-repo setups. Now an agent working across a separate front-end and back-end can create a worktree in each and open a pull request in each.
- **Better judging:** the parent has richer context about what its subagents did, so it can stitch together useful pieces.
- The flexibility comes from giving the coordinator the live conversation, rather than fixing every workflow in the interface up front.

### Instructions replace an enforced boundary · 12:14
- Feedback was mixed; some users missed the old workflow.
- **The most consequential regression is checkout discipline.** The old code made it impossible for a model to touch files outside its worktree. The new code says "work in this directory" and trusts the model.
- A directory instruction is not an enforced file-access boundary. Over a long session the model can forget where it should be. Gomes says weaker models are especially prone to this drift.
- **Visible setup:** worktree creation now shows up as agent activity in the chat. He says it *feels* slower without actually being slower, but gives no timing measurements.
- **Reduced discoverability:** the old dropdown offered local, cloud, or worktree execution. With it gone, users must already know `/worktree` exists. The team accepts this for an advanced feature.

### Evaluate both the intended work and the forbidden work · 14:13
- Cursor is improving adherence two ways: evals to improve prompts, and RL to train its own **Composer** model.
- Gomes says Composer 2 had no RL tasks using prompts like these in this kind of environment. The team is adding such tasks, hoping later Composer releases improve. For other providers' models, Cursor can only share feedback.
- Eval setup: Braintrust (shout-out) plus agent-assisted eval authoring, run through the headless Cursor CLI (non-interactive mode).

  | Scorer | Question |
  | --- | --- |
  | Intended work | Did the model work in its assigned worktree? |
  | Checkout violation | Did the model work in the primary checkout? |

- Both are needed: an agent can do the requested change *and* still modify the primary checkout along the way.
- Early evals are simple and do not reproduce very long sessions, where Gomes sees behavior degrade. In these tests Haiku often drifted; Composer and Grok did better. No numeric scores were given.
- Next steps: harder cases, finding recurring failure patterns, revising prompts, and adding better system reminders to hold the constraint over a long conversation.

### Where native worktrees still belong · 17:05
- Cursor plans a partial return to native implementation. Cursor 3.0 already introduced an agent-focused window; a fuller native worktree experience there is planned work, not finished.
- The reasoning is about *where the workflow belongs*: people who want heavy local parallelization are the same people likely to use an agent-first interface.
- Git worktrees also have limits: slow to create, heavy on disk, and only work in Git repos. Projects not on Git have no local parallelization option in Cursor.
- The team is looking into other parallelization primitives that don't depend on Git. Expect details later.

## Resources

- **Cursor 2.0 release notes** (article) — original parallel-agents release.
- **Cursor 3.0 release notes** (article) — introduces the Agents Window.
- **Worktrees and Best-of-N release discussion** (article) — command announcement and multi-repo notes.
- **Cursor agent skills** (docs) — how skill packages supply reusable instructions.
- **Cursor subagents** (docs) — delegation, separate context, parallel execution, model config.
- **Cursor CLI (headless)** (docs) — running Cursor non-interactively for evals.
- **Writing Braintrust scorers** (docs) — defining separate evaluation criteria.
- **Git worktree reference** (docs) — creating and managing extra working trees.
