# TLMs: Tiny LLMs and Agents on Edge Devices with LiteRT-LM

| Field | Details |
| --- | --- |
| **Speaker** | Cormac Brick |
| **Affiliation** | Google |
| **Type** | Workshop |
| **Date** | 2026-05-03 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=BKWpYIWvAo4) |
| **Tags** | `data-and-model-adaptation` `infrastructure-and-deployment` `coding-and-developer-tools` `safety-and-governance` |

## Summary

Cormac Brick, a tech lead on Google AI Edge, walks through how to put language models on phones and other constrained devices using LiteRT-LM and Gemma models. He frames the core question as a deployment choice: how much intelligence can an app put on the device, and what should that intelligence do? He presents two patterns — a shared **system model** built into the operating system and customized with prompts or skills, versus an **in-app tiny model** loaded with the application and usually fine-tuned for one task.

The first half focuses on **agent skills**. With Gemma 4, thinking plus function calling lets a local model pick up a described skill and use it, so a single app can add capabilities without a new fine-tune. The key engineering trick is **progressive disclosure**: the model sees only one-line skill descriptions and loads full instructions on demand, which keeps the context small and the model reliable on device. A second trick, constrained decoding, restricts tool calls to the allowed set.

The second half turns to **tiny models** (under one billion parameters), which Brick says usually need task-specific fine-tuning to reach production reliability. The recommended recipe is synthetic data from a stronger cloud model, a small base model, fine-tuning, and quantization. He closes with a production example — an offline dictation app built from a modular pipeline of small specialized models — and a discussion of memory, LoRA adapters, and safety for edge deployments.

## Key Takeaways

- **Two deployment patterns.** Shared system models (roughly 2–5B parameters, in the OS, customized via prompts/skills) versus in-app tiny models (loaded with the app, usually fine-tuned).
- **"Effective size" is about RAM.** In names like E2B and E4B, the number is how many billion parameters must stay resident in memory, not the total.
- **Skills extend a local model.** A described skill plus a little JavaScript lets the model add new abilities; no custom fine-tune needed for the demos shown.
- **Load instructions on demand.** Progressive disclosure shows the model one-line descriptions first, saving tokens and helping lightweight models stay reliable.
- **Constrained decoding tightens tool calls.** Restricting generation to the allowed set of tools helps small models, and matters less as models grow.
- **Tiny models need fine-tuning.** Below roughly 500M parameters, task-specific training is often required for production reliability.
- **The tiny-model recipe:** generate synthetic data with a stronger cloud model, fine-tune a small base (such as Gemma 3 270M), and quantize.
- **Fine-tuning adds roughly 20–40 eval points.** On ten mobile actions, FunctionGemma went from about the low 40s percent to around 86% (his recollection).
- **Keep it modular.** Separate recognition and polishing models, and use LoRA adapters so one shared base can serve many tasks.

## Detailed Notes

### What should run on the device? · 00:16
- Brick works on Google AI Edge, which builds infrastructure used both internally for Google products and released as open source, in close collaboration with the Gemma team.
- His background: running GoogLeNet on a USB accelerator plugged into a Raspberry Pi at NeurIPS 2016, then leading architecture for the NPU in Intel laptops, before moving to Google three years ago.
- The two halves of the workshop: general-purpose local models extended with skills, and tiny models specialized and shipped inside apps — both only recently made practical.
- Reasons to run on-device: **latency/UX** (a Pixel live voice translation feature that needs a fast local loop), **privacy** (messaging stays encrypted on the phone), **offline** use, and **cost savings** (laptop users experimenting with small models for parts of desktop agent workflows).
- The stack: MediaPipe for app-level features (Photos Unblur, YouTube Shorts effects), LiteRT (formerly TensorFlow Lite) for general inference, and LiteRT-LM for language models.
- It runs on Android system services and third-party apps, and also on iOS, macOS, Linux, Windows, web, and embedded devices. One artifact serves CPU and GPU; an **NPU** (a dedicated neural accelerator) needs a specially compiled artifact.

### Two patterns: system models vs in-app tiny models · 07:05
- A restaurant app should not download its own multi-billion-parameter model just to help someone pick dinner. Instead, large models arrive as **system GenAI**: a shared foundation model built into the OS.
- System models are typically 2–5B parameters. Android's AICore exposes capabilities such as summarization and a prompt API; Apple Intelligence follows a parallel approach. This route mainly serves premium devices, and customization is via prompts or skills.
- **In-app GenAI** is task-specific, loaded with an app or webpage, targeted at wider reach (not just premium devices), and usually customized through fine-tuning.
- Brick reports strong results from models in the 100–500M parameter range on bounded tasks such as summarization, transcription, and voice-to-action. A small enough task can make a tiny model useful to many more users.
- **FunctionGemma**, a 270M function-calling model released in December, was adapted to ten Android-relevant voice-to-function actions and reached roughly 85–90% reliability on an internal eval.
- The key engineering requirement is adaptation: below about 500M parameters, models generally need task-specific fine-tuning for production reliability; some models around that size and above can handle broader tasks.

