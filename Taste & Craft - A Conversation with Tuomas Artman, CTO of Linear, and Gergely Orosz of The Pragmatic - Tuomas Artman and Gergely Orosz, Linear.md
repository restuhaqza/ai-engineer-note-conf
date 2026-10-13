# Taste & Craft: A Conversation with Tuomas Artman, CTO of Linear, and Gergely Orosz of The Pragmatic Engineer

| Field | Details |
| --- | --- |
| **Speaker** | Tuomas Artman, Gergely Orosz |
| **Affiliation** | Linear |
| **Type** | Talk |
| **Date** | 2026-04-21 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=wjk0ulMAkbc) |
| **Tags** | `developer-workflows-and-testing` `coding-and-developer-tools` `enterprise` `leadership` |

## Summary

Tuomas Artman, cofounder and CTO of Linear, talks with Gergely Orosz of The Pragmatic Engineer about what happens when shipping a feature becomes easy. When an agent can build almost anything on request, the hard part shifts to deciding what deserves to exist. Artman argues that taste, restraint, and craft become more valuable, not less.

The pair draw on their shared time at Uber, where hypergrowth and revenue metrics pushed feature shipping hard. Artman uses that to explain a subtle risk: quality rarely shows up in a single metric or A/B test. Instead, users drift away slowly when a competitor feels better to use, and you may never see the moment you lost them.

Linear's answer is a set of habits. Every engineer must find and fix a small quality problem each week ("Quality Wednesdays"), and every reported bug is assigned immediately and repaired fast (a "zero-bugs policy"). Both are designed to build attention, not just to clean up the product.

The talk closes on culture: a paid week-long work trial to hire product-minded engineers, open access to customer conversations, and a prediction that most engineers will become more product-focused over time.

## Key Takeaways

- **Easy implementation makes restraint harder.** If every request can be built instantly, teams risk piling on features until the product becomes confusing.
- **Feature factories predate AI.** Uber's hypergrowth already showed how shipping pressure can crowd out quality; AI just makes the pressure easier to act on.
- **Quality is hard to measure and easy to lose.** There is often no clean metric or A/B test for polish, so users leave gradually rather than in one visible drop.
- **Quality Wednesdays build attention.** Each engineer finds and presents one small fix a week, which trains people to notice problems elsewhere.
- **A zero-bugs policy is about ownership, not perfection.** Every report gets assigned and either fixed quickly or explicitly declined.
- **Agents can build the effect but not choose it.** They lack taste and a felt sense of time, so motion and interface judgement still need a human.
- **Hire for product sense.** Linear uses a paid full-week trial to see whether someone can drive a product end to end.
- **Give engineers customer reality.** Open customer Slack channels and recorded, tagged meetings expose the evidence behind requests.

## Detailed Notes

### When every request becomes a feature · 00:42
- Artman opens with the danger of agents that can implement anything immediately: you may ship every request and every passing idea without thinking.
- He quotes Steve Jobs's idea that good products come from saying no to 999 things and yes to one.
- Historically, engineering difficulty acted as a natural gate. Teams had to think about a feature before starting, because building took real time.
- Orosz pushes back: feature factories existed long before AI. Artman agrees but says the mechanism is different now.
- At Uber, a winner-takes-all market against Lyft meant shipping constantly, fighting fires, and scaling at all costs. Artman says he does not want to repeat that hypergrowth.
- Now, a small team — or one capable person — can match a competitor's feature set. Artman sees tasteful, high-quality software as the way to stand out.

### Finding the problem behind the request · 03:52
- Orosz frames the challenge with Claude Code and **Opus 4.5**: if the team can ship faster, what should Linear ask them to build?
- Artman's answer keeps deliberate selection. Linear still weighs each feature against the user experience and says no to many requests.
- The product process rarely ships a request as submitted. It moves from a proposed feature to the underlying problem.
- The steps: talk to customers about what they are trying to do, group requests that share a cause, design one solution for that group, then work out the experience.
- AI can summarise and cluster incoming requests, which organises the evidence.
- But choosing which problem to solve, and how the result should work, still takes human time.

### Faster bug fixes, and the quality trade-off · 05:38
- Bug inflow is roughly constant, but fixing bugs has become easier. Artman reports that 10% of bugs get a single-shot AI fix, a pull request, and automatic landing with no engineer involved.
- He predicts that share could approach 100% over the next few years — clearly a forecast, not today's result.
- The useful split: delegate work with a clear, well-understood behaviour and little design judgement; keep attention for "what should this product do?"
- As a counterexample, Artman criticises Claude Code itself. Anthropic says Claude wrote all of its functionality, and Artman says it shows: bugs appear quickly, some interactions feel slow, and behaviour can be hard to understand.
- He links those rough edges to competition with OpenAI and the risk of another winner-takes-all market. Shipping volume can crowd out the work that makes existing features dependable.

