# Your coding agent doesn't always follow your rules

| Field | Details |
| --- | --- |
| **Speaker** | Talha Sheikh |
| **Affiliation** | Checkout.com |
| **Type** | Talk |
| **Date** | 2026-07-08 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=MpZzWMdmQCE) |
| **Tags** | `coding-and-developer-tools` `agent-engineering` `evals` |

## Summary

Talha Sheikh of Checkout.com describes a familiar annoyance. You hand Claude Code a feature, watch it split the work across subagents, and get "task completed" — then the result fails when you run it. He ends up as the enforcement layer, asking for one small fix after another. His real goal was simpler: play Cyberpunk on his Xbox while the agent worked.

His fix is a deterministic check between the agent claiming completion and the result being accepted. Vector V1 uses **Claude Code hooks** to run configured checks when the agent tries to finish; failed checks are fed back so the agent repairs and retries. The demo shows checks failing, the agent retrying, and eventually passing.

He argues capability and reliability are different questions. A smarter model, a better spec, or more MCP tools (MCP is a standard way for agents to call tools) still do not establish that this result meets the requirement. He proposes a language-agnostic pattern in which developers define their own checks, and closes by reframing the skill of the job: work on the harness and the verification system, not just the code.

## Key Takeaways

- **Completion is a claim, not proof.** Nothing verifies it unless you add a check.
- **Vector V1 loops on checks.** Hooks run deterministic checks when the agent stops; failures return as feedback and the agent retries.
- **Capability is not reliability.** Better models, specs, and instructions do not replace verification.
- **A stronger harness may cut model cost.** A cheaper model inside good checks might be enough for some tasks.
- **Share the pattern, not the rules.** Different teams enforce different things, so the reusable piece is language-agnostic and developer-defined.
- **Checks fit many points.** Conversation end, before commit, inside multi-agent workflows, on async work, and via LLM (large language model) judgment.
- **Deterministic invocation is enough.** A check can reliably run at a fixed point while using an LLM judge whose output varies.
- **"Enforce, don't instruct."** Value shifts from the code produced to the verification designed around it.

## Detailed Notes

### The agent says done; you check anyway · 00:16
- Typical flow: give Claude Code a feature, it splits the work into subtasks, runs subagents, and reports "task completed".
- Running the result reveals a failure; asking for a fix brings another assurance, then another small omission.
- Whether he gives a full spec, some instructions, or almost none, he still has to correct the work.
- His summary: **"I am the enforcement layer"** — the human is the only thing checking the result.
- He wanted a deterministic way to check the work the moment the agent claims it is done, including whether it was done the way he asked.

### Turn completion into a checked retry loop · 01:57
- **Vector V1** uses Claude Code hooks to invoke a checker automatically when the agent finishes.
- (He describes this as finishing a session; current Claude Code terminology is the `Stop` event, when the agent stops responding.)
- The hook can block completion and return a reason to the agent.
- The loop: the agent tries to finish, the hook runs the configured checks, a failed check returns feedback, the agent repairs, and the checks run again before acceptance.
- A simple check command can run the project's `test` and `typecheck` scripts and exit non-zero if either fails.
- The hook integration is the other half: it returns that failure to the agent instead of letting it disappear into a log.
- In the demo, checks fail at first, the agent retries, and eventually all pass — completion becomes something the system evaluates.

### Better models and clearer instructions don't remove the check · 02:56
- An Anthropic engineer told him a future agent or model would make enforcement unnecessary — worrying for someone building a verification tool.
- He also cites **Project Glasswing** and Claude Mythos Preview as signs of optimism about advancing capabilities.
- His point: capability and reliability are different questions; a model can do more yet still not guarantee a given result meets the requirement.
- The same holds for the inputs around the model: better specs, useful MCP servers, and the right subagents and context all help.
- They help the agent understand and perform the work, but instructions describe intent while verification checks whether it happened.
- Giving instructions is not the same as giving verification.

### Can a stronger harness offset model cost? · 04:24
- With checks constraining acceptable output, a smaller model such as **Haiku**, or an open-source model, might be sufficient for some tasks.
- Three positions: rely on a frontier model like **Opus**; pair Vector with a few guardrails; or invest heavily in the harness to cut model spend.
- Asynchronous tasks are another option he mentions.
- This is a cost hypothesis, not a measured result — the talk offers no benchmark or comparison that accounts for check execution and retries.
- The open question: does the extra harness work make a less expensive model sufficient for the required outcome?

### A portable contract, with checks developers define · 05:04
- As more people saw Vector, he noticed everyone building their own enforcement — Anthropic, his own company, Meta.
- A tool tuned to his rules will not fit someone else's, so the reusable piece has to be the pattern rather than a fixed set of rules.
- The proposed pattern is language-agnostic and lets developers bring their own enforcement.
- Integration points: conversation completion, before committing, inside multi-agent workflows, on asynchronous operations, and via LLM judgment.
- **Deterministic invocation does not require deterministic evaluators** — a check can reliably run at a fixed point while using an LLM judge whose judgment varies.
- The contract ties a task to the conditions for fulfilling it; developers define the checks in between, separating orchestration from project-specific meaning.

### Same loop, different systems · 06:47
- Anthropic's **executor–advisor** arrangement: one agent codes while an advisor gives feedback that can change its course. Current docs describe the advisor as strategic guidance, not deterministic verification.
- OpenAI's **harness engineering**: the agent gets tools and context inside an environment designed to support checking its work.
- **Qodo's code review**: findings and issues feed back into a pull request after the agent has produced changes.
- These differ in mechanism but share a refusal to treat generated code as the finish line.
- A WorkOS talk gives the compact line: **"Enforce, don't instruct"** — make verification part of the workflow rather than hoping the agent remembers to check.

### Verification sets the pace · 08:00
- Fast code generation does not remove the need to proceed carefully; something still has to confirm the changes work.
- He ties this to calls to go slow, including a keynote's blunter "slow the hell down."
- The value shifts from the code produced to **the verification designed around it**.
- The relevant skill expands from writing code to establishing that the code satisfies its purpose.
- His closing advice: work on the harness and the verification system, not only the code.

### Availability and the cost of building it · 09:08
- Asked whether the tool is public, he says yes: it is called **Vector Harness**, and he can share it through a LinkedIn message.
- He does not walk through installation or deployment; the route offered is direct contact.
- A final joke returns to cost: asked about being a top token spender at his company, he replies that he needs those tokens to build the verification layer.
- The exchange ends on the expense of building the harness, not on a measured saving.

## Resources

- Building Vector: Customizable Agent-Harness System — Sheikh's write-up of the developer-defined check pattern (related).
- [Claude Code hooks reference](https://code.claude.com/docs/en/hooks) — hook configuration and lifecycle events, including checks that can block completion (mentioned).
- [Project Glasswing announcement](https://www.anthropic.com/glasswing) — Anthropic's defensive cybersecurity initiative using Claude Mythos Preview (mentioned).
- [Claude advisor tool](https://platform.claude.com/docs/en/agents-and-tools/tool-use/advisor-tool) — pairing an executor model with an advisor that supplies strategic guidance (update).
- [Harness engineering](https://openai.com/index/harness-engineering/) — OpenAI's account of building an agent-oriented engineering environment (mentioned).
- [Qodo Code Review](https://docs.qodo.ai/code-review) — placing contextual review findings and rule checks into pull requests (mentioned).
- [WorkOS: Building AI Systems that Ship](https://www.linkedin.com/posts/workos-inc_how-do-you-make-your-product-work-for-agents-activity-7448333814797537280-CTNM) — lessons on making enforcement part of the workflow (related).
