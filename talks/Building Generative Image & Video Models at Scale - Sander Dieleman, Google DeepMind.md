# Building Generative Image & Video Models at Scale

| Field | Details |
| --- | --- |
| **Speaker** | Sander Dieleman |
| **Affiliation** | Google DeepMind |
| **Type** | Talk |
| **Date** | 2026-04-21 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=xOP1PM8fwnk) |
| **Tags** | `architecture` `data-and-model-adaptation` `creative-and-generative-media` |

## Summary

Sander Dieleman, a Google DeepMind research scientist on the generative media team behind **Veo** and **Nano Banana**, gives a behind-the-scenes tour of what it takes to train large image and video generators. The talk focuses on **diffusion models** — the dominant approach for audiovisual generation, in contrast to the autoregression that dominates language — and walks through eight stages, from data curation to user control.

His first message is that the neural network is only one part of the system. Data curation is essential and still underrated, and a big chunk of the work is choosing a **representation**: raw pixels are too large at scale, so a learned autoencoder compresses images and video into a smaller latent grid that stays convenient to model.

He then builds diffusion from first principles: gradually corrupt data with noise, learn a denoiser to reverse it, and sample by taking small corrected steps. The blurry single-step prediction is not a bug but a direction toward plausible outputs, and a Fourier view explains why the procedure naturally generates coarse structure before fine detail.

The back half covers architectures (U-Nets and transformers), **guidance** — the trick that trades diversity for quality and lets these models outperform their size — distillation into fewer sampling steps, and the conditioning signals that let people steer results.

## Key Takeaways

- **Data curation may beat model tweaking.** Dieleman argues time spent improving the data can be a better investment than tuning the model or optimizer.
- **Compress data before modeling it.** A learned autoencoder turns huge pixel tensors into a smaller latent grid while preserving the structure a generator needs.
- **Diffusion works by corrupting and reversing.** Add noise gradually, learn a denoiser, then sample by stepping toward its predictions.
- **A single denoising prediction is blurry on purpose.** The model predicts a region of plausible images, so you take small steps and re-predict rather than jumping.
- **Diffusion is "spectral autoregression."** Because noise hits high frequencies first, generation naturally proceeds coarse-to-fine — an approximation, not strict ordering.
- **Guidance is nearly universal.** Amplifying the difference between prompted and unprompted predictions improves quality at the cost of diversity; everyone leaves it on.
- **Distillation shortens sampling, it does not shrink the model.** Consistency models predict the trajectory's endpoint, though one-step generation asks a lot.
- **Control goes beyond text.** Reference images, camera motion, and timing need their own representations, often added during post-training.

## Detailed Notes

### What goes into a visual generator? · 00:15
- Dieleman has spent over a decade at Google DeepMind on the generative media team, working on models such as Veo and Nano Banana.
- The talk centers on diffusion because it is the prevailing paradigm for audiovisual generation; language modeling mostly relies on autoregression.
- Eight topics structure the tour: data curation, representation, modeling, architecture, training at scale, sampling, distillation, and control.
- Data curation is essential but under-incentivized in research, where reusing a common dataset keeps results comparable. He says improving data can matter more than tweaking the model or optimizer, and notes the details are sparsely published and part of the "secret sauce."

### Compress each example into a modelable latent · 03:59
- Images are pixel grids; video adds a time axis. A single 30-second, 1080p, 30 fps video is several gigabytes — far too much for one training example.
- Early diffusion models trained directly on pixels and worked surprisingly well, but the approach does not scale with resolution or duration.
- Existing codecs (JPEG, H.265) optimize for small files and obscure the structure a generator needs, so instead a **learned autoencoder** is used: an encoder squeezes input into a latent bottleneck and a decoder reconstructs it.
- The pipeline has three stages: train the autoencoder; train a generator (autoregression or diffusion) over the latents; sample a latent and decode it back to pixels.
- Dieleman says diffusion generally wins for audiovisual quality at a given parameter budget.

### Compress detail while preserving the grid · 06:53
- A 256×256 RGB image becomes a 32×32 latent grid — an eightfold spatial downsample, matching the autoencoder in the original **Stable Diffusion**.
- Extra latent channels carry information that plain resizing would discard, including some high-frequency detail. Compression is still lossy, but the tensor is far smaller.
- Video has extra temporal redundancy; learned compression can shrink video tensors by roughly two orders of magnitude — the difference between fitting in memory and not training at all.
- The latent deliberately keeps more structure than a codec like H.265; it still has grid topology at a coarser scale, which the network's inductive biases rely on.
- An **EQ-VAE** visualization maps latent principal components to RGB: animals remain recognizable, showing the compression mostly removes local texture and fine structure while keeping the scene's broad arrangement.

