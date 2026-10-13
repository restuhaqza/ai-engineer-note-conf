# How to Leverage Domain Expertise

| Field | Details |
| --- | --- |
| **Speaker** | Christopher Lovejoy |
| **Affiliation** | Notius Labs |
| **Type** | Talk |
| **Date** | 2026-05-16 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=kfSDc2eVLo4) |
| **Tags** | `evals` `leadership` `healthcare` `architecture` |

## Summary

Christopher Lovejoy — a former doctor who trained at Cambridge, worked in the NHS, and moved into AI in 2018 — argues that winning in vertical AI (AI built for one industry) is less about picking the best model and more about how a company organizes domain expertise. He says his earlier AI Engineer talk reached about 100,000 people, and the most common follow-up was organizational: which expert should I hire, and where should they sit?

His answer is a three-role framework. An **Oracle** both judges output and improves it directly. An **Evaluator** defines and measures quality while engineers act on the findings. An **Architect** designs a system that assesses and improves quality largely on its own. Which one you need depends on whether quality is measurable and where the bottleneck sits.

He grounds the framework in case studies. Granola keeps one Oracle central because there is no objectively perfect meeting note. Tandem, a medical AI scribe, distributes Oracles across specialties and countries. Anterior, which handles US prior authorization, moved from Oracle to Evaluator to Architect as it scaled. Lovejoy describes himself as Anterior's first technical employee, so he lived that progression.

He closes with hiring advice: define a principal domain expert, give them real ownership instead of an advisory role, and hire for breadth around the core expertise.

## Key Takeaways

- **The system for using expertise beats model sophistication.** Frontier models are often good enough; organizations struggle to operationalize expert judgment.
- **Vertical AI is an organizational problem.** The task is turning judgment into product changes, not simply having knowledgeable people nearby.
- **Three roles for expertise:** Oracle (judge and fix), Evaluator (define and measure quality), Architect (design automated improvement).
- **Pick the role from the bottleneck.** Taste-based quality → Oracle. Measurable quality → Evaluator. Measurable but manual iteration too slow → Architect.
- **It is not a mandatory maturity ladder.** Start with an Oracle at small scale and change only when the current process breaks and the next step is feasible.
- **Meeting notes have no single right answer.** Granola's taste-driven output keeps one central Oracle viable even at scale.
- **Scale expertise by distributing ownership.** Tandem gave doctors ownership of specific specialties, countries, and note types.
- **Define a principal expert with authority.** Shared responsibility leaves nobody accountable; advisory-only roles lose people and their accumulated context.

## Detailed Notes

### How does clinical expertise become a better product? · 00:31
- Lovejoy's background: medical training at Cambridge, several years in the NHS, then AI from 2018, including clinical AI at Tandem and prior authorization at Anterior.
- At each company the same challenge appeared: how to bake domain expertise into a product that stands out.
- His thesis: the system for incorporating domain insight matters more than the sophistication of models or pipelines.
- He calls the last mile the hard part — understanding the specific workflows, use cases, and nuances of the customer.
- A "domain-native AI organization" is designed to put the right expertise in the right place so it changes the product.

### The opportunity depends on operational judgment · 02:24
- Vertical AI is commercially attractive because AI moves into labor budgets, not only software budgets. He compares a roughly $50B vertical SaaS market with a multi-trillion-dollar labor market. (He cites Bessemer for the broader thesis, but those measures are not equivalent.)
- Execution is harder: he cites Gartner's finding that at least 50% of generative-AI projects were abandoned after proof of concept by the end of 2025.
- His diagnosis (not measured by that statistic): teams often build AI without understanding the workflow being automated or how experts actually do the work.
- Three recurring mistakes: missing or late expertise, hiring the wrong expert, and poor organizational placement of the right person.
- These map to three questions: why do you need an expert, who should it be, and how should you use them.

### Who assesses quality, and who improves it? · 04:43
- Recognizing good AI output requires judgment, and domain expertise supplies it — sometimes formal (doctors, lawyers), sometimes informal (learned by doing the task).
- He frames building an AI product as a loop: assess current performance, then improve it. The three roles differ in who does each part.
- **Oracle:** the expert judges outputs, uses the product, reviews traces, then changes the application — often prompt edits, but also adding documents, tools, or knowledge sources. The value is the short path from spotting a problem to fixing it.
- **Evaluator:** the expert defines meaningful metrics and builds a way to collect them (a customer north-star metric, clinician reviews of sampled outputs, or LLM-as-judge). Engineers then implement improvements using those findings.
- **Architect:** the expert steps back to design a system that assesses and improves with little human involvement in the loop.

### Choose the role from the bottleneck · 08:48
- Start with measurability: if quality is mostly taste and cannot be adequately measured, use an Oracle.
- If one Oracle cannot cover the work, split responsibility among experts who own different subsets of output.
- If quality is measurable and manual iteration keeps pace, an Evaluator is enough.
- If manual iteration is too slow and improvement can be automated, that creates a need for Architect capabilities.
- Small startups often begin with an Oracle. Oracle → Evaluator → Architect is not mandatory; move on only when the current process breaks and the next step is feasible. Adding more Oracles with distinct ownership is also valid.

