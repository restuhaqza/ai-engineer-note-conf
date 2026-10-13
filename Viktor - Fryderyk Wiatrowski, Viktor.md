# Viktor — AI Coworker That Lives in Slack

| Field | Details |
| --- | --- |
| **Speaker** | Fryderyk Wiatrowski |
| **Affiliation** | Viktor |
| **Type** | Talk |
| **Date** | 2026-05-11 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=ohKt066uFhg) |
| **Tags** | `agent-engineering` `safety-and-governance` `leadership` `enterprise` |

## Summary

Fryderyk Wiatrowski, co-founder of Viktor, describes an AI coworker that lives
entirely in Slack. It has no web app to open: it joins channels and threads,
answers direct messages, and uses company tools. He cites 3,000 integrations,
plus the ability to build a connection when one is missing. The pitch is an
employee, not a product you visit.

The talk traces how the team got there. In 2023 they tried browser agents, which
clicked and typed through web pages but were too slow and unreliable. Better
models and a first agent loop led to an email agent, where an incoming message
supplies both a trigger and the context. Viktor brought that idea to Slack in
February and saw strong adoption.

Most of the talk is about what breaks when an agent serves a whole company.
Shared integrations raise permission questions, Slack's many interaction modes
must be flattened into one context, and tone and proactivity matter as much as
tool use. Wiatrowski closes with three practical requirements for anyone wanting
to build their own coworker.

## Key Takeaways

- **An AI coworker lives where the team already works.** Viktor has no separate
  app; it participates in Slack channels, threads, and DMs.
- **One connection can serve the team.** A shared integration replaces every
  employee configuring their own, which changes memory and permission needs.
- **Shared context needs boundaries.** Growth- or executive-channel material must
  not leak into other teams, and DM access should depend on membership.
- **Slack changes latency expectations.** A ten-minute task feels frustrating in
  a web app but fast when it arrives like a message from a colleague.
- **A Slack conversation is not one thread.** DMs, threads, edits, reactions, and
  deletions all have to become one linear context for the model.
- **Tone can decide which model you ship.** Users reacted badly when the team
  A/B tested a cheaper model with strong tool use, preferring Opus's personality.
- **Proactive help must be earned.** Viktor has challenged weak A/B conclusions
  using analytics, but doing that on day one alarmed security teams.
- **Three pillars:** get work done, know the company, and be friendly.

## Detailed Notes

### An employee where the team already works · 00:15
- Viktor launched in February (2026) as an experiment with, he says, zero
  expectations of growth; adoption was immediate and hard to keep up with.
- It lives in Slack alongside teammates — channels, threads, and direct messages
  — with no separate web app.
- It has access to the tools you have access to: he cites 3,000 integrations and
  says it can build its own connections when one is missing.
- The ambition is horizontal context: unlike a human employee, it can hold
  knowledge across departments.
- His example: a CMO would work better with the codebase. He frames the goal as
  "PhD-level understanding" across the company; the real requirement is access
  to the information and tools to act across roles.

### From browser steps to an email-triggered agent · 02:08
- The team started building AI employees in 2023, after ChatGPT appeared. With
  no tool calling or strong code generation, browsers looked like the universal
  interface — most business apps already had one.
