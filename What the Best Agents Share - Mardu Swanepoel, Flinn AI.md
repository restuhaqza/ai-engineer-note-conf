# What the Best Agents Share

| Field | Details |
| --- | --- |
| **Speaker** | Mardu Swanepoel |
| **Affiliation** | Flinn AI |
| **Type** | Lightning talk |
| **Date** | 2026-05-26 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=7CrPrHgoEYk) |
| **Tags** | `agent-engineering` `architecture` `rag-context-and-search` |

## Summary

Mardu Swanepoel of Flinn AI asks a practical question: what can you learn from
an agent that already works well? He borrows Picasso's "steal" as a method — not
copying, but studying something deeply until you understand it, then making your
own better version. His subjects are Cursor, Claude Cowork, Harvey and Manus,
which he presents as worth studying rather than the winners of any ranking.

He identifies four patterns these products share and, for each, explains what it
does, the value it adds, and how it looks in a real interface. **Focus modes**
constrain the task and the input, which gives engineers a smaller optimization
target and gives users clearer expectations. **Transparent execution** exposes
the agent's process so delegation becomes collaboration. **Personalization**
supplies the knowledge and preferences a user would have applied, improving
speed to understanding. **Reversibility** lets users undo actions, which bounds
the cost of mistakes and makes people bolder about delegating.

The through-line is trust. A mode sets an agreement about what will happen, a
visible process lets a user intervene before work is wasted, personalization
keeps the agent from doing *something* instead of the right thing, and undo makes
the downside of a wrong result bearable. Swanepoel closes with a small but
pointed example: Harvey's Microsoft Word add-in, which puts the agent's changes
into Word's familiar review workflow instead of an opaque replacement document.

## Key Takeaways

- **Study good agents, don't just admire them.** The reusable lesson is the
  product behavior, not the brand.
- **Focus modes constrain the action space and the input space.** A planning or
  debug mode gives a smaller job to do well.
- **Modes also set user expectations.** Selecting one establishes what to supply
  and what kind of result to expect.
- **Transparent execution turns delegation into collaboration** by showing what
  the agent read, assumed and is uncertain about.
- **Visibility enables early intervention.** Seeing the agent read the wrong
  documents at step two is a concrete reason to redirect it.
- **Personalization improves speed to understanding, not just speed to
  outcome.** A fast result that ignores the user's approach is useless.
- **Reversibility bounds the cost of mistakes,** which makes users more willing
  to experiment and take on higher-value tasks.
- **Meet users where they already work.** Harvey's Word add-in adds the agent to
  an existing review workflow rather than replacing it.

## Detailed Notes

### Study strong agents to improve your own · 00:15
- Swanepoel opens with a quote he attributes to Picasso ("steal") and unpacks
  it: not taking someone's work, but studying it deeply, making it your own, and
  using that to build something better.
- His examples — Cursor, Claude Cowork, Harvey and Manus — are products he finds
  worth studying, not winners of a measured ranking.
- For each pattern he asks three things: what it entails, what value it adds,
  and how it looks in a working interface.
- That keeps the lesson on the behavior rather than the product label.

### Focus modes constrain the task · 01:46
- **Focus modes narrow both the action space and the input space.**
- For engineers this creates a tractable target: drop tools that do not belong in
  the mode, refine the system prompt around the remaining task, and tune evals
  for that narrower behavior before expanding.
- For users, an open-ended "ask me anything" interface invites broad
  expectations without saying which inputs will work. Choosing a mode sets a
  specific agreement about what to supply and what to expect.
- Cursor shows this with a mode dropdown whose modes differ in procedure, not
  just name:

  | Mode | Agent behavior | Expected result |
  | --- | --- | --- |
  | Planning | Asks questions and develops an approach | A plan, without writing code |
  | Debug | Forms hypotheses and collects runtime evidence | An investigation of the code problem |

- In planning mode, producing no code is part of the contract.
- In debug mode the agent follows hypotheses about faults — he describes
  spinning up a dedicated debug server and sending logs there to find out what
  is happening.

### Transparent execution enables collaboration · 03:42
- **Transparent execution turns delegation into collaboration** by showing the
  process alongside the result.
- Which material did the agent read? What did it assume? Where is it uncertain?
  Those details give the user a basis for trusting or questioning the output.
- Visibility also allows an intervention before more work is wasted — for
  example, seeing at step two that the agent read Notion documents A and B,
  which the user would not have chosen.
- The Claude Cowork interface makes several parts of execution inspectable:
  - **Progress:** a list separating completed work from upcoming work, including
    the next step.
  - **Context:** the context being used and the skills the agent draws on.
  - **Tool activity:** tool calls expose their inputs and outputs, so the user
    can inspect more than a generic working indicator.
- Manus offers something similar through task progress and what it inspected and
  concluded.
- Both keep the user involved while there is still time to influence the result.

### Personalization accelerates understanding · 05:44
- An agent can execute visibly yet still approach the task the wrong way.
- Personalization gives it the knowledge, systems, principles and patterns the
  user would have applied.
- The goal is **speed to understanding**, not just speed to outcome: a quickly
  generated result can be useless if it misses the user's intended approach or
  unstated preferences.
- Harvey makes this explicit through **playbooks** — a legal firm's methods and
  principles for a task such as contract review, which the agent can reuse.
  Swanepoel qualifies this as a nonexpert understanding of legal work.
- Harvey also uses **memory**: it creates memories from instructions and
  interactions and draws on them later. He presents this as a capability
  description, not universal availability.
- Claude's skills, connectors and connected systems add more ways to supply
  knowledge and personalize the agent.

### Reversibility bounds the cost of mistakes · 07:42
- The final pattern covers what happens when the agent gets something wrong.
- **Reversibility bounds the cost of mistakes** by letting users undo agent
  actions; knowing the downside makes delegating easier.
- He connects that confidence to users being bolder and attempting higher-value
  tasks.
- Cursor illustrates why the granularity of undo matters:

  | Scope | Control described in the talk |
  | --- | --- |
  | Line | Accept, reject or roll back individual changes |
  | File | Accept changes at the file level |
  | Conversation | Return to an earlier conversation state and undo associated changes |

- The conversation example goes back before the last three messages and undoes
  the changes made during them, letting a user retreat from a direction without
  deciding on every edit separately.
- Reversibility also supports deliberate experimentation: Cursor can produce
  multiple outputs from the same input in parallel using different models, with
  the expectation that all but ideally one will be discarded.
- Rejection becomes part of the intended workflow rather than an exceptional
  failure.

### Harvey uses Word's native review workflow · 09:31
- The last example places reversibility inside a familiar editor.
- Harvey's Word add-in runs inside Microsoft Word, where users already handle
  document revisions.
- He describes an integration with the native Word API that lets users make and
  view the agent's changes the way a reviewer or editor normally would.
- The agent's contribution enters an existing review workflow, giving users a
  familiar way to inspect changes instead of accepting an opaque replacement
  document.

## Resources

- [Cursor Debug mode](https://prod.cursor.com/help/ai-features/debug-mode)
- [Claude Cowork](https://claude.com/product/cowork)
- [Harvey's Word review experience](https://www.harvey.ai/blog/improved-word-experience)
- [Harvey Memory: personalization and controls](https://www.harvey.ai/blog/memory-is-here-harvey-personalized)
