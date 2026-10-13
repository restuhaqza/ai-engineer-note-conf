# Rewiring the State

| Field | Details |
| --- | --- |
| **Speaker** | Eoin Mulgrew |
| **Affiliation** | 10 Downing Street |
| **Type** | Talk |
| **Date** | 2026-05-18 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=ObNKGf9YR0g) |
| **Tags** | `industry-applications` `leadership` `safety-and-governance` |

## Summary

Eoin Mulgrew runs cross-government transformation and the fellowship program in the Number 10 Data Science team (10DS), a small technical unit at the centre of UK government. He sets out a public-service system under strain — long NHS waiting lists, a court backlog, and planning decisions that often miss deadlines — and argues the hard part is organizational, not just technical.

His team's answer is an "insurgency model": a small unit with a political mandate, market-friendly pay, its own technical recruitment, autonomy, and a rule that fellows are hired only from outside government. Mulgrew describes the program as an early experiment, with proof points from a policy-simulation tool, an in-house replacement for a £1.5m legal analysis commission, a delivery "red-teaming" tool, and public dashboards.

He also covers partner teams: the AI Safety Institute, the Incubator for AI, and Justice AI, where engineers embed with parole officers and prison wardens. A final section of audience questions explores sycophancy in policy tools, how the model scales beyond a special exception, AI tutoring, and international collaboration.

## Key Takeaways

- **Government is a complex industry, not one organization.** 10DS brings the best evidence to major national decisions and is now scaling its AI engineering capability.
- **Organizational barriers matter as much as technical ones.** Pay, hierarchy and slow process keep strong engineers out, while some safeguards are genuinely necessary for public accountability.
- **The insurgency model trades exceptions for speed.** Political backing, viable pay, autonomy, technical hiring, and outside-only recruitment let a small team move quickly inside the system.
- **Start from the workflow.** Forward-deployed engineers (embedded with the people doing the work) observe, co-design, implement, and ship in weeks rather than years.
- **Reusable capability beats one-off commissions.** A tool the in-house legal team can rerun stays useful as laws change; a single external analysis goes stale.
- **Judge the source, not just the report.** The delivery red-teaming tool flags optimism bias, over-use of "amber" risk ratings, and whether past mitigations worked.
- **Transparency is part of the work.** Mulgrew says two public government delivery dashboards shipped in two months.
- **The exception must become the norm.** Scaling needs horizontal work on shared processes like police transcription and large DWP/HMRC call centres, not only more single-team pilots.

## Detailed Notes

### A small team against big backlogs · 00:51
- Mulgrew leads cross-government transformation and the fellowship program at 10DS, set up during the pandemic to put the best evidence behind key national decisions.
- He opens with scale: about 7.25 million people on NHS waiting lists, roughly 350,000 backlogged court cases, and only around one in five planning decisions made on time.
- Behind the delays sits a public-sector productivity problem that worsened after the pandemic.
- He cites a Tony Blair Institute estimate of £40bn in potential annual productivity gains from AI in government — an opportunity estimate, not savings already delivered.
- His framing: government behaves more like a complex industry than a single organization.

### Constraints are organizational, not just technical · 03:23
- Government has struggled to build and keep high-performing technical teams; pay makes recruitment and retention hard.
- Hierarchy and bureaucracy slow delivery, and some barriers are real while others are perceived.
- Some safeguards exist for good reasons: public bodies answer to the public and to Parliament, so friction is not always waste.
- Together these conditions can repel exactly the impatient, capable engineers a transformation program wants.
- Changing the whole system is beyond a small central team; the wider reform agenda belongs to Chief AI Officer Kal Beer and the Department for Science and Technology.

### An insurgency model: a team with different rules · 05:23
- The team is built to combine permissions that normally sit apart: a Number 10 mandate, viable compensation, autonomy, and technical selection.
- Fellows come exclusively from outside government; Mulgrew reports a success rate of about 0.7–0.8%.
- Recruits have joined from AI labs, big tech, research institutes, plus Y Combinator founders and serial entrepreneurs.
- Some fellows stay and found their own teams, which is how the model spreads.
- He wants "missionaries, not mercenaries" — public purpose has to sustain people when the work gets hard.
- He stresses that political backing creates room to work; it does not dissolve data silos or make departments cooperate.