### Gemma 4 and "effective size" · 11:29
- Gemma 4 launched the week before the workshop and supplies the larger models used for the skills demos.
- **E2B** and **E4B** describe effective parameter sizes — roughly two or four billion parameters that must stay resident in RAM to run the model, not the total parameter count.
- The runtime memory-maps the embedding and per-layer embedding tables. During the autoregressive loop it reads only the rows needed for the current token — hundreds or thousands of bytes for the next-token step — and the OS can evict pages from older accesses. This separates total stored weights from the working set that must stay resident.
- E2B and E4B are available now for experimentation and are on the AICore roadmap, not yet integrated into AICore. Larger sibling models target consumer laptops with more memory (he gives roughly 32 GB as an example).
- The capability that unlocks skills is the combination of **thinking and function calling**. E2B and E4B accept audio, images, and text; the larger launch models accept images and text. This generation also moves to an Apache 2.0 license.

### Agent skills: a journal that acts, retrieves, and renders · 18:32
- **Google AI Edge Gallery** is the mobile test environment, available on Android and iOS, offering chat, image questions, audio transcription and translation, audio-to-function calling, and agent skills.
- A mood journal demo logs an entry (eight hours of sleep, plans to spend time with Amy), then asks for a seven-day trend; the model records observations and later summarizes the diary. A later request asks for the day's calendar as a bullet list. This uses skill instructions and callable JavaScript — not a custom fine-tune.
- The conversation extends to map lookup and Wikipedia queries for current Oscars information, giving the model knowledge newer than its training data. The journal skill can add, remove, and query entries.
- A breakfast image feeds a mood-music skill that calls a web service to compose and play lo-fi music — a reminder that local inference does not make every skill offline; the music service is an explicit network dependency.
- Skills can also change the output format: instead of three summary bullets, JavaScript can render them as cards or flashcards when the model judges that helpful.
- Three jobs for skills: retrieving information, performing actions, and presenting results in something richer than a chat message.

### Progressive disclosure and constrained decoding · 23:42
- Loading every tool's full documentation creates a large context before useful work begins. Gallery instead uses **progressive disclosure**: the model initially sees one-line skill descriptions; if one matches the request, it calls a tool to load that skill's detailed instructions and available functions.
- The same structure supports several kinds of extension: **input tools** (Wikipedia, a weather service), **output tools** (maps, cards), and **domain knowledge** (a CRM or a local retrieval system).
- A skill contains `SKILL.md` plus optional scripts or assets. Its metadata supports discovery, and full instructions arrive only when needed. Custom JavaScript then extends both Android and iOS apps.
- The loop is small enough to follow directly:
  1. Add enabled skill descriptions to the app's system prompt.
  2. Let the model select a relevant skill from its metadata.
  3. Call the load-skill tool, returning the contents of `SKILL.md` into context.
  4. Let the model call the run-JavaScript tool using the newly loaded instructions.
  5. Execute the skill's JavaScript and return its result to the conversation.
- **Constrained decoding** strengthens the tool-call boundary. The runtime applies constraints only while generating a tool call (not all natural-language output) and, because it knows the finite set of permitted tools, can require the call to match an allowed tool rather than merely be valid JSON. Brick says this work began while porting FunctionGemma and is especially helpful for roughly two-billion-parameter models; the benefit shrinks with much larger models.
- Gallery also lets users toggle skills (even controlling which descriptions are in context), load a virtual piano whose keys play sounds, import custom skills from a URL, and use skills that ask the user for API secrets. The team produced about 80 skills using tools including Antigravity and Gemini CLI.
- Underneath, an orchestrator keeps a skill registry and exposes three tools: load a skill, run JavaScript, and run an intent. Skills can include persona/scenario instructions, local JavaScript for offline use, or a web API call. Android native intents (such as Wi-Fi control) are another route, but depend on what the host app implements.
- A restaurant-roulette example combines language understanding, retrieval, and an interactive view: a request spoken in French (find a French restaurant in San Francisco, respond in English) requests an API key, searches for ten restaurants, and renders a roulette wheel. The `requireSecret` flag and `index.js` are the contract (later documented as `metadata.require-secret` and `scripts/index.html`), with secrets passed to scripts rather than through the model context.

