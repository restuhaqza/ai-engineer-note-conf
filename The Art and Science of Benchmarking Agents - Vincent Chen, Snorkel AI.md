# The Art and Science of Benchmarking Agents

| Field | Details |
| --- | --- |
| **Speaker** | Vincent Chen |
| **Affiliation** | Snorkel AI |
| **Type** | Talk |
| **Date** | 2026-06-04 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=iNkFlCiij0U) |
| **Tags** | `evals` `safety-and-governance` `leadership` |

## Summary

Vincent Chen, a research fellow and co-founder at Snorkel AI, argues that AI capabilities are improving faster than our ability to measure them. Coding agents feel stronger and model-card scores keep climbing, yet enterprises hesitate to let those agents operate in high-stakes settings. He treats benchmarks as one part of a measurement toolkit — alongside field deployments, red teaming, private human evaluations, and crowdsourced labeling — with open benchmarks playing a special role because they set a direction, not just a snapshot.

Chen draws on Snorkel's Open Benchmarks Grants: a $3 million commitment with more than 120 applications reviewed. He splits the lessons into a science and an art. The **science** covers credible measuring sticks — rigorously validated tasks, deliberately chosen task distributions, model headroom, and evaluation methods that go beyond accuracy. The **art** covers the benchmarks that actually change the field: they carry a thesis about where capabilities are heading, open new research directions, and are built with the researcher's experience in mind.

He uses well-known benchmarks as illustrations — GPQA's multi-reviewer quality control, MMLU's 57-subject taxonomy, ARC-AGI's headroom, τ-bench's policy-aware scoring, Terminal-Bench's bet on the command line, and SWE-bench's family of follow-on work. He closes by proposing three axes for the next wave of benchmarks: environment complexity, autonomy horizon, and output complexity, and invites the audience to turn their own unsolved work into a public target.

## Key Takeaways

- **Capability is outrunning measurement.** Enthusiasm for agents meets hesitation when enterprises weigh high-stakes deployment; the ability to evaluate is falling behind.
- **A good task is answerable and defensible.** Real-world complexity, clear instructions, a verifiable solution, and validation by other domain experts.
- **Distribution is a design choice.** Decide whether the benchmark mirrors typical traffic or deliberately over-weights rare, high-cost failures.
- **Leave meaningful headroom.** A saturated benchmark cannot tell models apart; human-solvable tasks that models fail point to a gap worth measuring.
- **Accuracy is not enough.** Cost, latency, reasoning-trace quality, tool use, and policy compliance can all matter — and must be measured validly.
- **Field-shaping benchmarks carry a thesis.** Terminal-Bench bet on the CLI as a general interface; SWE-bench opened a whole research program.
- **Researcher UX is underrated.** Easy setup, easy task contribution, and reusable signals decide whether a benchmark gets adopted.
- **The next frontier is realism.** Richer environments, longer autonomy horizons, and more complex outputs — including knowing when to stop.

## Detailed Notes

### Capability is improving faster than confidence · 00:29
- Chen is a research fellow and co-founder at Snorkel AI, a frontier data lab with academic ties (Stanford, UW, Wisconsin) and forward-deployed engineers working inside real enterprises.
- The asymmetry he opens with: agents feel more capable and model-card scores keep rising, especially in coding, but organizations hesitate to let them loose in high-stakes finance, insurance, or healthcare settings where results are outcomes, not numbers.
- His claim is not that capability is missing — it is that our ability to measure agents in practice is falling behind where the capabilities already are.
- Closing that gap needs a toolkit: field deployments, red teaming, private human evaluations, and crowdsourced labeling, alongside open benchmarks.
- Open benchmarks matter because the best ones do more than record past progress — they set a goalpost. He cites Terminal-Bench, METR's long-horizon benchmark, and ARC-AGI as guideposts for where the field is going.

