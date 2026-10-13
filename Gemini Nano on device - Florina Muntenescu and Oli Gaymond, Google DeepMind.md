# Gemini Nano on device

| Field | Details |
| --- | --- |
| **Speaker** | Florina Muntenescu, Oli Gaymond |
| **Affiliation** | Google DeepMind |
| **Type** | Talk |
| **Date** | 2026-05-22 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=owH1f0N-keY) |
| **Tags** | `architecture` `infrastructure-and-deployment` `rag-context-and-search` `apis-mcp-and-protocols` |

## Summary

How should an intelligent Android feature run, and how much infrastructure should its developer own? Florina Muntenescu and Oli Gaymond walk through the three deployment choices available: on-device, hybrid, and cloud. The choice is about where inference happens, not a different kind of user interface.

The talk centers on **Gemini Nano**, Google's on-device model, and **AICore**, an Android system service that shares one model across all apps. Developers reach Nano through ML Kit's GenAI APIs, including a general Prompt API. Where deliberate control over the model matters more, LiteRT-LM offers more customization at the cost of more testing work.

Most of the session is audience-driven. The pair take questions on memory and battery use, how the shared model is scheduled across many apps, whether the API grants file access, and when embeddings will arrive.

A recurring theme: a managed model service handles deployment and testing for you; broader custom-model reach means taking that work back.

## Key Takeaways

- **Three deployment options.** On-device, hybrid (local when available, else cloud), and cloud-only.
- **On-device wins on privacy and offline use.** Data stays local and there is no per-call server cost, though battery and storage costs remain.
- **AICore shares one model.** Apps don't each ship their own copy; requests still run in isolation.
- **Two paths.** ML Kit GenAI APIs for the managed Gemini Nano; LiteRT-LM for your own, more customized models.
- **Hybrid routing adds reach.** Firebase AI Logic uses Nano when available and falls back to the cloud otherwise.
- **Battery depends on the workload.** Occasional use (10–20 calls/day) is not a concern; continuous inference drains quickly.
- **The API is not file access.** Apps pass content they already have permission to read; the model can't browse the filesystem.
- **More device reach means you own testing.** AICore manages supported device/model combos; LiteRT-LM makes you test them yourself.

## Detailed Notes

### Three deployment options · 00:16
- Muntenescu works in developer relations across intelligent experiences and developer productivity; Gaymond is a PM for Android AI.
- Android AI work spans OS features (screen brightness, system memory), developer tools, and hardware acceleration with silicon partners.
- The first decision is where to process a request: on-device, hybrid, or cloud.
- **On-device:** the prompt runs on the phone with nothing sent to a server. Good for sensitive data like banking details and for offline use.
- Local inference also avoids extra server inference charges, but the phone still pays in battery and storage.
- Fit for local: personalization and shorter-context tasks such as translation.
- Android offers two routes — ML Kit's GenAI APIs (managed Gemini Nano) or LiteRT-LM (more customization).

### AICore and Gemini Nano · 03:11
- **Gemini Nano** is described as the most efficient model for on-device tasks, built on the same architecture as Gemma 4.
- That Gemma 4 connection applies most directly to the next generation, **Gemini Nano 4**, per Google's preview announcement.
- Nano reaches devices through **AICore**, a system service, so one model serves all apps instead of each shipping its own.
- AICore optimizes for the hardware present at runtime, aiming for lower latency and faster execution.
- Gaymond compares AICore to a managed cloud service: you bring the feature and prompt; the platform handles getting and running the model.
- **Sharing the model does not mean sharing context.** Requests run in isolation, and AICore does not retain inputs or outputs after processing.
- That non-retention guarantee covers AICore's processing; it does not stop the calling app from storing data itself.

### ML Kit GenAI APIs and hybrid routing · 04:49
- ML Kit GenAI APIs sit inside the broader ML Kit, which also covers vision and natural language.
- Specialized GenAI operations include summarization, proofreading, and rewriting.
- The most general is the **Prompt API**: text and image input, text output. It supports image understanding, content assistance, content analysis, and entity extraction.
- Device availability is the constraint. Nano targets Pixel 9- and Pixel 10-generation hardware plus devices from other manufacturers.
- **Firebase AI Logic** hybrid inference (launched a couple of weeks before the talk) routes to Nano locally and to the cloud otherwise.
- The pair expect a similar experience between local Nano 4 and cloud Gemini Flash; they do not present a quality comparison.
- Firebase AI Logic also offers cloud-only access to Gemini Pro and Gemini Flash-Lite, via the Gemini Developer API and Vertex AI.
- The platform goal is API consistency so developers can blend local, cloud, and hybrid paths.

