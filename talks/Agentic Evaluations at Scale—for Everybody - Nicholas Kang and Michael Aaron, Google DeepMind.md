# Agentic Evaluations at Scale—for Everybody

| Field | Details |
| --- | --- |
| **Speaker** | Nicholas Kang, Michael Aaron |
| **Affiliation** | Google DeepMind |
| **Type** | Talk |
| **Date** | 2026-05-25 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=Ubwb6NzegyA) |
| **Tags** | `evals` `agent-engineering` `creative-and-generative-media` |

## Summary

Nicholas Kang and Michael Aaron, both on Kaggle at Google DeepMind, argue that AI evaluation is broken in three ways: benchmarks are scattered and go stale, results are not transparent or reproducible, and too few people decide what gets tested. Kaggle's answer is to treat evaluation as a community platform rather than a leaderboard.

They describe four products aimed at different parts of the problem: hackathons to recruit contributors, agent exams to lower the effort of testing an agent, Game Arena to escape benchmark saturation through head-to-head play, and Benchmarks to let anyone build, run, and share evaluations.

The strongest thread is who gets to define a good test. Kang's example is a wastewater-plant engineer who built a benchmark from 20 years of experience — knowledge no lab would collect. The counter-thread, from Aaron, is that moving from models to agents makes the object of evaluation murky: how much of a score belongs to the model, and how much to the harness around it?

## Key Takeaways

- **Discovery is a bottleneck.** More than ten benchmarks can appear a day, and publication does not guarantee that a leaderboard stays maintained.
- **The same benchmark can measure different systems.** Kaggle published a benchmark whose rival lab improved after rerunning it with its own API compaction — a setting Kaggle had not enabled.
- **Coverage is a separate problem.** Evaluations are written by a small group (roughly 30,000 AI researchers) compared with the people AI should help (roughly 30 million technical professionals).
- **Domain experts fill the gap.** A wastewater engineer built a benchmark from experience that exists nowhere else and that labs are not motivated to collect.
- **Make participation practical.** Contributors need tools (dataset hosting, model access), readable write-ups, and human judging of creativity — which is still hard for agents.
- **Standardized agent exams lower the barrier.** Paste one line, let the agent take an exam, get a score; hundreds were evaluated in the first week of a quiet MVP launch.
- **Game Arena avoids saturation.** Head-to-head games with Elo-style ratings always produce a winner and a loser, though game choice shapes which abilities show up.
- **Know what is under test.** A credible comparison needs transparent configuration and identifiable model versions; harness behavior can move coding scores by a lot.

## Detailed Notes

### Three ways agent evals break · 00:35
- Kang introduces himself as the product manager for Kaggle Benchmarks and its agentic evaluation work; Aaron is a Kaggle software engineer focused on evaluations and benchmarks.
- Kaggle is described as a community of more than 30 million users, with about two years of work in generative AI evaluation.
- Problem one: evals are scattered, decentralized, and go stale fast. Kang says ten-plus appear daily, and finding them means hours on arXiv.
- Even a full-time benchmark reader cannot keep up. After publication, authors move on and leaderboards stop tracking the models people actually use.
- Problem two: evals are not always transparent, accessible, or verifiable.
- Problem three: coverage. He draws the world's knowledge as a big circle with small circles inside for the people who currently write evaluations.

### Same benchmark, different configuration · 02:53
- A published score does not reveal what configuration produced it — both model settings and how the benchmark is orchestrated matter.
- Model-release charts can look like clean comparisons while hiding enough setup that nobody else can reproduce them.
- Kang's example: Kaggle published a benchmark with one AI lab, and a competing lab objected to the results.
- The competitor reran it and got substantially better numbers. In Kang's account, the competitor used compaction supplied through its own API, which Kaggle had not enabled for any model in its comparison.
- The lesson: the benchmark name alone does not identify the experiment.

### Widen who writes the tests · 03:54
- Kang contrasts roughly 30,000 AI researchers (attributed to a Google AI search) with roughly 30 million software engineers, data scientists, and other technical professionals.
- His point is the mismatch between who builds evaluations and who AI is meant to help.
- Without a benchmark for a capability, developers have no measured target, which can deepen the unevenness of models that excel in some areas and struggle in others.
- The concrete case is a wastewater-plant engineer in Turkey with 20 years of experience, who saw fatal incidents from failures to follow safety protocols.
- He built a benchmark to ask whether AI could help him do his job and prevent similar incidents, using data from his professional experience.
- Kang calls that dataset original and proprietary — knowledge not on the web and outside labs' immediate economic priorities.

