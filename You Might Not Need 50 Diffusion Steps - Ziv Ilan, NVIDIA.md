# You Might Not Need 50 Diffusion Steps

| Field | Details |
| --- | --- |
| **Speaker** | Ziv Ilan |
| **Affiliation** | NVIDIA |
| **Type** | Talk |
| **Date** | 2026-06-16 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=gHs5ZiY80PM) |
| **Tags** | `data-and-model-adaptation` `infrastructure-and-deployment` `architecture` |

## Summary

Diffusion models — the kind behind image and video generation — build their output by repeatedly denoising it, usually over 20-50 steps. That repetition is why generation is slow. Ziv Ilan, from NVIDIA's Paris AI Labs team, argues model quality is already good enough for many uses; the missing piece is latency, which is what turns a demo into something an interactive app or an enterprise can actually run.

He walks through three ways to speed things up. **Quantization** reduces the precision of the math. **Caching** avoids recomputing parts of the image that barely changed between steps. **Distillation** trains the model to produce a good result in far fewer steps. Each attacks a different part of the cost, and Ilan stresses they stack rather than compete.

For distillation he separates two families: one that makes the student copy the teacher's denoising path, and one that only asks the student to match the teacher's final output distribution. The second is more common and higher quality today, and it can cut 50 steps to 4, 8, or even 1. He also introduces FastGen, NVIDIA's open-source framework for organizing this post-training work at the scale of 20-40B-parameter video models.

The talk ends with a practical order for experimenting: try the cheap interventions first, and only take on distillation when the remaining latency target justifies the extra compute, time, and expertise. In the Q&A he says distillation does not need top-end GB200 systems — Hopper/H100, B200, and B300 are all options — and that the real data question is whether your use case needs its own dataset.

## Key Takeaways

- **Diffusion is iterative.** A good image or video takes 20-50 denoising steps, and that repetition is the core latency problem.
- **Three techniques, one toolkit.** Quantization, caching, and distillation target different costs and can all be combined.
- **Quantization is the easiest start.** Lower precision saves memory and can speed inference, but diffusion is attention-heavy, so gains are smaller than for LLMs.
- **PTQ vs QAT.** Quantize after training for simplicity, or account for reduced precision during training to protect quality.
- **Caching reuses near-identical work.** TeaCache judges when a step can be skipped; newer chunk-based methods recompute only the regions that changed.
- **Distillation keeps the model size but cuts the steps.** It trains the student for a short generation process, rather than stopping a normal sampler early.
- **Match the output distribution, not just the path.** Distribution-based distillation lets the student find its own shorter route and is higher quality today.
- **Order experiments cheaply first.** Quantization → multi-GPU and caching → distillation; distillation is the most impactful and the most expensive.

## Detailed Notes

### From good images to usable latency · 01:12
- Unlike autoregressive LLMs (which emit one token at a time), a diffusion model denoises the whole image or video repeatedly — typically 20-50 steps.
- Recent models made generation useful: **FLUX.2** for images, **LTX 2.1** and Google's **Nano Banana** for video. But usefulness now depends on latency: how fast the first frame appears and how fast you produce 720p or 1080p.
- Ilan's team works with frontier model builders. He frames diffusion serving as less mature than the autoregressive LLM world around vLLM (a popular LLM serving library): many ideas transfer, but they must fit diffusion's computation pattern.
- The goal is real-time image and video generation for robotics world models, computer games, and content creation. Three tools: **quantization, caching, distillation**.

### Quantization: make each computation cheaper · 03:29
- **Post-training quantization (PTQ)** reduces precision after training — simpler to adopt, but preserving generation quality is harder than for LLMs.
- **Quantization-aware training (QAT)** accounts for reduced precision during training, which costs a training process but protects quality.
- Diffusion models are attention-heavy, so quantization helps less than it does for LLMs and vision-language models — but it is still a low-hanging way to use Blackwell and newer hardware.
- In NVIDIA's FLUX.2 work with Black Forest Labs, they use **dynamic scaling** (ranges computed as needed) rather than purely static ranges, so it adapts to the data. **NVFP4** (a 4-bit format) mixes per-tensor scaling, which may be static or dynamic, with per-block scaling computed for blocks of 16 elements.
- Two adoption routes: the open-source TensorRT-LLM VisualGen implementation, or pre-quantized checkpoints on Hugging Face if you will not fine-tune or add LoRA adapters.
- Benefits: lower memory (so a model can fit on consumer or lower-end data-center GPUs) and better inference performance. Attention stays a target — he points to recent FP4-attention research.