### Why one denoising prediction is blurry · 09:39
- Autoregression predicts a sequence element by element, which suits language but requires an arbitrary order for a 2D image. Diffusion instead defines a **corruption process** — add noise gradually — and learns a **denoiser** to reverse it.
- In the bunny example, a little noise blurs the whiskers while the silhouette survives; more noise erases the global structure too. Fine detail disappears before broad shape.
- Using a 2D diagram as a geometric aid: let `x₀` be a clean sample and `xₜ` its noisy version. Sampling starts from noise and reaches a state like `xₜ`, where the denoiser predicts a clean image.
- The prediction is ambiguous because corruption destroyed information; many originals could explain the same noisy observation, so the model predicts their average — which is blurry.
- **The prediction is a direction toward plausible images, not a finished image.** The blurry estimate represents a region of possible originals.

### Take a small step, then predict again · 13:49
- Jumping straight to the predicted clean image would keep the blur, so the sampler moves a small distance toward it and asks the denoiser again.
- This resembles neural-network optimization: the update direction is only locally valid, so an oversized step overshoots. The difference is the space — an optimizer moves parameters, a sampler moves pixels or latents.
- Many samplers add a little **fresh noise** afterward (less than they removed). It is new randomness, not the noise just removed, and it obscures the denoiser's inevitable mistakes, reducing error accumulation. Not all samplers do this.
- At the next state `xₜ₋₁` there is less noise, so the denoiser has more information, its prediction changes, and the region of plausible originals shrinks.
- Repeating this until the region collapses to a point yields a finished sample.

### Why diffusion generates from coarse to fine · 16:22
- A Fourier transform of an image splits into magnitude and phase; radially averaging the magnitude spectrum gives a one-dimensional curve.
- On log-log axes those curves look like straight lines — a power law. Natural images (and, he broadly extends, video) have stronger low frequencies and weaker high frequencies. Gaussian noise, by contrast, has a flat spectrum in expectation.
- Adding noise therefore drowns the weakest, highest frequencies first, then progressively lower ones, until no structure remains.
- Reversing this establishes low-frequency structure (broad semantics) before high-frequency detail. Dieleman calls the effect **spectral autoregression** — an approximate analogy, not a hard ordering.
- It also lets training emphasize the noise levels and scales that matter most perceptually.

### Choose an architecture for space and time · 19:59
- The denoiser's contract is simple: take a noisy image, predict the clean one.
- **U-Nets**, originally for segmentation, fit because they preserve spatial dimensions; early Stable Diffusion models used them. **Transformers** also work and allow fully bidirectional attention — no causal mask — making them slightly more expressive than a typical LLM transformer, while reusing LLM scaling knowledge.
- For video there are three options: **fully autoregressive** (flatten height × width × time and generate tokens), **joint diffusion** (noise and denoise the whole spatiotemporal volume — the common modern choice), and a **hybrid** (autoregression in time, diffusion within each frame), which suits real-time generation. **Genie** is an example of the hybrid.
- Training needs more than splitting a batch. **Data parallelism** spreads examples across chips; **model parallelism** spreads the model itself.
- These visual models still tend to be smaller than contemporary LLMs, but large enough that sharding and communication matter. He names JAX with `pjit` for expressing sharding and letting the compiler cut inter-chip communication (current JAX docs deprecate `pjit` in favor of `jax.jit`).

### Amplify the effect of the prompt · 23:29
- Sampling has its own choices: a **deterministic** sampler maps each noise sample to a fixed output (useful for distillation), while a **stochastic** one adds fresh randomness and can resist accumulated errors.
- **Guidance** trades diversity for quality. Raising the guidance scale makes samples more alike while improving each one, letting diffusion models perform well without LLM-sized parameter counts. It applies to autoregressive models too, but works especially well for diffusion.
- Mechanism: make two predictions for the same noisy input — unconditional (`u`) and conditional (`c`, with the text prompt). The prompt restricts the possible originals, so the conditional prediction is less blurry.
- Amplify the difference: `Δ = c − u`, then `x̂₀,guided = u + s·Δ`. A scale of 1 returns the conditional; larger values extrapolate. This needs two model evaluations per step.
- The guided-versus-unguided examples come from OpenAI's 2021 **GLIDE** paper. Guidance reduces diversity but improves prompt match, and Dieleman says modern image generation effectively always uses it — removing it would surprise users with the quality drop.

