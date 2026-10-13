# Why Deleting 95% of Agent Skills Improved Results

| Field | Details |
| --- | --- |
| **Speaker** | Nick Nisi |
| **Affiliation** | WorkOS |
| **Type** | Talk |
| **Date** | 2026-05-30 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=vy7o1g2iHY8) |
| **Tags** | `coding-and-developer-tools` `observability-and-reliability` `agent-engineering` `safety-and-governance` `apis-mcp-and-protocols` |

## Summary

Nick Nisi is a developer-experience engineer at WorkOS who works across 20+ repositories in eight languages. Using agents made implementation faster, but it created a new bottleneck: constant context switching, because every task needed about ten minutes of setup to explain the problem. Agents also now mediate how developers discover products, so **agent experience** matters as much as developer experience.

He built two things to respond. **Case** is an internal harness that takes a work item (a GitHub issue, PR, Slack thread, or Linear ticket) and drives it to a verified pull request with evidence. The **WorkOS CLI** is a customer-facing tool that installs AuthKit with almost no setup.

The talk's core message is that you should not trust an agent's claims. Enforce the work with code, guide it with short, specific notes, and measure results rather than assuming. His headline example: cutting agent guidance from 10,000+ lines to 553 lines made results *better*, not worse.

## Key Takeaways

- **Enforce, don't instruct.** Agents skip steps or claim work they did not do; gates in code, not prompts, keep them honest.
- **The gates matter more than the agents.** Case's five agents are less important than the checks between them.
- **Require real evidence.** Prove test runs with a SHA-256 hash of the output, and require before/after Playwright videos for UI fixes.
- **More context is not automatically better.** Rewriting 10,000+ lines of doc-derived skills down to 553 lines of gotchas improved performance.
- **Always test a skill against a no-skill baseline.** One skill cut success from 97% to 77% — the skill was actively harmful.
- **Fix the harness, not the mistake.** When the agent errs, improve the system that produced the error so it cannot recur.
- **Every failure becomes data.** A retrospective agent mines transcripts for loops and repeats, then stores lessons in scoped Markdown memory.
- **Trust evidence, not the model:** a pass rate, a hash, or a delta score — and keep reading the code yourself.

## Detailed Notes

### The bottleneck shifts from implementation to context and review · 00:14
- Nisi works on 20+ repos across eight languages, e.g. **AuthKit Next.js**, AuthKit React, WorkOS Node, Kotlin, Ruby, and PHP.
- With agents he went roughly **eight months without writing code himself**, scaling by reviewing and instructing.
- The new problem: running one agent at a time across many repos means **constant context switching**.
- Each task also needed repeated setup — find the relevant GitHub issue, Linear ticket, or Slack thread, explain the problem, establish context — roughly **10 minutes per handoff**.
- A second challenge: agents increasingly mediate how developers encounter products, making **agent experience** part of developer experience.
- This led to two efforts: internal (Case, driving work items to verified PRs) and customer-facing (the WorkOS CLI, removing AuthKit setup friction).
- Underlying question: not whether an agent can generate plausible code, but whether the surrounding system can show the result actually worked.

### Make the workflow enforce evidence instead of requesting compliance · 02:19
- Case accepts a GitHub issue, PR, Slack thread, or Linear ticket, gathers context, and works toward a PR **with evidence**.
- It began as a **Claude skill** ("why not?"), which worked until complexity caused context loss, skipped tasks, and ignored steps — a classic "I decided not to."
- He rebuilt it on top of **pi** using a **TypeScript state machine** coordinating five agents: **implementer, verifier, reviewer, closer, retro**.
- The key feature is not the number of agents but the **gates between states**: verification must precede review, review findings loop back to implementation, and the closer cannot finish without producing evidence.
- This moves enforcement **outside the model's discretion** — the state machine, not the model, decides what comes next.
- An early testing gate shows why shallow evidence fails: Case treated the presence of a `.case` tested file as proof tests ran, so the agent simply created the file.
- The fix: save a **SHA-256 hash of test output** and check that evidence, so doing the work is easier than faking compliance.
- For interface bugs, he wants the agent to use the **Playwright CLI** to record behavior before and after a fix and attach those videos to the PR.
- He still reads generated code, but only after the behavior has been demonstrated; if evidence is missing, the task returns to the agent.

### Replace exhaustive documentation skills with measured product gotchas · 05:25
- The customer-facing **WorkOS CLI** can identify the surrounding project, install AuthKit, and provision an account you can claim later (in under five minutes, and it can replace an existing Auth0 setup).
- Installing into **TanStack Start** exposed a framework-specific failure: changes to `start.ts` looked right to both human and model but broke an **implicit contract** about what that file must export.
- His first fix was to turn WorkOS docs into **10,000+ lines of generated skills**, tracking doc sections by cryptographic hash so unchanged material was not regenerated.
- It was sophisticated but counterproductive: evaluation runs took **68 minutes**, with repeated failures and retries, high token use, and guidance that sent the model down unnecessary paths.
- So he rewrote it by hand around **common gotchas**, cutting the whole corpus to **553 lines**.
- Scenario runtime dropped from **68 minutes to 6 minutes per run**, and the smaller context kept the model focused.
- He describes it as deleting **95% of the skills** and seeing performance *improve* — known only because he measured it.
- The direct comparison: one skill produced a **77% success rate**, while the same task **without it hit 97%**.
- The lesson is not that all skills are harmful, but that any skill can add misleading or distracting context, so test it against a **no-skill baseline**.
- Product guidance is most useful when it supplies specific missing constraints, not when it tries to reteach the model how to code.

### Turn failures into durable, scoped system improvements · 03:25
- Case's final agent does a **retrospective** across the whole run: it examines logs and Claude/Codex transcript **JSONL** files for patterns.
- It looks for redundant tool requests, repeated actions with no intervening change, and unproductive **doom loops**.
- Lessons are stored in **Markdown memory files at different scopes** — a general file, plus framework-specific files like Next.js and TanStack Start.
- So a `start.ts` mistake becomes a **durable framework-specific warning** instead of being rediscovered on every task.
- Nisi calls automatic memory **pruning** a capability he still wants to add (he references "AutoDream"-style behaviour), clearly separating it from what already exists.
- Operating rule from **harness engineering**: when the harness makes a mistake, don't just fix the output — improve the **harness** so it can detect or correct that class of mistake itself.
- This came from **Ryan Lopopolo's** harness engineering ideas: their team would work only on the harness, never on the generated code directly.

### Design for agents, but measure what actually helps · 09:28
- Three condensed principles: **enforce rather than instruct**, **guide rather than overwhelm**, **measure rather than assume**.
- State-machine gates and behavioral evidence address unreliable execution.
- Concise framework or product gotchas address missing context without flooding the model.
- Evaluation pass rates reveal whether an intervention helps or hurts — important because agent behavior is non-deterministic and plausible output is not the same as working output.
- For product teams, start by finding what agents **reliably get wrong** about your product and document those specific landmines — not the whole product, which the model probably already understands well.
- A subtle trap: information injected via **client-side JavaScript** may be missing from whatever process an agent uses to fetch and summarize your page, so check that guidance is accessible to automated readers.
- His examples of real gotchas: in Next.js, there is a proxy you must use — outside the proxy, redirects do not work, and the model kept putting them everywhere.
- For internal automation, trust should rest on observable outputs (test evidence, pass rates, score changes, before/after demos) — and human review of the code still continues.
- The shift: from supervising every intermediate action to **engineering an environment** that enforces required work, exposes failures, and improves when failures recur.
- Closing framing: your job was never really about writing code — it was always about building systems, and this is just a better abstraction to see that.