### Caching: reuse work that barely changed · 06:34
- Autoregressive serving leans on the **KV cache** (stored attention keys and values) because each step appends one token. Diffusion does not append tokens the same way, so that reuse pattern does not transfer directly.
- The opportunity is different: consecutive denoising steps often compute nearly the same thing, so some of that work can be skipped.
- **TeaCache** estimates how much the output changed, using timestep-modulated inputs as a proxy, and reuses computation when the change is small. It avoids computing the full output just to discover it was unchanged.
- Newer methods reuse at the **chunk** level. His classroom analogy: most of the audience sits still (reuse that region) while the presenter moves (recompute his region). A threshold decides how much change is acceptable.
- **The threshold is a quality choice, not just a speed knob.** Reusing too aggressively can visibly damage the output.
- Tooling: TensorRT-LLM VisualGen exposes caching with a flag and a threshold (beta docs list `cache_config`, `cache_backend`, and `teacache_thresh`). Caching also exists in vLLM Omni and SGLang Diffusion.

### Distillation: fewer steps, same size · 09:22
- Distillation here is not about shrinking the model. The parameter count stays the same; the goal is **step distillation** — train the student to reach the same quality in far fewer denoising steps.
- Example: instead of 50 steps, 4 or 8, sometimes even a single step, while aiming to keep quality.
- This is not early stopping of an ordinary sampler — the student is trained specifically for the shorter process.
- Ilan cites potential **10×-200×** performance improvements (without specifying a benchmark baseline, hardware, or quality metric). A GTC demo in San Jose used two distillation techniques to reach real-time generation.
- Streaming output would open many more interactive use cases, which is why many labs are chasing it.

### Two ways to distill: path or distribution · 11:07
- Both families use a teacher and a student, but they ask the student to reproduce different things.
- **Trajectory-based:** the student follows the teacher's denoising path step by step.
- **Distribution-based:** the student only needs to match the teacher's final output distribution, so it may take its own shorter route.
- Ilan says distribution-based is more common and higher quality at the time of the talk. The two can be combined — he cites a recent FastVideo release whose hybrid approach kept quality and made training more stable.
- Distillation is the hardest of the three because it is post-training: it needs data, reliable convergence, extra compute, time, and expertise, and recipes are still evolving.

### FastGen: distilling at model scale · 12:56
- **FastGen** is infrastructure, not another distillation algorithm — an open-source framework from NVIDIA Research that organizes many distillation techniques so teams can experiment and scale.
- Motivation: video diffusion models are already 20B, 30B, and 40B parameters, with hundreds of billions expected. At that size, sharding work across GPUs becomes part of the training problem.
- FastGen handles that machinery so practitioners can focus on output quality and the recipe they want to use.
- Training data is optional; open-source data works up to a point. If your target distribution differs a lot from general-purpose material, Ilan recommends fine-tuning on your own data.
- The payoff can be lower compute as well as lower latency: at GTC they reached near-real-time or real-time video generation on a single Blackwell B200 GPU, depending on output quality. (The talk does not specify resolution, frame rate, or a quality evaluation, and this is an inference result, not a training budget.)

### Combine the optimizations incrementally · 15:13
- The techniques stack: distillation cuts the number of steps, quantization cuts the cost and memory of each computation, and caching avoids repeated work. Multi-GPU execution and context parallelism add more options.
- Suggested experiment order (easiest first): start with quantization and stop if it meets your needs; if not, add multi-GPU, context parallelism, and caching; finally take on distillation when the remaining latency target justifies it.
- Note this order differs from a full optimization pipeline, where distillation can come before quantization and caching.
- He frames distillation as the most impactful and the most demanding option.
- Open-source tooling covers the **Wan**, FLUX.2, and **LTX2** model families, with more being added. The invitation: try it and contribute, so diffusion serving can mature like the LLM/vLLM ecosystem.

### Q&A: hardware and data for distillation · 16:48
- Audience question: if GB200 systems are hard to get, what compute do fine-tuning or distillation need, and how big should the dataset be?
- **GB200 is not required.** Ilan names Hopper/H100, B200, and B300 as options. Distillation needs less compute than pre-training, but it still needs real resources — not a single arbitrary instance.
- Model size drives the requirement: small video models (2B-4B parameters) need much less compute. A single-GPU inference result should not be read as a training budget.
- On data, the first step is **knowing how to evaluate.** Compare what a general-purpose dataset produces against data specific to your use case.
- General demos work without a special dataset; a specialized domain (his example is protein generation) needs more targeted data. He gives no fixed dataset size.

## Resources

- [NVIDIA's FLUX.2 optimization walkthrough](https://developer.nvidia.com/blog/scaling-nvfp4-inference-for-flux-2-on-nvidia-blackwell-data-center-gpus/) — NVFP4 scaling, TeaCache, and combined inference optimizations
- [TensorRT-LLM VisualGen docs](https://github.com/NVIDIA/TensorRT-LLM/blob/main/docs/source/models/visual-generation.md) — supported diffusion models, quantization, and caching flags
- [TeaCache](https://github.com/ali-vilab/TeaCache) — training-free diffusion caching
- [NVIDIA FastGen](https://github.com/NVlabs/FastGen) — open-source framework for diffusion distillation
- [FastVideo](https://github.com/hao-ai-lab/FastVideo) — video generation with inference optimizations and distributed training
