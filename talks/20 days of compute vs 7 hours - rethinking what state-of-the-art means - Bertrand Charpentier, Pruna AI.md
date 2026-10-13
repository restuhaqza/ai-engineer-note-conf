# 20 days of compute vs 7 hours: rethinking what state-of-the-art means

| Field | Details |
| --- | --- |
| **Speaker** | Bertrand Charpentier |
| **Affiliation** | Pruna AI |
| **Type** | Talk |
| **Date** | 2026-06-01 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=hqHC6Z_lXyo) |
| **Tags** | `vision-and-video` `industry-applications` `evals` `reasoning-and-models` |

## Summary

Bertrand Charpentier, cofounder and chief scientist at Pruna AI, takes on a deceptively simple question: which AI model is state of the art? He argues there is no single answer, especially for a specific application. Public leaderboards and internal benchmarks both tend to push people toward the same lazy answer — pick one large foundation model with the highest measured quality — without checking whether that measurement matches the task.

Using image editing as the running example, he shows that leaderboards disagree. Design Arena, Arena (formerly LM Arena), and Artificial Analysis produce different rankings, and the same model can move several places between boards. Even Elo numbers are not on a common scale. Splitting an aggregate editing score into tasks (object removal, background change, text editing) reshuffles the ranking again, and ChatGPT image is first in none of the task-specific views he shows.

He then points to two deeper problems: quality can be bought with compute, and benchmarks are often built from a few thousand samples, not the millions of daily inferences a deployed model might see. His alternative is to evaluate on the actual use case, use multiple meaningful metrics, and plot a **Pareto frontier** of quality against latency or price — which usually reveals several defensible "state-of-the-art" choices rather than one winner.

## Key Takeaways

- **"State of the art" is application-specific.** A single leaderboard winner is a starting point, not a final answer.
- **Leaderboards disagree.** The top model and relative rankings differ across Design Arena, Arena, and Artificial Analysis — disagreement suggests candidates may be roughly equivalent.
- **Elo values aren't comparable across boards.** Numbers only mean something next to the comparison that produced them.
- **Split the task before ranking.** A product built around object removal should read object-removal results, not an overall editing score.
- **Most models lose many battles.** Charpentier reports most models in the comparison lose at least 40% of preference battles — a relative loss rate, not a failure rate.
- **Manual inspection has two biases.** Your taste (evaluator bias) and your few examples (sample bias) both skew the choice.
- **Understand the metric.** CLIP scores gave inconsistent, tightly packed rankings; task-specific metrics like text rendering separated models more clearly.
- **Quality costs compute.** His estimate: ~20 days, ~$5,000, and ~556 kWh to evaluate ~26,000 images — versus about 7 hours for a Pruna alternative.
- **Use a Pareto frontier.** Several models can be state of the art once you weigh quality against latency or price.

## Detailed Notes

### Which model is best for your application? · 00:15
- The question matters for research and for deployed products, but "state of the art" is a fuzzy concept.
- Two common routes: check a public leaderboard, or run an internal evaluation.
- Applied naively, both lead to the same shortcut — choose one large foundation model with the top measured quality.
- For image editing, the easy move is to open Design Arena and pick ChatGPT image, the leader in his snapshot.
- That yields a reasonable first candidate cheaply, but says nothing about your users, their requests, or their operating conditions.

### The winner changes with the comparison · 02:20
- Put Arena (formerly LM Arena), Design Arena, and Artificial Analysis side by side, and the editing rankings disagree.
- The top entry changes, and so do relative positions: Hunyuan moves from tenth on Artificial Analysis to fifth on Arena in his snapshot.
- Duplicate entries and models that appear on only some boards add noise.
- Elo ratings aren't interchangeable — some boards sit around 1,100–1,300, others use different ranges.
- He reads large rank disagreement as a reason to treat candidates as roughly equivalent, not as proof that they all perform alike.
- His advice: don't trust a single leaderboard; look at several and investigate the evaluation context.

### Match the tasks and samples to deployment · 04:15
- An aggregate editing score blends different capabilities: removing objects, changing backgrounds, editing text, and more.
- Split those apart and the rankings change again; ChatGPT image is not first in any task-specific ranking he shows.
- Models are trained with different emphases, so a use-case-specific board is the right starting point.
- **The application determines which comparison matters.**
- Ask what was sampled: his example uses a few thousand samples per model, versus millions of daily inferences for some Pruna models.
- Preference rankings come from battles where users pick a preferred output; he reports most models lose at least 40% of their battles.
- More samples help only when they cover your subset and reproduce the final use-case conditions.

