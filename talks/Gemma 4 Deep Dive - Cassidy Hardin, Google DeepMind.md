# Gemma 4 Deep Dive

| Field | Details |
| --- | --- |
| **Speaker** | Cassidy Hardin |
| **Affiliation** | Google DeepMind |
| **Type** | Talk |
| **Date** | 2026-04-27 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=_A367W_qvc8) |
| **Tags** | `infrastructure-and-deployment` `apis-mcp-and-protocols` `speech-and-audio` `architecture` `rag-context-and-search` `vision-and-video` |

## Summary

Cassidy Hardin, a researcher at Google DeepMind, walks through the architecture of
**Gemma 4**, the latest family of open-source models released the previous week.
Rather than one architecture scaled up and down, Gemma 4 is four models: two small
**effective** models for on-device use, a mixture-of-experts model, and a larger
dense model. The release also moves Gemma to the **Apache 2.0** license, so the
same family can be used from early experiments through deployment.

The small models — **E2B** and **E4B** — target phones, iPads, and laptops and
accept text, vision, and audio but return text only. The **26B** model brings
mixture-of-experts to Gemma, using only a few billion active parameters per pass.
The **31B** dense model aims at advanced multimodal reasoning with a long context
window. Hardin reports both larger models ranked in the LM Arena's top six
open-source models at launch.

Most of the talk explains *how* these gains were achieved. Gemma 4 interleaves
local attention (which looks at a sliding window of recent tokens) with global
attention (which can see the whole preceding context), and shares key/value heads
to cut the memory cost. A new embedding scheme, **per-layer embeddings (PLE)**,
uses narrow per-layer tables stored in flash memory instead of scarce VRAM (graphics
memory), which is what makes on-device inference practical for small models.

The multimodal section covers image handling with variable resolution and aspect
ratios, pooling groups of image patches into single soft tokens to shorten the
sequence, and an audio path built on a Conformer encoder. She finishes with
practical guidance: self-host the smaller models via Hugging Face, Kaggle, or
Ollama, or use the hosted larger models through AI Studio and Vertex.

## Key Takeaways

- **Gemma 4 is a family, not one model.** Two on-device effective models, a mixture-of-experts model, and a dense reasoning model.
- **Apache 2.0 licensing** makes the family usable across the whole development cycle without changing license terms.
- **Local + global attention is interleaved.** Mostly local layers with periodic global layers; the final layer is always global.
- **Grouped-query attention saves memory.** Multiple query heads share key/value heads, with wider global heads to limit the quality loss.
- **MoE separates total from active parameters.** The 26B model routes to eight of 128 experts per pass, plus one shared expert.
- **PLE is the on-device trick.** Per-layer embedding tables stored in flash memory reduce the demand on VRAM.
- **Vision supports variable resolution.** Five settings let developers trade image detail against soft-token budget.
- **Nine image patches become one soft token.** Pooling shrinks the sequence the language model must consume.

## Detailed Notes

### Four models, from phones to reasoning systems · 00:15
- Gemma 4 (released the week before the talk) comes in **four sizes**.
- **E2B** and **E4B** are "effective," smaller models optimized for on-device use on phones, iPads, and laptops.
- **26B A4** is Gemma's first mixture-of-experts model; **31B** is a dense model.
- Hardin reports both larger models ranked in the **top six open-source models on LM Arena** (a public model leaderboard) at launch.
- The family moved to the **Apache 2.0** license to make it accessible across testing, deployment, and building.
- The larger models target complex reasoning, agentic workflows, and coding; the effective models prioritize local multimodal use.

### Reasoning capacity and active computation · 02:18
- The **31B dense** model targets advanced multimodal reasoning; Hardin reports a number-three global Arena ranking, calling it a launch-era comparison.
- Its practical features: a **256K context window** (how much text it can consider at once), native thinking, function calling, and structured **JSON** output.
- **26B** separates total capacity from the parameters used in one forward pass: **3.8 billion active parameters**.
- Its **router** selects **eight of 128 experts** per pass — sparse computation inside a larger parameter set.
- The effective **E2B** and **E4B** models accept **text, vision, and audio** input but produce **text only**.
- Hardin describes improvement over **Gemma 3** across agentic capabilities, coding, multimodal tasks, and multilingual tasks, without giving per-benchmark numbers here.

### Local and global attention with shared key/value heads · 04:13
- The dense decoder used by 31B and the effective models **interleaves local and global attention layers**, generally at a **5:1** local-to-global ratio (E2B uses **4:1**).
- **Local attention** restricts each token to a sliding window of preceding tokens; **global attention** can reach all preceding tokens.
- The last layer is always global. Sliding windows reduce the work of local layers while their representations still pass forward.
- Windows are **512 tokens** in the smaller models and **1,024 tokens** in the larger models — this is a per-layer window, not the model's total context limit.
- Global layers still attend over the full context, making their keys and values expensive to keep.
- **Grouped-query attention** reduces that cost by letting several query heads share key/value heads.
- In the illustration, local layers group 2 queries per shared head (dimension 256) and global layers group 8 (dimension 512); the wider global heads partly compensate for fewer independent heads.
- Hardin warns these ratios aren't universal — the published E2B config specifies eight query heads and one key/value head.