### Start with the workflow, then pick the deployment · 08:37
- 10DS matches the delivery approach to the difficulty of the problem.
- **Bounded internal workflow:** build directly with users; some simple cases take days, typically idea-to-implementation in a couple of weeks.
- **Complex departmental problem:** embed engineers in a partner team, sometimes for a long period.
- Number 10's first forward-deployed engineers work alongside policy advisers, lawyers, comms staff and pollsters.
- Their method: observe the workflow and pain points, co-design with the users, then implement and put the capability in people's hands.

### Policy simulation and reusable legal analysis · 09:59
- Policy simulation lets teams test a proposed change before deciding; the shown case is Universal Credit and household finances.
- It supports human analysis rather than replacing it, and Mulgrew says more decisions now get modelling support, and faster.
- The Cabinet Office had planned to spend £1.5m on external lawyers to analyse the whole UK statute book.
- Instead, one engineer embedded with the in-house lawyers for a couple of weeks and built a tool they could rerun themselves.
- The bigger win is repeatability: external analysis would age as new laws appeared and would eventually need redoing.
- Open-sourcing and sharing the tool across government are possibilities he raises, not completed releases.

### Delivery red-teaming and public dashboards · 12:18
- Number 10 receives reports on major projects and manifesto commitments across government.
- The delivery red-teaming tool is like a project management office in your pocket; Mulgrew says it is already used daily.
- Beyond reading a report, it questions the reporting organization: optimism bias, heavy use of "amber" risk ratings, and whether past mitigations worked.
- In-house engineering also supports transparency; he says two public dashboards shipped in two months.
- One is the AI Opportunities Action Plan dashboard, tracking commitments such as compute rollout and AI adoption.
- An unnamed new public service was expected to launch about two and a half weeks later, against a conventional discovery phase that could last a year or more.

### Partner teams: AISI, Incubator for AI, Justice AI · 14:32
- **AI Safety Institute (AISI):** described as the world's first government body for evaluating frontier models; early fellows helped set up its cybersecurity workstream.
- Fellow Dr. Harry Coppock led work on **Inspect**, an open framework for evaluating models and agents that includes sandboxing support.
- **Incubator for AI (now within DSIT):** a spin-out of the fellowship program whose original technical team was largely fellows; it incubates AI solutions for the public sector.
- **Extract:** built with DeepMind on Gemini, it digitizes planning materials including handwriting and hand-drawn maps; rollout to English local authorities is underway.
- **Justice AI:** founded by former fellow Dan James, it embeds engineers with parole officers and prison wardens.
- Its work targets drug flows into prisons, labour-intensive manual processes, and prison safety and security; operational details are not disclosed.

### AI tutoring and the frontline · 17:18
- AI tutoring could widen access to strong educational support regardless of a child's socioeconomic background.
- The team is evaluating frontier models against benchmarks for safe classroom interaction and educational behaviour.
- The illustrated criterion is cognitive load placed on the student — the interaction itself is what gets assessed.
- Justice AI takes the embedded model to the frontline, with engineers beside parole officers and prison wardens.
- A fellow named Will — a Harvard dropout and Y Combinator founder — was shown in his second week holding the keys to HMP Wandsworth.

### Question time: sycophancy, scale, motivation, collaboration · 21:25
- On **sycophancy** (a model telling users what they want to hear): the team red-teams models for it before release and coaches non-technical users on the risk. Little has surfaced so far, Mulgrew says, partly because of that testing — but he does not dismiss it.
- On scaling: organic growth through fellows founding teams is too slow. The bargain was to run a small team under different rules and use results as a proof point, then make those conditions business as usual.
- The ambition for the next 12–24 months is more **horizontal** work: capabilities that apply across shared processes, such as police transcription and the large DWP and HMRC call centres.
- On tutoring: the plan is to set benchmarks and guardrails for schools rather than build a rival product; initial testing used about 70 teachers role-playing pupils, and student motivation is still an open area.
- On international work: he names Tech Force, parts of the US Digital Service, and frequent talks with Singapore, and welcomes contacts from Norway.

## Resources

- Number 10 Data Science (10DS) team and its AI engineering fellowship.
- Inspect — AISI's open evaluation framework, plus its sandboxing documentation.
- Extract — the government's planning-document digitization tool.
- AI Opportunities Action Plan dashboard — public tracking of commitments.
- AI action plan for justice — the Ministry of Justice's approach to AI adoption and specialist recruitment.
