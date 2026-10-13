# Text Diffusion

| Field | Details |
| --- | --- |
| **Speaker** | Brendan O'Donoghue |
| **Affiliation** | Google DeepMind |
| **Type** | Talk |
| **Date** | 2026-06-04 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=r305-aQTaU0) |
| **Tags** | `evals` `data-and-model-adaptation` `reasoning-and-models` `architecture` `infrastructure-and-deployment` `coding-and-developer-tools` |

## Summary

Brendan O'Donoghue, a research scientist at Google DeepMind, explains **text diffusion**: generating text by starting from a canvas of random tokens and repeatedly denoising it, instead of writing one token at a time. The same principle already powers image and video generation; this talk shows what changes when the output is a sequence of tokens.

The headline benefit is latency. An autoregressive model (the usual next-token kind) is memory-bound when serving one request at a time — it streams the model's weights and cache for every single token. A diffusion model makes a handful of passes over a whole block of positions, moving that data far fewer times. The trade-off is throughput: autoregressive serving batches thousands of requests efficiently, while diffusion re-processes the same positions and hits the compute limit sooner. That serving cost is why diffusion has not landed in large hosted models yet.

Diffusion also enables behavior autoregression cannot: attention can look at later positions, so the model can revise an answer after finishing its reasoning. In the Google I/O demo the opening answer changes 60 → 49 → 39 as the solution develops. The model can also learn to spend more denoising steps on hard prompts — 4 steps for the first 100 digits of pi, 31 for a paragraph on quantum mechanics — and can edit a region in place while keeping its surroundings.

The second half is applications and Q&A. O'Donoghue shows generated Wikipedia-style pages, a generated Reddit-style site, a generated operating-system simulation, and a voice-driven to-do app demo. He answers questions on training data, distillation, model scaling, response-length windows, and where low latency matters most: on-device uses such as phones and robots, where there is no giant batch to share.

## Key Takeaways

- **Text diffusion generates by revision.** Start from random tokens and repeatedly clean the canvas instead of committing to tokens one at a time.
- **Lower latency, lower throughput.** Fewer memory transfers make one response fast; batching many requests is where autoregressive serving still wins on cost.
- **Bidirectional attention allows self-correction.** The model can write a guess, reason, then go back and fix the guess in place.
- **Computation adapts to difficulty.** The model decides when it is done: 4 denoising steps for pi's digits, 18 for FizzBuzz, 31 for a quantum-mechanics paragraph.
- **Editing is local and context-aware.** Fix a bug, add documentation, or insert a middle paragraph using the material on both sides of the edit.
- **Gemini Diffusion was an older preview.** It branched from Gemini 2.0 Flash-Lite, reached about 100,000 people, and was roughly comparable in quality at much lower latency.
- **The architecture is already a hybrid.** Prefill and generation across blocks remain autoregressive; diffusion happens inside each fixed-size block.
- **The obstacle is serving cost, not quality.** O'Donoghue says quality is not the concern and points to on-device uses such as robotics.

## Detailed Notes

### From corrupted sentences to readable text · 00:23
- Training works like image diffusion: take a clean token sequence, replace some tokens with random vocabulary entries at different noise levels, and train the network to repair it. The original sequence stays the recovery target.
- A small Python example makes it concrete: `noise_probability` controls how much of the clean sequence is corrupted. The token IDs are teaching values; the learned part is reversing the damage.
- At inference, start with pure random tokens and repeatedly denoise until readable text emerges. The model fills in information in whatever order it finds useful, not left to right.
- Corruption can happen in continuous space or directly over tokens; his explanation uses the discrete case, where tokens go in and tokens come out.

### Gemini Diffusion and the serving trade-off · 02:03
- Gemini Diffusion was a research preview released roughly a year before the talk, open to about 100,000 people. He says more is coming soon, without a date.
- It branched from Gemini 2.0 Flash-Lite, the main comparator. He reports broadly similar quality — a small edge in code, a small disadvantage elsewhere — at much lower latency, and warns not to fixate on year-old numbers.
- The two generation styles: autoregressive writes one token, conditions on it, then writes the next; diffusion initializes a canvas of hundreds or thousands of tokens and refines the whole block over several passes.
- The main advantage is speed from better use of GPUs and TPUs. Bidirectional attention also becomes possible, since positions are not strictly causal.
- That enables self-corrected generation, adaptive computation, and in-place editing — for example, fixing part of the output and generating a prefix that fits it.
- The main disadvantage is lower throughput for large batches. Autoregressive serving keeps the accelerator busy across many queries; diffusion hits a compute threshold earlier because it does repeated passes on the same data. He notes providers care a lot about throughput now, which is why diffusion is not used in the big hosted models.

### Why fewer passes can be faster · 06:13
- The hardware picture: tensor cores do matrix multiplies with abundant arithmetic capacity, but the weights, activations and KV cache sit in HBM (high-bandwidth memory), a narrow channel that is expensive to widen.
- At batch size one, next-token decoding is memory-bound: to produce each token, the chip streams over the model's data, then does it again for the next token.
- A diffusion pass also streams that data, but updates a whole block of positions. His illustrative calculation: 24 passes to generate 256 tokens means roughly ten times fewer memory transfers.
- The speedup holds only if execution stays truly memory-bound; once arithmetic becomes the bottleneck, fewer transfers no longer map to the same latency gain.
- The browser demo delivered about 2,000 tokens per second including prefill. Longer outputs amortize the initial prompt processing; a one-token answer is dominated by prefill.