- The first agent, **JAI**, took a DOM snapshot (a DOM is a page's element tree),
  minified it losslessly, and used it plus the user's goal to pick the next
  action.
- It worked for about three to five steps in 2023, with roughly 60% reliability.
  He does not say whether that figure is per step or per sequence. It was state
  of the art on WebArena, a common agent benchmark.
- Reliability and speed made a product hard: a function call returns at once,
  while a browser agent might spend a minute before failing. He notes browser
  agents have improved a lot since.
- With **Sonnet 3.5**, the team built its first agent loop and moved into email
  with **Jace**.
- An incoming email is both trigger and context; the agent connects tools, then
  replies with a draft, a tool action, or both, with approval gates where needed.
- The arc: navigate interfaces, react to incoming work, then join the company as
  an employee.

### Shared integrations create shared responsibility · 05:16
- **OpenClaw** is the personal-agent comparison: each employee connects their own
  integrations and runs their own agent.
- With Viktor, one person can connect an integration for the team; Viktor
  inherits its permissions, which can be tuned.
- That convenience creates a harder state-management problem: memory that gets
  cluttered for one user faces far more activity for a team — he offers "100
  users, 100 times faster" as a thought experiment, not a measurement. He says
  the team solved it but does not describe how.
- Hierarchy adds conflict: users give contradictory instructions, and not
  everyone may see the same information. Context must stay separated, so growth-
  or executive-channel material does not leak into engineering or support.
- In a DM, access to growth-team context should depend on whether that person is
  on the growth team.
- His point: shared context needs boundaries at retrieval and use, not just a
  bigger memory. The talk states the requirement without giving a
  channel-level authorization design.

### Slack changes what waiting feels like · 08:21
- Two reasons for Slack. First, it makes the agent feel like a human employee you
  speak to where your team already is.
- Second, it matches how powerful agents behave: hard tasks can take around ten
  minutes.
- In a dedicated web app, switching context and waiting ten minutes feels
  frustrating, especially beside a roughly thirty-second ChatGPT exchange.
- Ask a coworker in Slack to build an app and receiving it ten minutes later
  feels surprisingly fast. The computation did not get faster; the interaction
  became asynchronous in a familiar way.
- **The interface changes your reference point for latency,** so you can delegate
  and move on instead of waiting on the current screen.

### A Slack conversation is not a single thread · 10:07
- A typical agent web app presents one conversation. Slack has DMs, public
  channels, threads, emoji reactions, edits, and deletions.
- All of these can be inputs, but they must become a linear context the model can
  read.
- **Message changes carry intent.** He treats a deletion as "stop or no longer
  relevant" and an edit as something to respond to. Slack reports a deletion; it
  does not cancel an agent task by itself.
- Continuity is subtler: someone starts a thread in a DM, forgets it, and sends
  a new top-level message. A human reads that as the same conversation; an agent
  that opens a fresh task each time loses the connection.
- His fix: when a DM arrives, inspect previous messages and roll relevant context
  into the existing conversation.
- The talk sketches a TypeScript context builder that merges prior and incoming
  messages by ID, keeping each message's thread ID and text.
- Selecting the right prior messages is a separate job; concatenating every DM
  just replaces missing context with clutter.

### Model selection includes personality · 11:55
- Tone mattered more than expected. Viktor uses **Claude Opus 4.6** as its main
  model.
- The team tested **GPT-5.4**, praising its tool calling and code generation and
  its lower price, and expected to switch.
- Users reacted angrily during the A/B test and preferred Opus. Personality was
  one reason; he calls Viktor's Opus personality a bit sassy, and allows the
  preference could depend on the architecture.
- **Proactivity:** Viktor can suggest workflows, and it can intervene when a
  discussion outruns its evidence.
- Example: a growth team concludes an A/B option is winning; Viktor checks
  PostHog (an analytics tool), finds the result is not statistically significant,
  and supplies a calculation. No experiment data or method is given.
- The same initiative can alarm people. Security teams reacted badly when Viktor
  began DMing and joining threads right after being added.
- His rollout advice: earn trust with a few users first, then expand.

### Cloud execution and one company connection · 14:28
- Compared with Claude Code and Claude Cowork, Viktor runs in the cloud; Cowork
  works on the desktop and needs your computer open. Shared connections remove
  friction: for a hundred-person company with a twenty-person growth team, one
  person can connect the Meta ads integration instead of twenty.
- Personal connections create ambiguity for proactive work — with several
  accounts, which should the agent use?
- He frames Viktor as a hire, not a tool, then tells a story that tests the
  metaphor.
- At a large US e-commerce brand, an administrator installed Viktor and connected
  personal Gmail as the first team integration. Teammates then discussed his
  emails with Viktor, and he complained his data was leaking.
- Wiatrowski's response: would you give a newly hired employee access to your
  personal email?
- The incident drove a product change: **scoped integrations**, so a connection
  need not be shared. A user can connect a personal account and invoke it in a DM
  or publicly; enforcement mechanics are not detailed.

### What a coworker needs to become useful · 17:02
- He invites the technical audience to build their own version and gives three
  requirements.
- **Gets work done:** he considers current models capable, and recommends
  Pipedream for connecting integrations.
- **Knows the company:** Slack supplies context, but the application must use it
  well. Distribution also means Slack's approval process, which he calls
  difficult and tedious.
- **Is friendly:** the team must enjoy working with the agent; mutual liking keeps
  it a recurring participant in company work.
- Connecting tools alone does not create a coworker — execution, context, and
  social behavior belong in the same product.
- His bigger bet: every company will employ AI workers. He links it to Leibniz's
  17th-century work on calculators, quoting Leibniz on freeing people from "the
  labor of calculation."
- It is a vision for what to automate, not a claim about what Viktor does today.
  He closes with an offer for the room: scan a QR code, sign up, and get $100 in
  free credits; Viktor can be removed at any time.

## Resources

- *Viktor* — product overview and signup for the AI coworker.
- *Jace's AWA 1.5 WebArena report* — the team's 2024 browser-agent benchmark.
- *WebArena* — the original benchmark paper.
- *Pipedream integration toolkit* — developer guidance for connecting agent
  tools to APIs.
- *Viktor's current security controls* — integration permissions and related
  documentation.
- *Handling Slack message deletions* — event fields and client-state behavior.
- *OpenClaw* — the personal AI assistant used as a comparison.
- *GPT-5.4 announcement* — release details and pricing.
- *Claude Opus 4.6 announcement* — release details and pricing.