### Writing and testing skills with coding agents · 35:03
- The team's favorite authoring pattern is "skills to write skills": give Gemini CLI, Claude Code, or Antigravity the Gallery documentation, several examples, and a description of the desired feature.
- The workshop example is an offline archiver: photograph a London landmark such as a Churchill statue, retrieve relevant Wikipedia content, store and index it locally, and read it on the flight home. Retrieval needs connectivity, but the saved archive works later offline.
- Testing can join the same loop. With a phone connected over ADB (Android Debug Bridge, a tool for talking to devices), Gemini CLI's Android ADB skill can interact with the app, check that the skill does what it claims, and iterate. Brick estimates more than half of the team's roughly 80 internal skills were initially created this way.
- An audience question clarifies model size: the demos default to E4B. E2B can run skills too, but may need simpler skills or fewer enabled at once. The restaurant clip itself announces effective 2B, so it is not a controlled comparison.
- Asked about a local-model browser integration, Brick says the team had not tried it.
- Gallery is open source and builds on LiteRT and LiteRT-LM, so developers can reuse the underlying APIs and model artifacts. On mobile, Brick expects larger models to reach production mainly through shared system GenAI, while desktop and embedded apps can load them directly.

### From Transformers to a shippable edge artifact · 41:57
- LiteRT-LM supplies the full autoregressive execution loop behind Gallery. At the workshop it had C++ and Java APIs, a Python API added the preceding week, and Swift APIs coming. Hardware acceleration sits behind consistent application-facing APIs, though supported artifacts vary (earlier models had MediaTek and Intel support; the corresponding Gemma 4 artifacts were not yet published).
- The ordinary conversion path has four stages:
  1. Start with a model in Transformers.
  2. Use LiteRT Torch's generative optimizations and quantization workflow.
  3. Produce a LiteRT-LM artifact.
  4. Prototype with Gallery or integrate LiteRT-LM directly into the target application.
- For deeper customization, the Torch Generative API provides native PyTorch building blocks for common language-model architectures. It supports constructing and training a custom tiny model, not only fine-tuning an existing checkpoint, while arranging computation for efficient on-device execution.
- CPU or GPU uses a portable runtime/JIT path producing a shared LiteRT or LiteRT-LM file. An NPU uses a vendor compiler plugin ahead of time, producing an artifact specific to that NPU. Both are available through a consistent API, so the app integration stays similar even though the build path differs.
- Optimized libraries do much of the work: XNNPACK for CPU and dedicated GPU optimization code. The build changes more than the application API.

### Tiny models and fine-tuning · 46:09
- The export path is not limited to Gemma. Brick shows a Qwen example: Gallery can import any LiteRT-LM file and run it on a chosen backend (such as GPU). A Pixel or Samsung S25 are both common in the team's testing, though he cannot identify the demo handset definitively.
- An optional icon beneath a chat exposes prefill and decode statistics so developers can measure a compatible model on their own phone; `litert-lm run` supports desktop prototyping.
- **FastVLM**, Apple's roughly 500M-parameter vision-language model, repeatedly describes incoming video scenes and runs with Qualcomm acceleration. Brick estimates four-bit quantization could bring the added app payload to roughly 250–260 MB, but says that quantization was not performed for the demo. It is broadly useful within a bounded domain.
- Smaller checkpoints like **Gemma 3 270M** generally need specialization for a particular task, and small transcription and pairwise translation models offer other ready-made narrow capabilities.
- FunctionGemma's training examples cover dataset formatting and further function-calling fine-tuning. **Mobile Actions** specializes it for ten mobile functions; **Tiny Garden** uses voice-to-function calling in a game inside Gallery.
- Checkpoint suffixes describe what training has already happened: **`PT`** is a pretrained checkpoint before instruction fine-tuning (useful for expert users who want full control of behavior), while **`IT`** is instruction-fine-tuned. FunctionGemma deliberately starts from `IT` because function calling is already part of its learned behavior. Gemma 3 offered both kinds.
- **EmbeddingGemma** provides a roughly 300M-parameter text embedding model for retrieval — a useful tiny transformer, but not an autoregressive LLM. A complete app often needs models around the LLM too: voice activity detection and denoising can run as non-autoregressive models through LiteRT, while LiteRT-LM adds language generation. The architecture code also supports custom variants of Llama, Moonshine, Phi, and Qwen.

