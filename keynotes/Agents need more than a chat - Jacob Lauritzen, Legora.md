# Agents need more than a chat

| Field | Details |
| --- | --- |
| **Speaker** | Jacob Lauritzen |
| **Affiliation** | Legora |
| **Type** | Keynote |
| **Date** | 2026-04-22 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=XNtkiQJ49Ps) |
| **Tags** | `coding-and-developer-tools` `industry-applications` `agent-engineering` `reasoning-and-models` `data-and-model-adaptation` |

## Summary

Jacob Lauritzen, CTO of Legora (a collaborative AI workspace for law firms), argues that complex agents — especially in a "vertical" like legal work — cannot be driven well through a linear chat. Asking an agent to draft a contract and then fix clause three usually turns into re-reviewing the whole document, and a context reset can wipe the thread's understanding.

His core point is that as doing the work becomes cheap, planning and reviewing become the bottleneck. He uses the "verifier's rule" to explain why: tasks that are easy to verify get solved, while tasks that are hard to verify stay hard. Legal work has many kinds of verification, from easy definition checks to litigation strategy that even experts disagree on.

The fix, he argues, is to put human judgment where the work happens. That means decomposing tasks, adding guardrails, encoding judgment in reusable skills, asking the human when something is genuinely uncertain, and — most importantly — collaborating in persistent artifacts like documents and tables instead of a chat window.

## Key Takeaways

- **Cheap execution moves the bottleneck.** When generating the work is easy, planning and reviewing dominate the effort.
- **The verifier's rule frames the problem.** A task that is solvable and easy to verify tends to get solved; hard-to-verify work does not.
- **Verification varies by task, not just industry.** Checking contract definitions is easy; judging litigation strategy has no single right answer.
- **Control and trust are different questions.** Control is how well you steer the work; trust is how much you need to review.
- **You can raise trust** by making tasks verifiable, using proxies (past "golden" contracts), decomposing tasks, and adding guardrails.
- **Skills beat upfront planning for contingencies.** Reusable judgment attaches to nodes in the work graph and appears when needed.
- **Don't block the agent.** Let it make a provisional decision, log it, keep working, and let the human reverse it later.
- **Chat is input, not the whole workspace.** Persistent documents and tables keep context where the decision belongs.

## Detailed Notes

### Fix clause three. What happened to clause four? · 00:27
- The opening scene: you ask an agent to research a matter, draft a contract, and make no mistakes.
- It starts reading, launches subagents, searches the web, and writes files — over and over. After about thirty minutes you get a contract.
- Clause three looks wrong. You ask it to check another document and fix it. The agent replies, "You're absolutely right," and then you see a **compaction** (the context getting squeezed), which is when confidence drops.
- It produces a new contract. Now the question is bigger than clause three: did anything else change? A targeted correction has become a full-document review.
- Lauritzen introduces Legora: a collaborative AI workspace for law firms, with more than a thousand customer organisations across more than fifty markets. He says it is growing very fast and is hiring in London.
- The shared goal for vertical AI companies is to let agents complete more complex work end to end.

### Cheap execution moves the bottleneck · 02:18
- Over the past six to twelve months, doing the work became cheap. Planning and reviewing it now dominate the effort.
- Someone still has to specify the result, capture the non-functional requirements, and inspect what comes back.
- A large GitHub pull request captures the pain: producing a big change can be easier than judging whether it is right.
- "AI-reviewing" your own agent's output might help, but Lauritzen leaves its reliability open.
- He cites the **verifier's rule**, attributed to Jason Wei: if a task is solvable and easy to verify, AI will solve it.
- For foundation models, easy verification enables reinforcement-learning environments and post-training. Lauritzen extends the idea to agents: run the task, find what's wrong, give corrective feedback, repeat.
- The rule is a way to reason about the bottleneck; it does not guarantee the loop will converge or how long it will take.

### A vertical contains many kinds of verification · 03:56
- Verification difficulty varies within an industry, not just between industries.
- Checking contract definitions is easy to perform and to check. Drafting contract language is also easy for an agent, but proving it will have the intended legal effect is much harder.
- Lauritzen's strong framing: the real test happens when a contract reaches court and a judge interprets it.
- Litigation strategy is harder still. Ask five lawyers how to run a case and they may disagree, so there is no single answer to score against.
- Coding shows the same spread: a bounded implementation task is easy to check, while building a successful consumer app depends on an outcome you cannot verify in advance.
- The practical unit of analysis is the task, not the label "legal" or "coding."