### Distill the trajectory into fewer steps · 28:01
- Here **distillation** means reducing the number of network passes, not shrinking the network.
- Sampling traces a nonlinear trajectory; at each point the denoiser gives a local direction. With a deterministic sampler, one noise sample follows a fixed path to one output.
- **Consistency models** exploit that by learning to predict where the path ends, potentially replacing many updates with a direct jump.
- A single jump asks the network to do what roughly fifty passes once did — a tall order that usually will not work well out of the box.
- A compromise predicts endpoints over shorter intervals, keeping several steps (three in his example). Those counts illustrate the quality-versus-compute trade-off, not a measured speedup.

### Give users controls beyond language · 30:02
- A generator is useful only if people can steer it. Text prompts are convenient but awkward for precise requests.
- Reference-based generation is more direct for some goals — to put yourself in a video, supply a photo or short clip rather than describing your appearance in detail.
- Video also needs explicit control over camera motion, event speed, and event timing; these signals need representations suited to what is being controlled.
- Most pre-training data may lack such annotations, so a common approach is to pre-train with text conditioning and add other controls, plus preference tuning (reinforcement learning or direct preference optimization), during post-training.
- The broad visual distribution is learned first; later training shapes how users steer it and which outputs the system favors.

### Q&A: style, guidance schedules, and frameworks · 32:15
- On why models like Nano Banana have a recognizable style: guidance plays a part, but Dieleman suspects post-training dominates — pre-training models a broad image distribution, and post-training picks an opinionated slice of it.
- Excessive saturation is a common sign the guidance scale is too high. Because the scale acts at every noise level, people often use a **guidance schedule** — he cites work suggesting little or no guidance at the very start and end, ramped up in the middle.
- For text diffusion, guidance applies the same way (the prompt is a conditioning signal), but how well is open. He says guidance works best when there is a **semantic gap** between the conditioning signal and the output — text carries high-level meaning while images carry detail — which is why a segmentation mask benefits less and guidance is less common in language generation.
- On JAX versus PyTorch for sharding, he caveats limited PyTorch experience after over a decade at Google. JAX was designed to make TPU use easy, especially across many chips with fast interconnects; PyTorch can also shard and parallelize. It is a matter of design priorities and experience, not a benchmark.
- Do more denoising steps help? Up to a point: finite updates approximate a nonlinear trajectory with a piecewise-linear path, and straying too far hurts quality. **Rectified flow / Reflow** trains straighter trajectories so fewer, coarser steps stay accurate.
- On camera control, he cannot disclose internal practices and is not on the capabilities team. The general problem is finding a semantically meaningful representation, then choosing how to insert it: as **additional tokens** the model attends to, or as **broadcast conditioning** available to every token — how the noise level is typically supplied. Global properties should inform processing throughout.
- Finally: without fresh noise, generation can be fully deterministic. Once you have a denoiser, you can build stochastic or deterministic generators; deterministic paths are useful for moving both ways between data and noise, and are required for a deterministic teacher in consistency distillation.

## Resources

- [Sander Dieleman's blog](https://sander.ai/) — diffusion intuition, latent-space guides, and "diffusion is spectral autoregression"
- [EQ-VAE](https://arxiv.org/abs/2502.09509) — equivariance regularization for latent representations
- [Stable Diffusion v1 repository](https://github.com/CompVis/stable-diffusion) — original latent diffusion implementation
- [GLIDE](https://arxiv.org/abs/2112.10741) — early large-scale text-guided diffusion and guidance comparison
- [Consistency Models](https://arxiv.org/abs/2303.01469) — one-step and multistep sampling via endpoint prediction
- [Flow Straight and Fast](https://arxiv.org/abs/2209.03003) — rectified flow and repeated rectification
- [JAX API changes](https://docs.jax.dev/en/latest/changelog.html) — documentation of the `pjit` → `jax.jit` transition
- Genie 2's autoregressive diffusion architecture — Google DeepMind's frame-by-frame latent diffusion write-up
