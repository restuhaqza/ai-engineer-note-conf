# Gemma, DeepMind's Family of Open Models

| Field | Details |
| --- | --- |
| **Speaker** | Omar Sanseviero |
| **Affiliation** | Google DeepMind |
| **Type** | Keynote |
| **Date** | 2026-04-20 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=_gVFUEdhCyI) |
| **Tags** | `reasoning-and-models` `architecture` `apis-mcp-and-protocols` `infrastructure-and-deployment` `vision-and-video` `coding-and-developer-tools` |

## Summary

Omar Sanseviero introduces **Gemma 4**, Google DeepMind's newest family of open models, a week after its release. "Open" here has a concrete meaning: you can download the weights, run them on your own hardware, and fine-tune them for your own task instead of relying only on a hosted API.

The lineup is organized by deployment role rather than one model scaled up and down. Small **E2B** and **E4B** models target phones and even Raspberry Pi devices; a **26B A4B** mixture-of-experts model trades capability for low latency; **31B** is the most capable option and still runs on a consumer GPU. Across three generations, the emphasis is better results without bigger models.

The technical core is **per-layer embeddings**, the "E" in E2B and E4B. These weights behave like lookup tables that can live on CPU or disk while the compute-heavy part stays on the GPU, which is what makes on-device deployment practical. The talk also covers multimodal and multilingual abilities, a switch to the Apache 2.0 license, ecosystem integrations, and community fine-tunes.

Sanseviero's closing ask is simple: spend an hour trying today's open models. He still recommends API models such as Gemini for maximum raw capability, but argues local models are already good enough for agentic, private, and offline work.

## Key Takeaways

- **Gemma 4 launched a week before the talk** and, in the speaker's words, is the most capable open-model family Google has released.
- **Four sizes for four jobs:** E2B and E4B for phones, 26B A4B for low-latency inference, and 31B for maximum capability — even 31B runs on a consumer GPU.
- **Effective parameters are not total parameters.** E2B holds more weights than its "2 billion" name suggests; the extra per-layer embeddings can sit outside GPU memory.
- **Per-layer embeddings act like lookup tables**, avoiding dense matrix math and letting `llama.cpp` offload them to CPU or disk with `--override-tensor`.
- **Everything can run offline.** Demos include an Android agent, live coding in airplane mode, and ten parallel Gemma instances on a laptop generating SVGs.
- **Gemma 4 is multimodal and heavily multilingual**, trained on 140+ languages with a Gemini-based tokenizer that helps low-resource language fine-tuning.
- **The license moved to Apache 2.0**, addressing a common complaint about earlier Gemma releases.
- **Specialization is a first-class use case:** ShieldGemma screens policy-violating content and MedGemma handles medical tasks, among many community variants.

## Detailed Notes

### What can you run on your own hardware? · 00:15
- Gemma is Google DeepMind's family of open models: downloadable, runnable on your own infrastructure, and fine-tunable.
- The previous generation, Gemma 3, spanned roughly 1B to 27B parameters and was described as the most capable open models that fit on a single consumer GPU at its release.
- A comparison slide pairs LM Arena scores with dots showing how many H100/A100 GPUs each model needed just to load — fitting the weights is a prerequisite, not a measure of speed.
- The talk does not specify the precision or memory assumptions behind those load comparisons.

### Choose for the device, latency, or capability · 01:20
- Gemma 4 ships as distinct roles rather than one model scaled uniformly.
- The smallest **E2B** and **E4B** models run on Android phones, iPhones, and even Raspberry Pi devices.
- **26B A4B** is a mixture-of-experts model tuned for speed and low latency.
- **31B** is the most capable dense model in the launch family and is still aimed at consumer GPUs; the sizes slide labels its memory column as **8-bit**.
- A later 12B variant now appears on the model card but was not part of this launch discussion.

### Agents and coding without API calls · 02:27
- On an Android phone, an agent picks from available skills — the example selects piano playing and performs it.
- Another phone example does live coding in airplane mode with no API calls, keeping both inference and app behavior on the device.
- On a laptop, ten Gemma instances run in parallel through `llama.cpp`, each producing a different SVG; the speaker reports about 100 tokens per second.
- He does not state whether that figure is aggregate or per instance, nor the hardware, model variant, precision, or context length.