### Turn participation into usable evals · 05:49
- Kaggle's response spans four products: hackathons, agent exams, Game Arena, and Benchmarks.
- Hackathons channel effort toward a defined problem, with enough constraints to stay relevant but room for creativity; results are meant to be open source.
- Kang points to a hackathon with DeepMind's AGI (artificial general intelligence) team, based on a paper proposing ten cognitive faculties. Participants build benchmarks for five of them.
- Practical obstacles: with 1,000 globally distributed participants, they need dataset hosting and model access; paying for five API keys can exclude people with fewer resources.
- Submissions need readable write-ups so others can learn from them and build on them.
- Judging innovation and creativity still needs human experts, and even getting experts to agree is hard work.

### A quick exam before an agent goes live · 09:35
- Kang originally called these "standardized agent tests" but renamed them over a trademark issue.
- The workflow is short: paste a one-line prompt into an agent, let it take the exam, and get a score you can compare on a leaderboard.
- At the time of the talk it was an experimental MVP launched the week before.
- The motivation is a gap: labs and enterprises use tools like Braintrust, while many people send consumer agents into real workflows without testing them first.
- Kang references another conference talk reporting 1,100 security advisories filed that morning, and floats safety-focused exams as a baseline before an agent gets control of an inbox or an Amazon account. He frames this as a proposed direction, not a guarantee.
- Design tension: an exam that is too hard may never finish, while one that is too easy gives no signal. The useful middle delivers signal within a run people complete.
- He reports 500-plus agents evaluated in the first week with little promotion; after a post on Moltbook, agents began sharing results and even an exam-prep course appeared.

### Game Arena: replace the ceiling with opponents · 11:58
- Aaron explains that benchmarks saturate: as models improve they approach a fixed ceiling that stops distinguishing them.
- Game Arena uses player-versus-player competition instead, with Elo-style ratings, so there is always a winner and a loser and the comparison stays unsaturated.
- Game selection determines which capabilities become visible: Werewolf for deception, Poker for randomness, deception, and risk-taking, and Chess as an established setting.
- Poker exposes behavioral differences a general score would hide — Aaron says Grok frequently goes all in, and some newer models play worse because they are more risk-averse.
- The team iterates on the game, confirms models can actually play it, and spends substantial time refining prompts for fairness. The harness is open source and inspectable.

### The real cost of a credible comparison · 15:11
- Aaron says a Poker evaluation needed about 400,000 hands for statistical significance, with many model turns inside each hand, so inference cost grows with the turns.
- He does not give a confidence threshold or stopping rule, so the count describes that evaluation, not a general requirement.
- Bradley–Terry pairwise methods help reduce the number of games, but reaching a credible comparison quickly remains an open problem.
- Engagement is a second constraint: watching models play is fun at first and repetitive later. He proposes a prompt hackathon where the community supplies prompts and competes on a leaderboard for how well they make a model play.
- Continuity is a third: old models disappear, new ones arrive, and a hosted endpoint may not honestly reveal which model is behind it.
- Longitudinal comparison depends on knowing which system actually played, not just keeping an endpoint name in a results table.

### From assertions to benchmarks — and what's under test · 16:36
- Kaggle Benchmarks targets open community evaluation, not a specific application's production code, though its workflow will look familiar to anyone writing production evals.
- You generate an output and check whether it satisfies an expectation. A simple example: ask what gets wetter as it dries, and check the answer contains "towel."
- Checks can be explicit Python assertions; when a rule cannot capture the judgment, the platform also supports LLM judging.
- Checks group into a task, run against the models a user selects; tasks aggregate into a benchmark, such as the wastewater one.
- The richer demo is a task recreating an SVG from xkcd, with assertions on whether an SVG was produced and whether the text is correct; side-by-side comparison lets readers inspect the artifacts.
- On incentives, a product team has an immediate reason to test its model, while a community contributor must first find a question worth testing; hackathons, Kaggle points, and medals help but do not remove the effort.
- Moving from models to agents obscures the object of evaluation. Aaron cites an unverified Morph LLM post reporting six frontier models within a couple of points on SWE-bench Pro, while coding performance varied by 22% depending on the harness.
- The takeaway: an agent evaluation needs a clear account of the system under test. Otherwise a model comparison may really be measuring harness differences, and rapid releases make well-defined experiments hard to repeat.

## Resources

- **Game Arena harness** — Open-source game harness with a chess demo and documentation. *(mentioned)* — https://github.com/google-deepmind/game_arena
- **OpenSpiel** — Game environments and algorithms for reinforcement learning, search, and planning. *(mentioned)* — https://github.com/google-deepmind/open_spiel
- **Kaggle Benchmarks Python library** — Define evaluation tasks, call models, and check outputs. *(mentioned)* — https://github.com/Kaggle/kaggle-benchmarks
- **Measuring Progress Toward AGI: A Cognitive Framework** — DeepMind's proposed framework for evaluating cognitive faculties. *(mentioned)*
- **Standardized Agent Exam instructions** — How to take Kaggle's agent exams and retrieve results. *(related)*
