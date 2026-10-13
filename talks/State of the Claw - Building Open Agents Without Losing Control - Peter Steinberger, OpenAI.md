# State of the Claw: Building Open Agents Without Losing Control

| Field | Details |
| --- | --- |
| **Speaker** | Peter Steinberger |
| **Affiliation** | OpenAI |
| **Type** | Talk |
| **Date** | 2026-04-17 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=zgNvts_2TUE) |
| **Tags** | `safety-and-governance` `leadership` `agent-engineering` `reasoning-and-models` |

## Summary

Peter Steinberger, creator of OpenClaw (an open-source personal agent) and now at
OpenAI, gives a candid status update on the project's explosive growth and the
pressures that came with it. OpenClaw grew faster than any project in GitHub
history: roughly 30,000 commits, nearly 2,000 contributors, and close to 30,000
pull requests in about five months. Popularity, he argues, does not automatically
create sustainable ownership.

A large part of the talk is about security. Steinberger says he was "DDoS-ed" by
security advisories — over 1,100 total, around 16.6 a day — and argues that a
severity score is not the same as real-world exploitability. Many reports look
agent-generated, and triage is a heavy human burden. He explains the "lethal
trifecta" — an agent with access to private data, exposure to untrusted content,
and the ability to communicate externally — and the boundaries that reduce risk.

The final section is a moderated AMA with swyx. Topics include keeping OpenClaw
independent through a foundation, letting users own their data and choose models,
agent personality and "taste," why he is skeptical of a fully automated "dark
factory," and the skills he thinks humans should double down on: taste, system
design, and the ability to say no.

## Key Takeaways

- **Growth is an organizational problem.** Huge contributor counts do not remove
  dependence on a few people making consequential decisions.
- **Keep governance independent.** The OpenClaw Foundation is meant to keep the
  project open, model-agnostic, and not controlled by one company.
- **Severity is not exploitability.** A maximum CVSS (a 0–10 severity score) can
  describe a configuration almost nobody uses.
- **Triage is the real bottleneck.** Many advisories look agent-generated, but
  humans must still read each one; fixes are rare, and rushed fixes break products.
- **Know the lethal trifecta.** Access to private data plus untrusted content plus
  the ability to send data out is the core agent risk.
- **Set boundaries by role.** A personal agent should take instructions only from
  its owner; a group-chat agent should be sandboxed; a team agent should know only
  what the team may know.
- **Own your data.** He prefers a personal agent that keeps data local and calls a
  stronger model only when needed.
- **Taste, iteration, and "no" are the human edge.** You cannot fully specify good
  software up front, and one feature is rarely the problem — many together are.

## Detailed Notes

### Growth creates an organizational problem first · 00:52
- OpenClaw is five months old and reports around 30,000 commits, close to 2,000
  contributors, and nearly 30,000 pull requests.
- Steinberger calls it the fastest-growing project in GitHub's history, and says
  velocity is not slowing.
- He took on two roles: he joined OpenAI and also helped create the OpenClaw
  Foundation, so he now effectively has two jobs.
- Running the foundation is "a company on hard mode": all the responsibilities,
  but volunteers you cannot simply direct.
- One goal is improving the bus factor — how many people can make commits. It is
  improving slowly.
- Collaborators now come from NVIDIA, Microsoft, Red Hat, Tencent, ByteDance, and
  others. He deliberately avoids staffing only with OpenAI people, to prevent the
  impression that OpenAI took the project over.
- He frames the foundation as "Switzerland": neutral, so OpenClaw stays open and
  works with any model, including local ones. Setup was nearly done, held up by
  banking administration.

### Security severity is not the same thing as practical exposure · 03:42
- He reports 1,142 security advisories, about 16.6 per day, 99 marked critical;
  roughly 469 published and 60% closed.
- For comparison, he says the Linux kernel gets about 8–9 a day and curl has
  around 600 reports total; OpenClaw has roughly twice each.
- His rule of thumb: the more loudly a report is labeled critical, the more likely
  it is slop.
- He hooked NVIDIA's NemoClaw (a sandbox and security layer for OpenClaw) into
  Codex Security and says it found about five ways to escape the sandbox in half
  an hour.
- Attack-surface categories cited: remote code execution (RCE — running attacker
  code on your machine), approval bypass, code injection, and path traversal.
- One example reached a maximum severity only through a reduced-permission device
  model he says almost no one uses, and required actively fighting the default,
  local-only gateway setup.
