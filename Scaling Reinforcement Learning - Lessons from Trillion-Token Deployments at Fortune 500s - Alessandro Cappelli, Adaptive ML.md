# Scaling Reinforcement Learning: Lessons from Trillion-Token Deployments at Fortune 500s

| Field | Details |
| --- | --- |
| **Speaker** | Alessandro Cappelli |
| **Affiliation** | Adaptive ML |
| **Type** | Talk |
| **Date** | 2026-05-12 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=X6NShR2ccOg) |
| **Tags** | `data-and-model-adaptation` `architecture` `agent-engineering` `infrastructure-and-deployment` |

## Summary

Alessandro Cappelli, co-founder and chief customer officer at Adaptive ML, argues that the hard part of enterprise AI is not the demo. Teams build a convincing MVP, then stall, because prompting or instruction-tuning gives them no systematic way to turn each new defect into a measured improvement. He frames the MVP as the "first mile" and production as an ongoing loop.

His answer is **reinforcement learning (RL)** — training a model by rewarding good outcomes rather than copying fixed examples. RL lets you fold client feedback, business metrics, and environment rewards back into the model. Compared with **supervised fine-tuning (SFT)** — training on example input/output pairs — he says RL can reach the same performance with a smaller model, which changes serving economics.

That matters at enterprise scale. Cappelli cites AT&T spending millions to summarize customer transcripts, and speech-to-speech support needing responses under half a second. Smaller specialized models cut serving cost and latency, and give the business ownership of a system trained on its own data.

For agents, RL fits naturally because it was built for learning in an environment. If an environment already exists, plug a model into it; if not, mock the tools and a simulated user. Environment plus reward doubles as a synthetic-data pipeline: score candidate trajectories, keep the good ones with rejection sampling, and bootstrap training. Humans stay in the loop by defining rewards and rubrics, not by running endless annotation campaigns.

## Key Takeaways

- **The "last mile" is a myth:** getting to an MVP is the first mile; production is a continuous lifecycle.
- **Prompting and SFT have limits:** they steer behavior but do not systematically convert each defect into measured improvement. RL is his proposed mechanism.
- **RL can match SFT with a smaller model**, which matters when a repeated task is served at enterprise volume.
- **Four benefits of specialization:** lower serving cost, better task performance, lower latency, and ownership.
- **Latency is a constraint, not a nice-to-have:** speech-to-speech support wants under ~0.5s (ideally ~0.33s), which he ties to ~10B models like Gemma, Mistral, and Qwen.
- **Agents raise the stakes:** more tokens, actions that change external systems, and less room for error.
- **Environment + reward = data:** score trajectories, apply rejection sampling, and use the result to bootstrap training.
- **Humans define the reward:** small feedback sets refine a prompted **LLM judge** (a model used to score outputs), while thousands of production examples train a reward model.

## Detailed Notes

### What happens after the demo works? · 00:23
- Adaptive ML builds an RL operations platform that helps enterprises such as AT&T, Manulife, and CCS build, evaluate, and serve specialized models.
- Cappelli traces the focus to his team's work training **Falcon**; they concluded RL was the missing ingredient between an open model and production-grade behavior.
- He cites a 95% failure-to-production figure for generative-AI pilots (no source given) and presents it as his framing, not an established rate.
- The failure is a maintenance problem: with prompting you edit the system prompt and introduce new defects; with SFT you revise the dataset, which can be expensive and never ends.
- His stronger claim — that RL is the *only* systematic way to close the loop — is the thesis of the talk, not a demonstrated exclusion of alternatives.

### Specialization changes the serving economics · 03:52
- Prompting, SFT, and RL all steer model behavior; Cappelli compares how much useful behavior each yields at a given model size.
- He says RL can match SFT performance with a smaller model, moving the decision from buying general capability to training the capability the task needs.
- At scale, serving cost decides whether a useful feature becomes common. He says AT&T spends millions summarizing customer–agent transcripts (no period specified).
- Latency is a hard constraint for real-time use: for speech-to-speech support, half a second is an upper bound and about a third of a second is preferable.
- He associates that target with models around the 10B range, naming **Gemma**, **Mistral**, and **Qwen** — deployment targets, not hardware-independent limits.
- The last benefit is control: a model trained on your data can be evaluated and released on your schedule, instead of shifting when an upstream provider updates.