### Two prompts expose two kinds of bias · 07:24
- Internal evaluation often means trying a few prompts and inspecting outputs, or running an automated benchmark and taking the highest score.
- Charpentier shows three images and asks the audience to pick a favorite; someone asks what the requirement was and what the prompt was.
- This separates two judgments: which image looks nicer, and which image follows the instruction.
- Audience members choose differently, showing the limits of one person's taste.
- He repeats the exercise with a second prompt, and some people switch their preferred position — while the model positions stay fixed (Seedream left, FLUX.1 middle, P-Image right).
- **Evaluator bias:** your visual preferences may not match other users'. **Sample bias:** a few prompts may not represent the real requests.
- Inspection is still useful for intuition; human evaluation gets stronger with many evaluators and broader examples.

### Understand what the metric measures · 10:28
- Automation doesn't remove the need to interpret the test.
- Ranking eight models by CLIP score across datasets, the order changes and the numerical gaps stay small.
- Those scales (0–1 or 0–100) shouldn't be treated as a universal normalization, and tiny differences may not resolve which model is better.
- Start by understanding what each metric measures, then use more than one.
- Task-specific metrics help: in his text-rendering evaluations, Z-Image ranks first and P-Image second, with clearer separation on the stated 0–1 scale.
- **Running a benchmark is not the same as understanding its result.** Otherwise a high score just moves the selection problem into the metric.

### The workload behind 20 days vs 7 hours · 12:18
- Quality can be purchased with compute, a limitation shared by public rankings and internal benchmarks.
- He estimates an ~26,000-image evaluation for ChatGPT image at 62 seconds per image.
- That yields roughly 20 days of accumulated generation time, about $5,000 in cost, and ~556 kWh of energy.
- He's unsure which leaderboard supplied the battle count, so treat this as his workload estimate, not a reconstruction.
- The energy comparison comes from his own Strava estimate: about 400 marathons (an illustration, not a standard conversion).
- A Pruna alternative is described as generating and editing images in under a second.
- For the same evaluation count: about 7 hours, $265, and roughly 4 marathons — the estimates preserve his numbers and don't claim equal quality.

### Several models can be state of the art · 14:27
- A **Pareto frontier** shows the decision without collapsing it into one score.
- Put efficiency (generation latency or price per image) on the X-axis and quality (e.g., Elo) on the Y-axis; lower cost and higher quality are better.
- A model is dominated if another is at least as good on both objectives and better on one; the frontier holds the remaining choices.
- His red frontiers contain roughly three or four models, with quality clustered around 1,100–1,200 but efficiency varying a lot — he cites ~20x faster generation as an illustrative difference.
- That doesn't make frontier models interchangeable; it means several choices are defensible depending on how much you value the quality gain.
- Make the vertical axis task-specific: for text rendering, plot text-rendering quality against efficiency.
- He describes optimizing FLUX.2 and other FLUX models with BFL; the optimized models run faster while staying on the frontier — optimization changes the available tradeoffs.

### From evaluation to optimized and compressed models · 16:02
- Benchmarking stays useful when it reflects the application: enough relevant samples, realistic conditions, multiple meaningful measurements, and explicit efficiency costs.
- Charpentier argues this favors smaller models tuned to particular preferences rather than automatically picking a large general model.
- Pruna serves "preference models" behind endpoints — image and video models with runtimes of roughly 1–5 seconds.
- For teams optimizing their own models, he points to the open-source Pruna package and efficiency research materials.
- On compression, he names two ways to change model components: **quantization** (per-module treatments rather than one uniform setting) and **pruning** (removing unimportant components).
- Image and video generation typically uses ~20–50 denoising steps with the same backbone, making repeated denoiser work a big target.
- **Distillation** and **caching** reduce that repetition — he illustrates cutting 50 backbone computations to 20 or even 4.
- Those counts are examples of the opportunity, not guaranteed results; he distinguishes public caching algorithms from internal ones used behind Pruna's endpoints.

## Resources

- [Pruna model optimization framework](https://github.com/PrunaAI/pruna) — open-source caching, quantization, and pruning.
- [Pruna and BFL's FLUX.2 optimization](https://www.pruna.ai/blog/flux2flex-3-faster) — collaboration report with hardware-specific latency gains.
- [CLIPScore paper](https://arxiv.org/abs/2104.08718) — the reference-free image metric discussed.
- [How Arena Works](https://arena.ai/how-it-works) — anonymous model battles and user votes.
- [Artificial Analysis image benchmarking methodology](https://artificialanalysis.ai/image/methodology) — definitions of the image benchmarks cited.