### Self-correction: an answer that changes from 60 to 49 to 39 · 08:46
- The demo prompt combines the square root of 81, two-thirds squared and more arithmetic. The correct answer is 39.
- After one pass, the model writes 60 as its opening answer and starts the solution beneath it. The answer is still part of a mutable canvas, not a committed token stream.
- After two passes the answer becomes 49 and the reasoning advances. After three passes the reasoning reaches 36 + 3 = 39 and the model rewrites the early answer to match. Additional passes clean up remaining text.
- Two much larger models struggled with the same prompt: GPT-4o answered 40, worked through the problem, then appended a correction to 39; Gemini 2.5 Flash answered 42 and stuck with it, later treating 36 + 3 as 42.
- O'Donoghue acknowledges that modern thinking models address this, but says diffusion's distinctive move is revising the existing answer rather than appending a correction or accommodating the error.

### Spending more time on hard problems · 11:56
- Dynamic computation means giving the model a larger inference budget. More denoising passes let it inspect and repair an answer even when it already looks nearly clean. Across six internal coding evaluations he reports quality generally improving with the step budget, though not strictly monotonically.
- Adaptive computation is a separate idea: train the model to decide when it is finished, so easy requests consume less of the available budget.
- Preview examples: the first 100 digits of pi took 4 steps (he credits memorization), FizzBuzz took 18, and a paragraph explaining quantum mechanics took 31. In the same time, next-token generation would have produced only four tokens.
- Across task suites, GPQA Diamond (graduate-level science questions) was hard for the target model size and took longer, while MBPP (basic Python problems) was quick. The stopping decision came from the model, not a hand-assigned per-benchmark step count.
- These observations concern the older preview and its evaluation setup, not a current ranking.

### Editing in place, and generating interfaces on demand · 14:20
- Image inpainting is the analogy: cut a region from a photo and a diffusion model fills it using the surrounding pixels and an instruction. Unlike raster-order generation, it can condition on context from anywhere in the image.
- Text demos apply the principle at several scales: fixing a code bug with a local indexing correction, inserting documentation into code, and adding a middle paragraph to a story that must fit both the first and third paragraphs.
- The important property is **context-preserving insertion or revision** — using material on both sides of the edit instead of regenerating every token from the start.
- Latency demos: a Wikipedia-like page whose article text and HTML are generated on demand; a Reddit-style site with invented communities, where a separate image model supplied the pictures (arriving later, he recalls about a minute); and an operating-system simulation where each click generates the next screen.
- His framing: this is not just the same thing faster. Low latency can unlock interactions that were not practical before.

### A voice-driven loop for building an app · 18:37
- An external user demoed building a to-do app by voice through the Gemini Diffusion webpage (not an API).
- The incremental requests: create the app and add 10 random to-dos; add a completed state and mark four random items complete; enable sorting by name and by state; test sorting, add an item, delete several; then switch to dark mode.
- The tested interactions worked, and the demonstrator called the session "15 seconds of work" — his description, not an independently measured duration.
- O'Donoghue's closing product question: what becomes possible when a model responds fast enough to keep up with a user's next instruction? He gives no release date.

### Training compatibility, windows, and where low latency matters · 20:06
- Same data, different algorithms: the team uses the same pretraining corpora and RLHF data as its autoregressive models, but the algorithms need changes. Diffusion models can be distilled, though he names no published technique. Reinforcement learning for reasoning is possible with algorithm changes, and diffusion can be combined with autoregressive models — his team has done so.
- Scaling: larger models tend to need fewer denoising passes for the same output, so the higher cost per pass is partly offset. He gives no scaling curve and says pricing has not been decided.
- Response length: a fixed window generated block by block, autoregressively, is the simplest approach; a prediction head that estimates response length is another option. A fixed window does not fix total answer length — the model just continues through more windows.
- Bounded refinement: the examples all had a maximum step budget, and the model could finish early. That caps how much refinement work a request can consume while still returning easy answers quickly.
- Completed windows are frozen before generation continues. Tokens are revisable inside the active window, but earlier windows are committed — so the architecture is already hybrid: diffusion within blocks, autoregression across blocks. Prefill is unchanged; generation then proceeds in fixed-size blocks (he offers 512, 1,000 or 32 as illustrative sizes).
- Discrete, not latent: a standard prediction head produces logits over token IDs. Every denoising step takes tokens in and returns tokens out; latent-space approaches exist, but he says most current text-diffusion work is discrete.
- Initializing from a smaller model's draft sounds appealing but changes the input distribution: a denoiser trained on noise expects noise, so refining real drafts would require training on that kind of input. He says the team does not usually do this because it adds complexity.
- Where it fits: for now the architectures serve different use cases. On a phone or robot, one local interaction does not share an accelerator with thousands of others, so the batch-throughput advantage of autoregression disappears — and he says text diffusion is already used in some Alphabet on-device applications, including robotics. Asked about frontier quality, he identifies throughput, not quality, as the hosting obstacle and says the research is roughly at that level, without presenting a benchmark. The last question — what if you ask it to generate noise — he has not tried.

## Resources

- [Gemini Diffusion](https://deepmind.google/models/gemini-diffusion/) — official overview and evaluations.
- [GPQA benchmark paper](https://arxiv.org/abs/2311.12022) — the graduate-level science questions behind GPQA Diamond.
- [MBPP](https://github.com/google-research/google-research/blob/master/mbpp/README.md) — Mostly Basic Python Problems, one of the coding datasets mentioned.