### Train agents where their actions have consequences · 06:48
- Summarization, classification, and OCR already raise cost and reliability questions; agents add longer workflows and actions that modify external systems.
- Errors can touch more than the text: an agent may read customer data or change a database, so accuracy standards rise.
- He asks what happens if an agent uses ten times the tokens of a summarization workflow — an illustrative cost scenario, not a measured result.
- RL fits agents because it was designed for acting in an **environment** and learning from the responses to those actions.
- Two starting points:
  - **Reuse an existing workflow** — he describes Manulife already having an agent environment, so a model such as Qwen 3.5 can be plugged into it.
  - **Build an environment** — mock the tools and, for conversational tasks, provide a simulated user, which another LLM can play.
- The reward decides which interactions count as success: a business outcome, a KPI, helpfulness, or adherence to tone and vocabulary guidelines.

### The environment becomes a source of training data · 09:33
- Companies often have plenty of business data but little agent training data. What is missing is a **trajectory**: the sequence of decisions, tool calls, and responses that completes a task.
- Once an environment and reward exist, they form a synthetic-data pipeline:
  1. Run candidate interactions in the environment.
  2. Score the trajectories with the reward.
  3. Use **rejection sampling** to keep the good ones.
  4. Bootstrap initial training with the retained set.
- The reward supplies the selection criterion, so you no longer depend only on people writing successful demonstrations.
- Existing customer conversations can improve the environment: give transcripts to the mock user, or train it on them, so it reproduces repeated requests and messy exchanges.
- His medical-supply example includes panicked callers; desired behavior there might be escalation to a human or contacting emergency services — illustrations of behavior to represent, not a validated protocol.

### Put human judgment into the reward definition · 11:24
- **RLHF** (reinforcement learning from human feedback) became widely known through ChatGPT, but keeping humans involved need not mean repeated annotation campaigns, which he calls expensive and hard to sustain.
- Different requirements need different reward sources:
  - **Mechanical check** (code runs, syntax is valid) — a directly testable property.
  - **Business outcome** (customer-support containment) — whether the model completes the task.
  - **LLM judge** (tone, business requirements) — open-ended behavioral criteria.
- For the medical-supply customer CCS, he names **containment rate** — the share of calls handled end to end — as a business objective.
- Humans define the judge's rubrics and system prompt, and pick scenarios that resemble real interactions.
- He estimates that rubric and scenario work at minutes to hours rather than weeks of repeated annotation, so the human role shifts to specifying and checking the evaluation standard.

### Operationalize evaluation, training, and serving together · 13:21
- Accelerating the lifecycle is not just faster training; you also need to evaluate, observe, and find defects before and after deployment.
- He presents **Adaptive Engine** as an integrated platform to observe, train, and serve models together — the capabilities he describes, not independently verified results.
- The starting model remains a choice, with the Gemma, Mistral, and Qwen families as foundations.
- The catch is that RL is genuinely hard. Cappelli describes a common **PPO** (Proximal Policy Optimization) setup as orchestrating four language models at once, noting this is one pattern, not a universal requirement.
- Adaptive Engine lets customers define rubrics while the platform handles the training machinery, including prebuilt recipes so customers need not implement algorithms such as **GSPO** (Group Sequence Policy Optimization) themselves.

### From a single feedback event to a learned reward · 15:50
- An audience question cites Cursor's online RL work on Tab completion, where implicit signals like accepting or rejecting a suggestion arrive one at a time instead of as several sampled outputs.
- Cappelli splits the answer by how much feedback is available. With roughly 10–20 early human examples, his team refines the judge's description of good and bad behavior.
- Early on, the judge can be a prompted large model (he names Qwen 235B as an example). With thousands of production examples, his team trains reward models to scale that feedback into further training.
- The progression: humans clarify a prompted evaluator, then production data supports a learned evaluator.
- For implicit feedback, he says the interpretation is use-case-specific: build the reward model accordingly and compare approaches against a defined evaluation, choosing empirically rather than assuming replay beats direct training from a single event.

## Resources

- [Improving Cursor Tab with online RL](https://cursor.com/blog/tab-rl) — how accepted and rejected suggestions become a training signal.
- [Group Sequence Policy Optimization](https://arxiv.org/abs/2507.18071) — the original GSPO paper.
- **Create a training recipe** (documentation) — Adaptive ML's guide to configuring models, datasets, and training.
- **How AT&T Saves Millions with Specialized Language Models** (article) — a vendor case study.
- **The GenAI Divide: State of AI in Business 2025** (paper) — a public mirror of the preliminary report.