### What Uber's revenue metrics could not measure · 07:49
- Artman recalls five major metrics at Uber. Between them he and Orosz name revenue, trips taken, ride quality, and time from signup to first trip; they do not reconstruct the full set.
- Revenue dominated the incentives. Measuring ride quality still did not settle whether the app felt pleasant to use.
- Uber Pool illustrates the gap. A cheaper ride can attract demand and raise revenue even if the surrounding experience is rough, especially when no rival offers an equally cheap option.
- Engineers might care about polish, yet the business metric rewards the new capability without isolating the quality of its execution.
- Artman remembers quality being explicit when he joined Uber in 2012. In an early pull request, he changed the map margins around the ETA pole, and a senior iOS engineer measured it and found it two pixels off.
- The lesson was the attention: someone was willing to defend a detail most users would never consciously notice.

### Quality Wednesdays: making small imperfections visible · 11:56
- As Uber grew and revenue incentives took over, shipping more features became the stronger organising force.
- Artman's theory: quality becomes commercially visible once competitors offer similar features at similar prices. Users compare occasionally and may find the alternative nicer.
- The danger is not being leapfrogged but being **slowly overtaken**, a slow drift that no single A/B test captures. That is his reason to invest in quality before a metric demands it.
- Linear's **Quality Wednesdays** are a recurring, remote show-and-tell. Orosz describes about 25 engineers presenting fixes in roughly 37 minutes, each under two minutes.
- Fixes range from a one-pixel tweak to backend efficiency. To date, Artman estimates Linear has fixed 2,500–3,000 small details.
- The practice started at an offsite three or four years earlier. The trigger was an interaction rule: highlights should appear instantly on hover but fade out over about 150 milliseconds.
- To teach the attention, Artman had the team inspect one small view-options menu for an hour. They found about 35 problems in a single menu.
- The key rule is that each engineer must **find** a problem rather than be assigned one. This keeps people looking, and Artman says engineers who look for small issues also introduce fewer regressions.

### The zero-bugs policy · 16:22
- Linear's zero-bugs policy is about immediate ownership, not a claim that software never has bugs.
- When a bug is reported, an agent immediately assigns it to the likely creator or someone familiar with that area.
- That bug becomes the engineer's highest priority — the first thing to pick up from My Issues.
- The engineer either fixes it or explicitly decides against a fix when the cost outweighs the impact. Artman's example: a hard bug affecting one in 100,000 users.
- His reasoning: bug inflow is constant. If you defer fixes, you eventually do the same work months later while the product feels worse.
- Linear paused new features for about three weeks to clear its backlog. After that, bugs are typically fixed within seven days, often in two or three hours.
- The customer payoff: report a problem, get an email a couple of hours later, refresh the browser, and it's fixed.
- An assigned bug does **not** count toward Quality Wednesday. That is a defect; Wednesday still needs an independently discovered improvement, so both habits survive.

### Agents can build it, but they can't feel it · 19:43
- Orosz pushes back: agents can write unit tests and run feedback loops, so should they not produce better code and interfaces?
- Artman separates those abilities from **taste**: judging whether an interface serves its purpose and understanding how a person feels while using it.
- His example is time. An agent inspecting screenshots or the DOM sees a timeless view of an interaction.
- It can know that one second is shorter than two, and it can recite advice about caching or hosting, but it cannot feel the frustration of a two-second wait after a click.
- Animation shows the same gap. Linear design engineer Emil shared a test where agents built pop-ups, button highlights, and moving elements, then he refined them by hand.
- The functionality existed in both versions. What changed was the feeling of the motion — an easing choice or a duration too long or short. Implementing an effect and choosing the right effect are different skills.

### Product engineering: hiring, customer access, and the road ahead · 22:21
- Linear hires people who already want to build beautiful, high-quality software. Most engineers are product engineers, though the sync engine and infrastructure create real technical challenges.
- The hiring process includes a paid, full-week work trial for every employee, usually on a greenfield project. Some trial work ships at the end of the week.
- The evaluation is broader than implementation: can the candidate work out what is needed and drive the work from start to finish?
- Orosz notes the cost to candidates who can't take a week off. Artman accepts that exclusion and treats unwillingness as a sign of low desire to join.
- Compared with Uber's conventional five- or six-interview process, Artman reports very few hiring misses at Linear.
- Customer context is readily available: open Slack channels with big customers, recorded customer meetings, and tagged, searchable moments so an engineer can hear what customers said about a feature.
- This matters more as customers diverge from the team. Linear began as a tool its own engineers built for themselves; it now serves enterprises with needs the team does not share.
- Prediction: if AI keeps improving, engineers who simply move data around will be less in demand, while those who understand customers and can judge a good feature or experience will stay valuable — becoming "mini PMs."
- Closing advice: move closer to customers, build something yourself, and ship it so you learn from your first outside users. For interface design, he recommends Apple's Human Interface Guidelines.

## Resources

- **Quality Wednesdays** — Artman's account of building a weekly habit of finding and fixing small quality issues.
- **Why we committed to a zero-bugs policy** — Linear's approach to clearing its bug backlog and its repair rules.
- **Train Your Judgement** — interactive animation comparisons about easing, movement, and timing.
- **Apple Human Interface Guidelines** — Apple's interface-design guidance, recommended as a foundation for good UX.
