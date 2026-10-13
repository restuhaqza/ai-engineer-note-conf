# Spec-Driven Testing for Agents With a Brain the Size of a Planet

| Field | Details |
| --- | --- |
| **Speaker** | Steven Willmott |
| **Affiliation** | Safe Intelligence |
| **Type** | Talk |
| **Date** | 2026-05-31 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=UQKg0td-Bf4) |
| **Tags** | `reasoning-and-models` `evals` `agent-engineering` `safety-and-governance` |

## Summary

Steven Willmott, CEO of Safe Intelligence, argues that a dataset of good input/output examples is not enough to specify what an agent should do. Traditional ML validation expresses desired behavior through a dataset and measures it with metrics like F1 and accuracy. That works up to a point, but deployment adds requirements — explicit rules, domain boundaries, permissions, and a defined range of inputs — that a set of examples does not capture.

His framing question is whether a smarter agent is automatically a better agent. Not always, he says. A more capable model can understand a harmful instruction hidden inside a poem that a weaker model would simply fail to parse. A broader remit also creates more attack surface, more behavior to test, and real token and latency costs. For automated deployment, he suggests aiming for enough capability to do the task with bounded opportunities for harm.

The talk's core idea is **specification-driven testing**: define the role and its constraints independently of whatever framework implements the agent, then use that specification to drive both security tests (can the agent be pushed across a boundary?) and robustness tests (does it keep working as inputs vary?). The specification should outlive the platform, so the accumulated definition of correct behavior survives a framework or model change. He closes with a call for an open, versioned way to express and share these specs.

## Key Takeaways

- **Examples alone under-specify behavior.** A dataset of passing cases leaves everything outside it unspecified, including rules like "never offer more than 10% off."
- **Bigger is not automatically safer or better.** Willmott's example: a stronger model may decode a jailbreak wrapped in a poem that a weaker model ignores.
- **Weigh two kinds of risk.** Instruction flexibility (what prompts it accepts) and action authority (what tools it can use) determine how much harm is possible.
- **Design the role independent of the agent.** Build the task benchmark before deciding which model or framework fills the role.
- **Capture rules, ontologies, domain knowledge, and roles.** Rules like refund windows, valid destinations, and term substitutions all shape what a valid test case is.
- **Tests should resemble integration tests.** The eval depends on context — agent card, policies, surrounding requirements — not just an isolated prompt and answer.
- **Specs improve security and robustness testing.** An agent's domain is both where it's most willing to help and where it has the most power.
- **Keep specs implementation-independent.** Integration, unit, and penetration tests should stay runnable even after you swap frameworks.

## Detailed Notes

### How do you specify what an agent should do? · 00:15
- Safe Intelligence began with formal verification of vision and tabular models: with the model in hand, examine whole regions of input space and find perturbations that push correct examples into failure.
- For language models whose internals aren't available, the work changes — generate edge and test cases intelligently, which is analogous but not the same guarantee.
- He distinguishes this from spec-driven **development** (using a spec to write code); his topic is using a spec to define what an agent should do.
- In ML, the usual answer is a dataset plus F1/accuracy — useful, but incomplete once you deploy.

### A smarter agent is a better agent — right? · 02:11
- Intuitively, a larger model should be better at its job, but that doesn't always hold for a deployed agent.
- His illustration: Marvin, the Hitchhiker's Guide robot with a planet-sized brain, is asked to make tea — plenty of intelligence, wrong fit for the role.
- A harmful instruction wrapped in a poem may fool a weak model but be decoded and followed by a strong one. He frames this as a hypothesis, not a proven rule.
- A broad remit increases both the surface an attacker can exploit and the behavior a developer must test.
- Using a large model for simple tasks wastes tokens and time.
- The target for automation: **enough capability to perform the task, with bounded opportunities for harm.**
- Harm has two dimensions — how flexibly it accepts instructions, and what it can actually do in your infrastructure. Wiring millions of dollars is riskier than answering questions.

### Define the role independently of the agent · 05:03
- Start by designing the task or role benchmark on its own, before choosing the agent.
- Ground-truth examples are one component; **rules** are another, like never offering over 10% discount or never refunding beyond 30 days.
- The word "never" makes testing hard: passing examples cannot prove a rule is never broken.
- **Ontologies and dictionaries** matter — an airline bot only serves certain destinations, and internal policy terms may be unknown to a general model.
- The testing system needs that same information to know which substitutions are valid.
- **Domain knowledge** constrains substitutions: gross profit and gross sales are different business quantities, not interchangeable phrasing.
- **Rights and roles** change expected behavior — logged-out vs. logged-in vs. privileged users.
- **Robustness requirements** set the stress range: how many typos, how much rephrasing, and how stable the result must be.

### Make the surrounding context part of the test · 07:53
- A fair eval for a support agent captures more than a test set: an A2A agent card (a description of what the agent does) sits alongside examples and requirements.
- From an engineering view, these look like **integration tests** — behavior depends on more than an isolated prompt and answer.
- Build a growing collection of them and make assumptions explicit instead of leaving them buried in the implementation.

### Use the specification to direct testing · 08:48
- Safe Intelligence feeds the intended behavior into security testing, because an agent's assigned domain is where it's most willing to talk and where its tools give it the most power.
- A banking agent has consequential infrastructure access in exactly the area it's meant to help with.
- The spec identifies both legitimate activity and the boundaries an attacker might try to cross.
- For robustness testing, the same context defines which input changes should still produce correct answers.

### A capability description is not an evaluation boundary · 09:56
- Prompt-management platforms can record why a test exists, which helps when generating variants.
- Agent cards add capability descriptions, but a skill description alone is not enough to evaluate behavior.
- Example: knowing an agent can book meetings doesn't tell you **who** it may book them for — that boundary decides whether changing the person is a valid variation.
- Generating variations inside these envelopes is hard because the generator needs the task's constraints, not just plausible language.
- Capture the task and its context alongside the benchmark.

### Keep the tests when the implementation changes · 11:08
- **The behavioral specification should outlive the infrastructure that implements it.**
- A team may build on LangSmith or Vertex agents now and move later; its integration, unit, and penetration tests should stay independently runnable.
- That independence enables an improvement loop: run tests → collect results and find robustness gaps → iterate on the system → run again.
- Willmott calls this a kind of "backyard reinforcement learning," explicitly not RL on the model itself — the loop wraps around the agent.
- Next step: an open way to express and exchange specs. He connects this to his earlier work helping write the OpenAPI specification.
- His proposal is to keep agent specs in a GitHub repo, pull pieces into different tools, and version them thoroughly — a proposed approach, not a finished standard.

## Resources

- [A2A specification: Agent Cards and skills](https://a2a-protocol.org/v0.3.0/specification/) — versioned spec for agent discovery and capabilities.
- [OpenAPI specification](https://www.openapis.org/category/announcement/page/3) — the API spec the speaker helped write.
- Adversarial Poetry as a Universal Single-Turn Jailbreak Mechanism — related research on poetic jailbreaks.
- Spec-driven Testing for AI Agents: Now in Early Access — Safe Intelligence's write-up of its test-generation product.