- Other threats are real and separate: a misleading look-alike package that
  delivers a rootkit, and a supply-chain incident where OpenClaw was hit through a
  dependency (MS Teams or Slack using Axios) even though it does not use Axios
  directly.
- He criticizes fear-driven research, citing a paper that described OpenClaw in
  detail but skipped its security guidance, and says reports should disclose the
  configuration used to produce a finding.
- Triage is a huge time cost. Reports rarely arrive with fixes, and when they do
  the fix is often bad; rushed fixes reliably break the product.
- Most help came from companies, especially NVIDIA engineers doing full-time
  triage and hardening.

### Agent safety starts with access boundaries and realistic threat models · 10:50
- The structural risk he calls the **lethal trifecta**: access to your data,
  exposure to untrusted content, and the ability to communicate. Any powerful
  agent with all three is at risk — it is not unique to OpenClaw.
- Recommendations: a personal agent should accept instructions only from its
  owner; a group-chat agent must be sandboxed; a team agent should hold only what
  the team is allowed to know.
- If anyone can talk to your agent, they can reach anything the agent can do,
  including exfiltration of data.
- On prompt injection, he says frontier models now handle one-off untrusted content
  (a website or an email clearly marked as untrusted) much better than before.
- Repeated, unlimited access to an agent is still exploitable: an attacker who can
  bombard it with content may eventually find a way.
- Small local models with no defensive training, combined with a browser or email,
  worry him; OpenClaw warns users about that configuration while still supporting
  many model choices.
- He points to trust/reputation systems — privileges that grow as trust is earned
  — as a promising direction, and mentions Simon Willison's "dual LLM" idea of
  using separate models for untrusted content.
- No single safeguard resolves every case; model capability, access controls,
  untrusted inputs, and user configuration all interact.

### Personal agents should preserve user control and adapt to their environment · 19:40
- A core motivation is **data control**: he dislikes hosted tools that gain broad
  access to his email through connectors, and prefers data that stays under his own
  control.
- His model hierarchy keeps data local and sends only a little to a top-tier model
  when needed, while preserving the option of local or alternative models.
- One reason is practical: obtaining enterprise-style integrations (for example
  Gmail) is slow for a startup, whereas a user-operated agent can act through the
  access the user already has.
- He acknowledges the hacker flavor, and that this can work around the silos large
  companies build.
- The long-term vision is ubiquitous, context-aware interaction rather than a phone
  chatbot: talk from any room, use the nearest screen via a canvas feature, and
  eventually use devices such as glasses.
- He imagines separate personal and workplace agents that can talk to each other
  within trust and authorization arrangements both the person and the employer
  accept — concrete mechanisms left as future work.
- Personality matters because the agent lives in social channels. An early WhatsApp
  integration felt wrong because it was too wordy and unlike friends' messages, so
  he iterated on tone and writing style.

### Automation increases the importance of taste, iteration, and system design · 24:03
- He once ran nearly ten coding-agent sessions at once when models were slow, and
  now runs around five or six as output got faster.
- He frames many parallel agents as a workaround for waiting time, not an ideal:
  it is unnatural to work on six things at once.
- He is skeptical of the **dark factory** — fully specifying a product up front and
  building it with no human involvement.
- Good software is iterative: build a step, use it, notice what feels wrong, and
  change the next prompt. An agent does not reliably know which direction serves
  the project's vision.
- Pipelines help with bounded tasks, but auto-merging PRs risks pulling the
  product in conflicting directions.
- His definition of taste starts low: does it "stink like AI"? He points to
  machine-written prose, generic agentic UI, and a recognizable feeling.
- Higher-level taste lives in small, delightful details — for example OpenClaw's
  occasional roast messages — which you miss when you prompt only at a high level.
- Architecturally, OpenClaw moved from a sprawling codebase to **extensions and
  plugins**: memory, a wiki, "Dreaming," and other capabilities install or swap
  independently, so experiments do not all queue up in the overloaded core.
- Dreaming is an early attempt to reconcile session logs and memories, inspired by
  how sleep consolidates memory — garbage-collecting and moving some to long-term
  storage while dropping others.
- The remaining human work is system design, asking the right questions, and
  saying no, because many individually plausible changes can combine into an
  incoherent, hard-to-maintain system.
- His closing point: agents in a large codebase see only a slice, so humans must
  supply hints and context to help them do their best work. The bottleneck is still
  thinking and big-picture judgment.
