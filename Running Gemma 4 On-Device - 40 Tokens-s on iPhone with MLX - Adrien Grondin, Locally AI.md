# Running Gemma 4 On-Device: 40 Tokens/s on iPhone with MLX

| Field | Details |
| --- | --- |
| **Speaker** | Adrien Grondin |
| **Affiliation** | Locally AI |
| **Type** | Lightning talk |
| **Date** | 2026-04-20 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=a2muGkT4WD4) |
| **Tags** | `infrastructure-and-deployment` `agent-engineering` `speech-and-audio` `reasoning-and-models` |

## Summary

Adrien Grondin, developer of the on-device chatbot **Locally AI**, shows how to run Google DeepMind's **Gemma 4** on an iPhone using **MLX**, Apple's machine-learning framework built for Apple silicon. His demo claims roughly 40 tokens per second for a quantized Gemma 4 E2B model on the latest iPhone.

The talk is a practical recipe in three steps: add the **MLX Swift LM** package to an app, choose a compatible model repository from the MLX Community on Hugging Face, and pass its model ID to the framework, which handles the download. Quantization does the rest, shrinking weights until they fit on a phone.

Grondin also covers the wider MLX ecosystem for vision, audio, and video, the trade-offs of quantization, and the fact that the model download — often 1–3 GB — remains the biggest barrier for users. In the Q&A he confirms tool calling works but structured generation is not yet directly supported.

The session ends with news: Locally AI has been acquired by **LM Studio**, connecting the phone experience to a desktop app that can also expose a local model server.

## Key Takeaways

- **MLX** is Apple's framework tuned for Apple silicon; **MLX Swift LM** (`mlx-swift-lm`) runs compatible models inside native iOS, iPadOS, and macOS apps.
- **Three steps to ship:** add the package, choose a model repo from MLX Community on Hugging Face, and pass the repo ID to the framework.
- **Quantize to fit the phone.** Grondin recommends 4–8 bits; below 4-bit quality tends to drop too much.
- **He reports ~40 tokens/s** for 4-bit Gemma 4 E2B on the latest iPhone, though he names no specific device or benchmark setup.
- **Older iPhones still work**, just slower — he offers 20 tokens/s as an illustrative rate that remains useful.
- **Downloads are the real friction:** models are typically 1–3 GB and must be fetched before the app is useful.
- **Tool calling is supported** by `mlx-swift-lm`; structured generation is not yet built in, only attempted by add-on packages.
- **Curated beats complete:** Locally AI limits its catalog to models verified to run well on iPhone.

## Detailed Notes

### A chatbot that runs on the phone · 00:15
- Locally AI is a fully native chatbot that runs models on-device via MLX; it also runs on iPad and macOS.
- Users can chat with Apple Foundation as well as compatible downloadable models, and the app manages model selection.
- Smaller variants of Gemma 4 are what make the phone experience feasible; Grondin has shown them running in social posts.
- MLX is Apple's framework optimized for Apple silicon — the iPhone chip and Mac chips alike.

### Start with the integration package · 01:49
- Gemma is one option; Grondin also points to **Qwen** and Hugging Face's **SmolLM**.
- For a native iOS, iPadOS, or macOS app, the entry point is **MLX Swift LM** (`mlx-swift-lm`), which downloads and runs compatible models.
- He calls the API straightforward and, since the audience has coding agents, suggests letting the agent handle implementation details.
- His estimate: an iOS app running a model can be assembled in under ten minutes.
- The surrounding MLX ecosystem spans more than text: **MLX-VLM** (vision/omni, Python and Mac), **MLX-Audio** (text-to-speech and speech-to-speech), and **MLX-Video** (image/video generation).