### Control vs trust, and giving the agent something to check · 05:05
- Two dimensions shape human-agent collaboration. **Control** is how effectively a person can put their knowledge into the work and steer decisions.
- **Trust** is how much of the work that person needs to review — with low trust they inspect every trace, with high trust they inspect little.
- These are separate. You might need strong control over a strategic choice while trusting the agent to do the mechanical work.
- One way to raise trust is to make a task more verifiable. For feature work, browser access and test-driven development let the agent check its own behaviour.
- In legal work, where direct verification is unavailable, previous successful contracts act as **golden contracts**. Comparing the new draft to them is a verification proxy, not proof of legal correctness.
- Decomposition also helps. Separate the judgement calls — risk profile, precedent documents, negotiation stance — from operations that allow clearer checks.
- Definition checking is essentially linting: are all defined terms used, and is every used term defined? Those two checks are just set differences.
- That gives the agent specific findings to fix, while the human keeps the contract's strategic choices. Extracting and interpreting the terms remains a separate concern.

### Trust can also come from limiting scope · 07:13
- **Guardrails** raise trust by shrinking what the agent can do: edit only three named files, read only one directory, or search only approved sites.
- Claude Code shows both extremes. At one end, the agent asks permission for every action, which makes real progress cumbersome.
- At the other end, the user bypasses permission checks and lets it run; the slide shows `--dangerously-skip-permissions`, with a warning about deleting a production database.
- The important engineering distinction is between *requesting* a restriction and *enforcing* one.
- Current Claude Code documentation treats permissions as harness controls, not something a prompt can change — a limit of the harness, not the prompt.

### Planning only reaches what it can anticipate · 07:53
- Model a complex agent task as a tree of work — more generally a **directed acyclic graph** (a set of steps with dependencies and no loops).
- Example: a report on employment contracts. The agent researches the organisation, reviews several aspects of each contract, then drafts the report.
- If the human can only intervene at the root, they issue a request and wait for all the downstream work. That recreates the opening contract problem at a larger scale.
- **Planning** improves this by aligning on the approach first: the human approves steps, names the clauses to inspect, and sets review criteria.
- But knowing enough to write a complete plan may require doing much of the task already. An unusual clause may not become visible until the agent reads that contract.
- Lauritzen compares this to a coworker who agrees an approach and then disappears until the final document is ready. There is no chance to react to discoveries along the way.
- He therefore predicts planning will not stay the dominant collaboration method: the problem is the gap between when judgment is given and when it is needed.

### Put judgment at the point of discovery · 09:49
- **Skills** place reusable human judgment at individual nodes of the work graph.
- Instead of describing the whole report upfront, a person can specify how confidentiality reviews should be done, or how to handle a special EU-law case in a termination clause.
- This supports **progressive discovery**: the agent picks up relevant guidance as work unfolds, handling situations the plan never listed.
- The catch is that skills only help if guidance already exists; there will not be a skill for everything.
- For the gaps, the next mechanism is **elicitation** — asking the human about the specific uncertainty encountered during the task, rather than demanding the user anticipate every problem in advance.
- To avoid blocking, combine elicitation with a **decision log**: make a provisional decision, record it, and keep working.
- The human reviews the log afterwards and reverses decisions where needed. The decision was used to keep going, but that is not the same as human approval — keeping it visible and reversible is what makes later intervention possible.

### Review decisions inside the artifact, not in chat · 11:14
- Now imagine the work graph ten or a hundred times larger. A chat with fifty questions is not a good way to get fifty useful answers.
- Each question needs the context of the contract, clause, or decision it concerns. A linear conversation flattens that branching structure into a stream the user must reconstruct.
- Lauritzen's claim: humans and agents should collaborate in **persistent artifacts** that expose the work in context.
- The right artifact depends on the industry. For legal drafting, a document is the natural workspace: highlight clause three, and the agent changes only clause three.
- Documents also support comments, tags for agents and people, and handoffs of specific portions to specialist agents. These keep the instruction attached to where it applies.
- For the employment-contract report, Legora uses a **tabular review** — a familiar primitive for its users. The agent sets up the review, examines the contracts, and flags items that need a human's judgment.
- The user then supplies judgment in place and sees what work the agent already did, giving both control and inspection. After that, the remaining agent work can start.
- Lauritzen notes PostHog and Linear recently converging on chat-style UIs. Chat as input is fine — it is flexible and language is universal — but it should not be the main mode of collaboration for a complex task.
- He ends with an org-chart example: he wished he could show a candidate an interactive chart instead of describing it verbally. Agents are not human, so they need not answer only in the form they were addressed.

## Resources

- **Asymmetry of verification and verifier's rule** — Jason Wei explains why reliable, inexpensive verification matters.
- **Equipping agents for the real world with Agent Skills** — a practical introduction to packaging reusable skills.
- **Claude Code permission configuration** — current guidance for configuring tool approvals and guardrails.
- **Introducing Linear Agent** — Linear's launch announcement showing agents operating in workspace contexts.