### Battery and memory questions · 08:16
- First audience question: RAM and battery impact of Nano and LiteRT models.
- Gaymond: the models have been shrunk as far as possible while keeping capability, but they still need flagship hardware.
- Continuous inference drains the battery quickly; AICore exists partly to optimize execution.
- Intermittent use — a user asking questions or manipulating data 10–20 times a day — is not concerning for battery life.
- That is a workload observation, with no specified device, workload size, or measured energy number.
- For latency-insensitive batches, he describes processing in the background while the phone charges overnight.
- Choosing a custom model shifts optimization work back to the developer, who must profile resource impact with available tools.
- No quantified runtime-RAM answer was given.

### Many apps, one model · 10:13
- An attendee asks whether a phone with 100 apps using Nano creates a bottleneck.
- Gaymond separates the cost of delivering the model from the cost of executing requests.
- He estimates the smallest useful models at roughly 1 GB, and the shipped ones at roughly 3–4 GB total (model-size estimates, not runtime RAM).
- Shipping that much data per app is a heavy burden; a system copy lets many apps share the distribution and storage cost.
- Execution still consumes resources, but central management lets AICore queue work and attribute battery use to the apps causing it.
- His comparison is GPS or Wi-Fi: users accept the battery cost when a feature is valuable and reject it when it is not.
- Scheduling detail: background requests queue, while the actively used foreground app gets top priority.
- Important boundary: ML Kit documentation checked in August 2026 allows GenAI inference only from the top foreground app — background calls, including from foreground services, get `BACKGROUND_USE_BLOCKED`. So the overnight-processing idea is not an implementation recipe for that API.

### The API vs the assistant, files, and embeddings · 12:43
- An attendee asks whether "Hey Google" answers on their phone run locally or remotely. Muntenescu cannot establish which app is involved and expects remote.
- A familiar voice interface does not by itself reveal where inference happens.
- She separates two journeys: using the Gemini app versus building an app that calls a model. These APIs serve the second.
- **Skills live one layer higher.** A skill is just instructions composed into a prompt next to the user's query; the app owns that.
- Gaymond offers the Kotlin string-composition example: instructions and query joined into a prompt, independent of the inference client.
- Pocket Claw or OpenClaw are mentioned as hypothetical software that could assemble skills and call the API, not demonstrated integrations.
- On files: Nano is contained within AICore, accessed through APIs. The Prompt API supports a RAG-style workflow and can take text and image input.
- The app must supply the content it already has permission to read; the inference API does not grant filesystem access.
- On embeddings: there is no embedding API through this interface yet. Gaymond says it is coming, using a Gemma embedding model.

### Device reach and who owns testing · 17:51
- Final question: across diverse devices and models, what is capable where, and do you need flagship hardware?
- Gaymond separates classical ML Kit models (text, OCR, vision) from the larger generative models.
- Classical models reach a billion-plus devices; the GenAI APIs currently require fairly flagship devices from the last couple of years.
- The device-count claim is a broad reach estimate, not a compatibility guarantee for every API.
- The remaining choice is who owns validation across that hardware range.
- **AICore:** the platform manages supported model/device combinations, so availability signals it will run well.
- **LiteRT-LM:** if the supported range is too narrow, developers can reach a wider range but must test it themselves.
- Gaymond hands that topic to Cormac's following session. Managed service reduces deployment and testing work; broader custom reach takes that work back.

## Resources

- [ML Kit GenAI APIs](https://developers.google.com/ml-kit/genai) — on-device APIs, supported devices, model sharing, and application integration.
- [Gemma 4 in the AICore Developer Preview](https://android-developers.googleblog.com/2026/04/AI-Core-Developer-Preview.html) — April 2026 announcement connecting Gemma 4 to next-generation Nano.
- [AICore Developer Preview setup](https://developers.google.com/ml-kit/genai/aicore-dev-preview) — enrolling a supported device and testing preview capabilities.
- [Firebase AI Logic hybrid inference on Android](https://firebase.google.com/docs/ai-logic/hybrid/android/get-started) — setup and capability limits for hybrid inference.
- [LiteRT-LM](https://github.com/google-ai-edge/LiteRT-LM) — Google's open-source framework for deploying language models on edge devices.
- [Google AI Edge Gallery](https://github.com/google-ai-edge/gallery) — an app and repo for exploring locally running generative models.
- EmbeddingGemma architecture and training (related) — how EmbeddingGemma produces text representations for retrieval.