### More capability without larger models · 03:39
- A chart plots parameter count (x-axis) against LM Arena score (y-axis); the desirable corner is upper-left: strong results from small models.
- The speaker treats Arena as an imperfect proxy for community preference in general chat, not a full measure of coding or agent reliability.
- Across Gemma 2, Gemma 3, and Gemma 4, the emphasis is improving capability without increasing size.
- Gemma fits desktops, laptops, and phones; a community experiment even tried it on a Nintendo Switch via `llama.cpp` — an illustration of experimentation, not a recommended deployment.
- Gemma 4 switches to an **Apache 2.0** license, giving builders a familiar, permissive framework.

### Why effective parameters differ from total parameters · 05:23
- The 26B model uses **mixture of experts (MoE)**. The "E" in E2B/E4B instead means **effective parameters**.
- These smaller models use **per-layer embeddings (PLE)**, introduced the previous summer; the speaker's spoken totals are approximate.
- Per the model card, E2B has about 2.3B effective parameters and 5.1B including embeddings. The effective count describes compute footprint, not the full weight inventory or a GPU-memory budget.
- PLE behaves like lookup tables instead of dense matrix multiplications, so those weights can live in CPU-accessible memory or on disk while the active part stays on the GPU.
- In `llama.cpp`, `--override-tensor` controls tensor placement; a pattern such as `PLE_TENSOR_REGEX=CPU` moves PLE tensors off the GPU.
- The pattern must match the model file's actual tensor names; disk-backed PLE is separately documented for LiteRT-LM.

### Combine sensory input with language adaptation · 06:55
- The smallest models understand images, video, and audio, including speech recognition and speech-to-translated-text (e.g., Spanish speech producing French text).
- Larger models add fine-grained video understanding and spatial tasks such as pointing to a llama in a picture or detecting multiple objects.
- Gemma 4 was trained on over 140 languages and uses a tokenizer based on Gemini's multilingual research.
- Training coverage does not imply equal proficiency everywhere, but tokenizer design matters independently — it shapes how a language is represented before fine-tuning.
- That helps adaptation for low-resource languages such as Quechua in Peru or India's official languages, where developers bring their own data.
- Multilingual and multimodal behavior can combine, e.g., reading Japanese text inside an image.

### Keep the tools developers already use · 08:43
- About a week after release, Gemma 4-based models had roughly 10 million downloads and 1,000+ derivatives (quantizations and fine-tunes); the whole Gemma family passed 500 million downloads.
- The strategy is to meet developers where they are: Unsloth, MLX, Ollama, Hugging Face, vLLM, and SGLang are named collaborators.
- Someone fine-tuning with Hugging Face Transformers should be able to stay there rather than switching toolkits.
- Android Studio's agent mode is the product example: an offline setup backed by Gemma served through `llama.cpp`, Ollama, or vLLM.
- Part of that Android-development strength comes from Android-related datasets and benchmarks included during training.

### Fine-tunes, regional languages, and staying local · 10:45
- Reasons to fine-tune go beyond improving a chatbot: stronger task ability, a different conversational style, or a non-conversational predictor.
- Official variants show two specializations: **ShieldGemma** (guardrail models that flag policy-violating text or images, including toxic content) and **MedGemma** (a multimodal, Gemma-3-based model for tasks such as radiology and chest X-ray reading, with open weights for narrower tuning).
- Community work includes AI Singapore's research on Southeast Asian languages and Sarvam in India, where government investment supports national models for official languages — multilingual and sovereign AI interests meeting.
- The speaker cites a December paper in which researchers used Gemma 3 to propose cancer-therapy pathways that were then tested and validated in a laboratory (lab validation, not clinical treatment outcomes).
- Local deployment serves different needs: finance and legal work that must keep data on private servers, offline subway and airplane use, a Chrome extension reading the screen, and on-device control.
- His ask: spend an hour in the next two weeks with current open models. For maximum raw intelligence he still points to Gemini or another API model; his 6–12 month outlook is more capable models on personal devices, customized with their owners' data.

## Resources

- [Gemma 4 model card](https://ai.google.dev/gemma/docs/core/model_card_4) — sizes, effective-parameter counts, and modalities
- [Building on-device agents with Gemma 4](https://developers.googleblog.com/bring-state-of-the-art-agentic-skills-to-the-edge-with-gemma-4/) — launch article on mobile agent skills
- [Understanding llama.cpp tensor placement](https://github.com/ggml-org/llama.cpp/discussions/13154) — CPU tensor-placement syntax
- Gemma-based research into cancer antigen presentation — C2S-Scale druggable-pathway study