### Open Benchmarks Grants and two kinds of benchmark value · 04:12
- Snorkel committed $3 million to open benchmarks through its Open Benchmarks Grants; Chen reports reviewing over 120 applications across academia and industry labs.
- He frames the well-known benchmarks as a chance to honor the "greats," borrowing a Timothée Chalamet reference.
- The examples fall into two complementary groups. **Measurement science** — rigorous individual tasks, deliberate task distributions, robust evaluation — is what makes a score worth trusting.
- **Field-shaping design** — a thesis about future capabilities, new research roadmaps, a strong researcher experience — is what makes a benchmark worth adopting and improving against.

### The science: task quality, distribution, and headroom · 06:34
- Individual tasks need real-world complexity, well-posed instructions, and a verifiable solution. Difficulty does not excuse ambiguity, and validating a task can require more than one expert's judgment.
- GPQA builds quality control into its method. A rough loop: an expert writes a question and its intended answer; another expert attempts it and gives feedback; the writer may revise; a second expert validates; then non-experts attempt it to confirm it is still hard outside the specialty.
- Roles matter — author, reviewer, adjudicator — and so does the chance to revise. A disagreement can expose a defective question, not just a split vote.
- Incentives are part of the design. GPQA's payouts reward questions that experts answer correctly while non-experts struggle, encouraging **answerable difficulty** and borrowing the adversarial spirit of peer review.
- Distribution is a deliberate choice. Start from a clear taxonomy, then decide whether to mirror representative production traffic or to allocate coverage to rare failures whose cost makes them important.
- Self-driving shows the second target: yellow lights, pedestrians and motorcyclists are infrequent, but their importance is not proportional to how often they appear.
- MMLU's endurance comes from a deliberately broad taxonomy of 57 academic and professional subjects — designed in, rather than assumed to emerge from collecting enough questions.
- Headroom is the third axis. A useful frontier benchmark stays unsaturated, exposes real weaknesses, and separates models that otherwise look alike; making tasks merely harder is not enough. Human-solvable tasks are a reference point showing a capability exists that models have not yet acquired.
- Chen ties ARC's progress to the o1-style reasoning push of the preceding 18–24 months. At ARC-AGI-3's launch, frontier models scored below 1% while every task was human-solvable to some degree. That gap is what makes a new result informative — and the scoring concerns human action efficiency in interactive environments, not a percentage of tasks solved.

### The science: measure what actually matters · 11:27
- Accuracy alone does not describe an agent's usefulness. Cost, latency, reasoning-trace quality, intermediate steps and tool use may matter for the capability under test, and can also provide supervision or reward signals.
- The hard requirement is **measurement validity**: a reproducible evaluation must measure what it claims to measure.
- τ-bench makes this concrete. Its agents hold multi-turn conversations with a simulated user and must complete the task while respecting domain policy.
- Example: an agent that books the correct flight but breaks fare-class rules still fails. Completion and compliance are separate conditions, and success requires both.
- He shows a small evaluation predicate returning `task_completed`, `policy_compliant`, and `success`. The predicate exposes the contract; producing reliable checks for each condition is the substantive work. A completion-only score would discard the constraint that makes the booking acceptable.

### The art: a thesis about where the field is going · 12:55
- A field-shaping benchmark adds a research thesis: which capability deserves attention, and why will it matter? The thesis can revisit familiar capabilities under new assumptions or predict how agents will operate.
- Terminal-Bench's bet was that the command line is an interface for general-purpose computer use, not just a place to write code. Chen points to teams at Claude and Codex building broader enterprise abilities on coding and CLI tools as evidence the bet is paying off — and argues that measuring the capability helped accelerate it.
- A benchmark can also act as a roadmap by making new questions easier to formulate. SWE-bench started from a simple idea — use existing repository issues and pull requests to test real coding work — and spawned Lite, Verified, Pro, Multilingual and Multimodal variants.
- Its influence is broader than one leaderboard: it changed which coding-agent questions researchers could isolate. Chen notes that open questions remain about how these workflows apply to vibe coding and what to measure when developers work at a higher level of abstraction than individual patches.