### A production example: offline dictation (and its limits) · 55:36
- **AI Edge Eloquent** turns the tiny-model workflow into a production iOS app: it records dictation and automatically polishes the transcript, removing interjections and cleaning up spoken phrasing.
- A personal vocabulary handles a common failure: a system might hear the technical term `LoRA` and write the name `Laura`. Eloquent accepts a biasing dictionary so the output favors the user's intended spelling. The core dictation and cleanup run offline with no per-use cloud charge.
- The vocabulary can be entered manually or populated through an optional Google-account connection that looks for unusual words (names are especially useful entries).
- The pipeline is modular: microphone input goes to a speech-recognition engine producing an unfiltered transcript, a separate personalization path supplies relevant terms, and both enter a dedicated tiny **text-polishing LLM**. Keeping recognition and polishing separate lets the app reuse recognition weights elsewhere and exposes the intermediate transcript for debugging. A single combined model is possible but separate models make reuse and inspection easier.
- The recognition and polishing models are unpublished derivatives from the Gemma 3 270M lineage. The training workflow uses a stronger cloud LLM to generate synthetic examples (Brick describes datasets from low single-digit millions to tens of millions of examples depending on ambition), then fine-tunes a tiny base. Eloquent's exact dataset size is not given.
- The polishing model's instruction fine-tuning teaches it to use supplied special words, correct similar-sounding terms, remove interjections, and interpret spoken revisions such as "Scratch that." This behavior is trained into the model, not implemented as a Gallery skill.
- Asked for before-and-after numbers on Mobile Actions, Brick recalls an improvement from the low 40s percent to about 86% across ten functions, with roughly eight simpler functions above 90% (around 93%) and two harder ones lowering the average. The public FunctionGemma release reports a different pair, 58% to 85%; the evidence does not establish that the two accounts use the same eval version.
- Across the team's tiny-model evaluations, he reports gains of roughly 20–40 evaluation points from fine-tuning. Full-fine-tuning Colab notebooks shipped with Gemma 3 270M and FunctionGemma.
- On safety: Brick defers model-level safety to the Gemma team. An OS vendor can add product-specific input and output checks around a system model, so the raw checkpoint and the deployed OS service are different safety surfaces. Tiny application models typically expose a narrower task and API — Eloquent transforms supplied content rather than inviting open-ended generation, which narrows (but does not remove) the safety problem.
- For multi-skill execution, switching skills across turns is easier to establish than chaining several within one request. Gallery supports multiple enabled skills, and an explicit chain (look up a topic in Wikipedia, summarize in three bullets, display flashcards) worked with specific prompting, but the team had only had the model a few weeks and was still exploring the limits. Brick notes there was no Gallery-specific training: the base model was trained for agentic workflows, thinking, and function calling, and the skill architecture came afterward.
- On context windows, the recording's recalled limits need correcting: Gemma 4 specifies 128K for E2B/E4B and 256K for the larger launch models. Model capacity is separate from Gallery's configured limit, which Brick recalls only approximately as 8K or 12K for performance. Long context consumes memory through the **KV cache** (cached attention state), and the small models were optimized for per-input-token cache requirements, though no bytes-per-token figure is given.
- On customization cost: for mobile E2B-class deployments, start with prompting and skills; for tiny models, fine-tune. LoRA (small trainable adapters added to a base model) is another route for larger local models; Apple's foundation-model framework supports adapters. For an embedded robotics platform running a two-billion-parameter model, he recommends task-specific LoRA adapters that can be hot-swapped while the base stays loaded, with adapter sizes roughly 8–100 MB depending on rank.
- On web deployment, the Colab workflow reaches a LiteRT artifact while LiteRT-LM browser support remains work in progress. The published tiny checkpoints discussed here belong to Gemma 3; Brick does not announce a future tiny Gemma 4 release.
- Availability has a product dimension too: one attendee reports being able to find Eloquent on the web but not download it from a European App Store. Brick says he will pass the feedback along.

## Resources

- ["Gemma 4 launch announcement"](https://blog.google/innovation-and-ai/technology/developers-tools/gemma-4/) — the four launch models, their capabilities, and context limits
- [LiteRT-LM](https://github.com/google-ai-edge/LiteRT-LM) — Google's open-source runtime for deploying language models on edge devices
- [Google AI Edge Gallery](https://github.com/google-ai-edge/gallery) — the mobile app for experimenting with local models and on-device skills
- [Build an AI Edge Gallery skill](https://github.com/google-ai-edge/gallery/blob/main/skills/README.md) — the `SKILL.md`, JavaScript execution, and secret-handling guide
- [FunctionGemma release and evaluation](https://blog.google/innovation-and-ai/technology/developers-tools/functiongemma/) — the 270M function-calling model and its reported accuracy
- [Fine-tune FunctionGemma](https://ai.google.dev/gemma/docs/functiongemma/finetuning-with-functiongemma) — the official tutorial for adapting FunctionGemma to specialized functions
- [Gemma 4 on LiteRT-LM: deployment and benchmarks](https://developers.google.com/edge/litert-lm/models/gemma-4) — cross-platform deployment and the prefill/decode benchmark table
- [Apple FastVLM](https://github.com/apple/ml-fastvlm) — efficient vision-language model variants for image-to-description tasks