### Choose compatible weights, then pass their ID · 03:12
- Installing the framework is not enough — you also need compatible weights.
- MLX Community on Hugging Face publishes full-size and quantized versions; Grondin estimates roughly 4,000–5,000 models and says quantizations can appear around 30 minutes after a lab release (his observations, not guarantees).
- His iPhone example is **Gemma 4 E2B**, available as BF16, MXFP4, and 5-bit/6-bit variants, among others.
- The sequence: add `mlx-swift-lm`, pick a repository, then pass its ID to the framework, which downloads the model through its Hugging Face integration.
- Example configuration: `ModelConfiguration(id: "mlx-community/gemma-4-e2b-it-4bit")`.

### Fit the model to the workload · 04:41
- Full-size weights are generally too large for an iPhone, so **quantization** reduces the precision used to store weights at some cost to quality.
- Grondin's practical range is **4–8 bits**: below 4-bit often hurts output too much, 4-bit is his lower bound, and 8-bit is plausible for very small models.
- The right choice depends on model size as well as precision.
- A chatbot is not the only use: he describes a **Liquid** model with 350M parameters running through Shortcuts for fast text processing and automation (he does not name the exact checkpoint).
- For the chat case he reports about **40 tokens/s** for Gemma 4 E2B quantized to 4-bit on the latest iPhone — his reported setup, not a hardware-wide benchmark.

### What offline generation looks like · 06:01
- The demo runs live and offline, streaming the answer as it generates; the shown response is a long multi-day Paris itinerary.
- Streaming lets the user start reading immediately; a non-streaming interface could instead wait about four seconds — an illustrative design choice, not a measured time for every response.
- Older iPhones also run these models, though they should not be expected to match the latest-phone rate; he offers 20 tokens/s as an example that is still useful.
- The real question is whether generation is fast enough for the interaction you are building.

### Downloads, LM Studio, and a local server · 07:11
- The app is free on the App Store, but users must still download the model weights, typically around 1–3 GB depending on the model.
- Grondin calls model size the biggest remaining barrier; smaller, smarter models and improving iPhone hardware make the trade-off steadily easier.
- He announces that **LM Studio has acquired Locally AI**.
- LM Studio is a desktop tool for downloading Hugging Face models and running them locally; it supports both `llama.cpp` and MLX so users can compare engines.
- It can expose a running model through a local server using OpenAI-style and Anthropic-style interfaces, including streaming, so another app can send requests to a separate local process instead of embedding the runtime.

### Tool calling, structured generation, and curated models · 09:02
- An audience question about tool calling: yes, `mlx-swift-lm` supports it, and models have become much better at choosing and using tools than a year earlier.
- **Structured generation is a different capability and is not yet directly supported**; add-on packages (including work at Hugging Face) are trying to add it.
- Tool calling lets a model request an operation in another system; it does not by itself constrain output to a required structure.
- Two distinct entry points close the talk: developers install the package and fetch weights; regular users just download Locally AI and try it.
- The ready-made app still needs a model download, but removes the integration work. Its catalog is curated — Grondin checks models actually run well on iPhone, since availability does not equal suitability.

## Resources

- [MLX Swift LM](https://github.com/ml-explore/mlx-swift-lm) — Swift package for running models in Apple-platform apps
- [MLX](https://github.com/ml-explore/mlx) — Apple's framework for Apple silicon
- [Gemma 4 E2B model card](https://huggingface.co/google/gemma-4-E2B-it) — architecture, modalities, and effective parameters
- [Gemma 4 E2B 4-bit MLX weights](https://huggingface.co/mlx-community/gemma-4-e2b-it-4bit) — example model-ID loading
- [MLX-VLM](https://github.com/Blaizzy/mlx-vlm) — vision-language and omni models on Mac
- [MLX-Audio](https://github.com/Blaizzy/mlx-audio) — speech synthesis, recognition, and speech-to-speech
- [MLX-Video](https://github.com/Blaizzy/mlx-video) — image, video, and audio generation tooling
- [LM Studio's Anthropic-compatible local server](https://lmstudio.ai/blog/claudecode) — connecting apps to a local model server
- Liquid AI LFM2-350M — compact 350M-parameter edge model used for automation