### The art: treat researchers as product users · 15:59
- Researcher UX is part of benchmark design. A benchmark has users who must run models and agents, contribute tasks, extend the environment, and reuse its signals for reinforcement learning or later tuning.
- Every difficult interface adds friction between a research question and an experiment, so making these operations easy is a product requirement, not a finishing touch.
- HELM, built at Stanford CRFM, pioneered a standardized, modular harness for reproducible evaluation across scenarios and models.
- Harbor, shipped with Terminal-Bench 2.0, has become widely used evaluation infrastructure for agent teams.
- The adoption question behind both: how easily can another researcher plug in a system, extend the benchmark, and start improving against it?

### Where the next benchmarks should push · 18:17
- Chen proposes three axes for increasing realism, offered as design dimensions rather than a demand that every benchmark maximize all three.
- **Environment complexity**: capture the dynamic operating conditions that professionals and scientists actually face.
- **Autonomy horizon**: measure how long agents stay reliable, and choose deliberately where a task sits between a copilot and a fully autonomous agent. Longer is not automatically better — the choice matters as much as the duration.
- **Output complexity**: evaluate richer work products with differentiated reward signals, including new artifacts for agent–human and agent–agent interaction.
- Environment complexity is often where agents fail today. A real codebase has org-specific policies, Slack context and screenshots, flaky toolchains, distributed CI, human reviewers with unwritten preferences, and many contributors editing in parallel. Today's benchmarks capture a fraction of that.
- Autonomy horizon is about surviving change. A customer-experience agent may need context from weeks earlier, and while it works, integrations change, product specs shift requirements, and reorganizations move priorities midstream. The environment cannot stay fixed while the agent takes more steps — which also makes it a continual-learning problem.
- More representative work needs more nuanced judgment. Strategic recommendation reports, proposals and roadmaps have no single easily verified answer; what makes one good can depend on organizational context and informed human judgment.
- Trustworthiness adds another requirement: the agent should recognize and communicate its own uncertainty, and sometimes the right move is to stop or ask for more information. Judging that decision expands the notion of a good output beyond a polished plain-text answer.

### Turn your own work into a benchmark · 22:35
- Chen closes by inviting the audience to look at the work already in front of them: which unsolved capability could become a meaningful benchmark?
- At the time of the talk, the Open Benchmarks Grants program was still accepting proposals.
- The aim is to turn practical experience into a public target that helps determine what agents should become capable of next — measuring progress and shaping it at the same time.

## Resources

- [Open Benchmarks Grants](https://benchmarks.snorkel.ai/) — program overview and application entry point.
- [GPQA: A Graduate-Level Google-Proof Q&A Benchmark](https://arxiv.org/html/2311.12022v1) — expert review and validation methodology.
- [MMLU](https://arxiv.org/abs/2009.03300) — the 57-subject knowledge benchmark.
- [ARC-AGI-3 launch](https://arcprize.org/blog/arc-agi-3-launch) — interactive reasoning environments and the human–model gap.
- [ARC-AGI-3 technical report](https://arcprize.org/media/ARC_AGI_3_Technical_Report.pdf) — human validation, environment construction, and scoring.
- [τ-bench](https://github.com/sierra-research/tau-bench) — policy-guided multi-turn agent benchmark.
- [Terminal-Bench](https://www.tbench.ai/) — evaluating agents on terminal-based computer tasks.
- [SWE-bench](https://www.swebench.com/) — software issue-resolution benchmark and variants.
- [HELM](https://crfm.stanford.edu/2022/11/17/helm.html) — standardized, modular evaluation harness.
- [Harbor](https://www.harborframework.com/) — agent evaluation framework in containerized environments.
- [METR long-horizon evaluation](https://metr.org/blog/2025-03-19-measuring-ai-ability-to-complete-long-tasks/) — measuring AI ability on long software tasks.