### Granola: one Oracle can stay central · 10:51
- Granola, the AI meeting-notes company, recently passed a billion-dollar valuation.
- Lovejoy's example is Jo, a writer and journalist who joined as first employee and wrote all the prompts.
- She researched what makes a good meeting note — reading widely and talking to hundreds or thousands of users — becoming the primary gatekeeper of quality.
- Her loop is complete: review the current product's notes, decide what to improve, and revise the prompts directly.
- It fits because there is no objectively perfect meeting note (human taste matters) and meeting notes are Granola's single core output, so one central review-and-improvement function remains workable at scale.
- This does not mean Granola lacks evals or contributions — it also runs evaluations and has internal tooling letting others contribute to prompts. The Oracle model describes where primary judgment stays concentrated.

### Tandem: distribute ownership across clinical contexts · 12:19
- Tandem's medical AI scribe also generates notes, but from doctor consultations.
- Its first domain expert, Roy — a doctor with McKinsey experience — reviewed notes and updated prompts himself, the same Oracle loop.
- As the product scaled, one person could no longer cover the work, so Tandem created decentralized Oracles: more doctors owning subsets of the product.
- Specialties, countries, note types, and use cases each introduced differences that one centrally managed prompt could not handle.
- A local expert could own a specialty or region, understand its needs, adjust the prompt, and make that version live — with many prompt variations across contexts.
- Medical expertise was essential but did not remove subjectivity: there is still no single perfect medical note. Scaling here meant distributing control while providing platform support to keep variations usable.

### Anterior: scale assessment, then improvement · 14:14
- Anterior's prior-authorization product handles a different workflow: a doctor requests a treatment (say an MRI), and an insurer's nurses or doctors decide whether it is appropriate.
- As Anterior's first technical employee, Lovejoy built the initial code and prompts, reviewed outputs with clinical judgment, and changed the implementation when a decision was wrong — the Oracle loop.
- Growth in customers and variation made personal review insufficient. He moved to Evaluator work: defining metrics and failure modes, building a review dashboard, and hiring clinicians to review outputs.
- Assessment became a system that produced performance information the engineering team could act on.
- Fixing problems then became the bottleneck: organizations interpret policies and rules differently, so manual iteration could not keep up. The output was approval or escalation for clinician review (not final denial), which is measurable against medical evidence, and clinical reasoning was still needed to judge it.
- That pushed him toward Architect work: designing methods for automated improvement. Prior Oracle experience helped — firsthand knowledge of failure modes guided what to measure and change.

### Hire for the actual task and adjacent skills · 17:00
- For an Oracle, relevant expertise means direct experience of the use case. Being a doctor may not qualify someone to shape a medical-coding product; have they done medical coding and know where it goes wrong?
- Prompting and context engineering help and are comparatively learnable. Depth of insight, attention to detail, and customer communication matter, because the expert must discover requirements and translate subtle quality judgments into changes.
- For an Evaluator, add data-science intuition: choosing useful metrics, collecting them reliably, and making them actionable. Supporting skills depend on scale — statistics, industry connections (to recruit reviewers), leadership (to manage a review team), and product management (to work with engineers).
- For an Architect, add experience with LLM-powered products and the levers for improving them; engineering skill helps them implement or steer changes. Domain expertise remains the baseline in every case.

### Give the expert authority and room to grow · 19:35
- Define a **principal domain expert**: one person ultimately accountable for AI quality, with authority to decide.
- This avoids consensus-by-committee, where shared responsibility leaves nobody clearly responsible, and gives the person time to understand performance deeply.
- Give ownership beyond advice: they should be in the room for product decisions, not just reviewing outputs for someone else to act on.
- A cautionary tale: an unnamed company had two senior clinicians but no principal expert. Authority was ambiguous, roles were largely advisory, and progress on the quality-improvement system was slow. Both left after about 12–18 months, and the company also lost the context they carried.
- Hire for breadth without expecting one person to have every adjacent skill; pair the expert with complementary specialists (for example, a statistician).
- Breadth lets the role evolve. Oracle experience is the best foundation for later Evaluator or Architect work, because it reveals recurring failures and what should be measured. The staffing playbook, he notes, is still developing. His slides are on his website.

## Resources

- Gartner on GenAI project abandonment — https://www.gartner.com/en/articles/genai-project-failure
- Bessemer, "The future of AI is vertical" — https://www.bvp.com/atlas/part-i-the-future-of-ai-is-vertical
- Granola's Series C announcement — https://www.granola.ai/blog/series-c
- Anterior prior authorization product — https://anterior.com/prior-authorization-solution
- Chris Lovejoy's website (slides) — https://chrislovejoy.me/