### Mixture of experts: one shared expert plus routed experts · 06:48
- Mixture of experts (**MoE**) changes the decoder's feed-forward part. Instead of one dense network, it uses many smaller networks called **experts**.
- A **shared expert** runs on every pass; a router also selects **eight experts from 128** for that pass.
- The shared expert is **three times the size** of a regular routed expert in Hardin's description.
- The structure is **128 routed experts plus one shared expert** — not 128 including the shared expert.
- Only the selected experts (and the shared expert) run, which is what makes "active parameters" smaller than total parameters.
- This is why the 26B model can be efficient while still carrying the capacity of a larger model.

### Effective size and flash-backed per-layer embeddings · 07:42
- The effective models distinguish parameters **needed to operate** the model from total representational parameters: E2B has **2.3 billion effective** and **5.1 billion including embeddings**.
- The goal is useful local inference without calling a remotely served model.
- **Per-layer embeddings (PLE)** add a dedicated embedding table at each layer; they do **not** replace the standard token embedding table.
- A token ID still maps to an ordinary model-width vector — **1,536 dimensions** in E2B, **2,560** in E4B.
- Each PLE table also covers the vocabulary but holds a much smaller representation, only **256 dimensions**.
- The key move: these PLE tables are stored in **flash memory rather than VRAM**, easing the main constraint on local devices.
- Following the token "hi": its identity stays the same, but each layer supplies a different learned PLE vector (35 layers for E2B, 42 for E4B).
- At the end of a decoder block the model looks up the token's 256-dimensional per-layer vector and projects it up to the full embedding width.
- Hardin attributes the stronger performance of the small models versus prior Gemma generations to this PLE design.

### Variable-resolution vision and spatial positions · 11:02
- Gemma 4 is natively multimodal; Gemma 3 introduced vision, and Gemma 3n added audio, vision, and text, paving the way.
- The larger 26B/31B models use a **550-million-parameter vision encoder**; E2B and E4B use a smaller **150-million-parameter** encoder.
- New in Gemma 4: **variable aspect ratios and resolutions**, giving five supported settings for image resolution and soft-token budget.
- The vision encoder splits an image into **16×16-pixel patches**, flattens them, and linearly projects them into patch embeddings.
- **Positional information** is added so the model can tell where each patch sits. A 4×2 grid and a 3×3 grid place "patch four" in different positions, so a plain sequence index isn't enough.
- Variable resolution matters separately: two images can share a 3:2 ratio but carry very different detail; a bigger image budget preserves more detail at the cost of tokens.
- Budget is a workload decision: use a larger budget for OCR and spatial object recognition, and a smaller one for text-focused tasks.
- This replaces Gemma 3's **pan-and-scan** approach, which split an image into square regions and processed it as several sequential images.

### Nine patches become one soft token · 15:00
- The language model doesn't receive one embedding per original patch. Gemma 4 **pools each 3×3 group of patches into one embedding**.
- So a budget of **280 image soft tokens** corresponds to a **2,520-patch budget** (280 × 9).
- Padding handles image dimensions that don't divide cleanly into 16×16 patches.
- Of the five settings, Hardin highlights budgets of **560 and 1,120 soft tokens** for object detection and OCR.
- The full path: **N image patches → N patch embeddings → N/9 pooled soft tokens → projection to language-model inputs**.
- In the final diagram, patches pass through a vision transformer (`GemmaVis ViT`), then pooled tokens go through a linear projection and RMSNorm before entering Gemma 4.
- The point of pooling: patch encoding preserves visual detail at the vision stage, while pooling shortens the sequence the language model must process.

### Audio encoding and deployment paths · 16:31
- Audio support in E2B and E4B targets translation and speech recognition.
- An audio tokenizer feeds a **305-million-parameter Conformer encoder** (a neural network with an added convolutional layer).
- Despite the "tokenizer" name, this path supplies **continuous audio embeddings**, not discrete token IDs.
- The pipeline: convert raw audio into a **Mel-spectrogram**, split it into N chunks, downsample through two convolutional layers, then feed N/4 soft-token embeddings into the Conformer.
- The Conformer follows the decoder architecture seen earlier, with an added convolutional layer.
- **Deployment options:** self-host all models via Hugging Face, Kaggle, or Ollama.
- **Hosted options:** access the 31B and 26B models through Google **AI Studio** and **Vertex**, useful for prototyping, agentic workflows, and testing function calling.
- Hardin's closing guidance: start from the workload you need to test, then choose whether to operate the model yourself or use the hosted route.

## Resources

- [Gemma 4 model card](https://ai.google.dev/gemma/docs/core/model_card_4) — specifications, modalities, evaluation results, and image budget details.
- Gemma 4 Technical Report — architecture, multimodal encoders, and training details.
- [Gemma 4 E2B model configuration](https://huggingface.co/google/gemma-4-E2B-it/blob/main/config.json) — inspect the attention schedule and embedding dimensions.
- [Run Gemma 4 with Ollama](https://ollama.com/library/gemma4) — local model access.
