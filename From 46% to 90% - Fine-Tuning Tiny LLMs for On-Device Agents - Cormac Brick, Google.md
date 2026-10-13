# From 46% to 90%: Fine-Tuning Tiny LLMs for On-Device Agents

| Field | Details |
| --- | --- |
| **Speaker** | Cormac Brick |
| **Affiliation** | Google |
| **Type** | Talk |
| **Date** | 2026-05-20 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=-TiET_K-E_g) |
| **Tags** | `infrastructure-and-deployment` `architecture` `safety-and-governance` `vision-and-video` `reasoning-and-models` |

## Summary

Cormac Brick, a software engineer and tech lead on Google's AI Edge team, explains how to put an AI agent on a phone. There are two routes: lean on the intelligence the operating system already ships, or bundle a small model inside your own app. He uses **tiny LLM** to mean a model under a billion parameters — small enough to embed when the system AI does not cover your use case.

The talk tours the AI Edge stack (LiteRT-LM, MediaPipe, and the LiteRT runtime), a skills system built on Android's AI Core, and the workflow for packaging and deploying a custom tiny model. The headline result is a small function-calling model, FunctionGemma (270M parameters). Out of the box it succeeded about 46% of the time on the team's app-intent task; after fine-tuning, eight of ten functions cleared 90% and the remaining two landed in the 80s.

The broader message is that small models are specialists. At 100–200M parameters, a model needs one narrow job, and fine-tuning with synthetic data is what makes it dependable. Brick shows a production example — offline transcription built from two chained Gemma-3-based models — then takes audience questions on how many skills a small model can juggle and how LiteRT-LM replaces older MediaPipe packaging.

## Key Takeaways

- **Two ways to add on-device AI:** use the system model (Gemini Nano via Android's AI Core) or ship your own model inside the app with LiteRT-LM.
- **"Tiny LLM" means under a billion parameters.** Small enough to embed, but it needs a sharply focused task.
- **Try system AI first.** If Gemini Nano meets your need, using it adds no model size to your app.
- **Skills load on demand.** The prompt carries short descriptions; a load-skill tool fetches full instructions only when needed.
- **Skills can render UI.** A skill can carry JavaScript the app runs — a map or a roulette wheel, not just text.
- **Fine-tuning beats prompting at this scale.** FunctionGemma went from ~46% to >90% on eight of ten app-intent functions.
- **Chain small models for bigger jobs.** Offline transcription pairs a Gemma-3 ASR engine with a Gemma-3 polishing model.
- **One skill is easy; several at once is not.** Using different skills across turns is robust, but combining multiple skills in a single answer only sometimes works.

## Detailed Notes

### Two paths for on-device agents · 00:15
- The question: how do you add an agent skill to a phone when the built-in intelligence does not cover your task?
- **Path one:** a simple skill harness on top of Android's **AI Core** (the system service that serves on-device models).
- **Path two:** ship a model specialized for your app.
- Brick defines **tiny LLM** as a model with fewer than a billion parameters — small enough to embed.
- Both paths chase the same wins: lower latency, privacy, offline use, reliability, and possible savings.
- The real design question is where the model comes from and how much behavior the app must own.

### The Google AI Edge stack · 01:37
- Google AI Edge separates app integration from model execution.
- **LiteRT-LM** is the language-model harness for apps that bundle or download their own model.
- **MediaPipe** covers other model-driven tasks, such as face mesh.
- Both run on **LiteRT** (formerly TensorFlow Lite), the cross-framework runtime. You choose CPU, GPU, or NPU (a Neural Processing Unit, hardware that accelerates model math).
- LiteRT reports support for more than 2.7 billion devices, with a version built into Android itself.
- It also spans iOS, macOS, Linux, Windows, Web, and IoT. **Gemma 4** is part of that reach.

### System AI or in-app AI · 02:58
- **System generative AI** is pre-installed on the device. On Android the example is **Gemini Nano** through AI Core, shown with a summarization API; Apple has a parallel concept on iOS.
- The Nano base models Brick names are **Gemma 4 E2B and E4B**. They are optimized and preloaded, so using them does not grow your app.
- If the system model fits the task, it is the recommended starting point.
- **In-app generative AI** uses LiteRT-LM so the app (or even a webpage) carries the model. That means more control and broader device reach, at the cost of more integration work.
- The extra effort is worth it for a narrow, highly customized task.

### A skill harness on top of system AI · 05:06
- **Google AI Edge Gallery** is the team's playground for running models locally on Android and iOS. It uses AI Core for the Gemma model when available.
- It also demonstrates chat, image questions, audio transcription, and third-party models such as Qwen and Phi. It is built with LiteRT-LM, and the Android source doubles as an integration example.
- Demo: a "restaurant roulette" skill loads restaurants, picks a winner, and shows it on a spinning wheel.
- Gemma 4 runs inside a small agent harness; Brick describes this as a few lines of code or a good coding-agent prompt.
- What stands out: the result is an app interaction, not just a paragraph of generated text.

### Load skill details only when needed · 07:43
- The harness sends the app's system prompt plus a catalog of **skill descriptions** — enough for the model to know what exists.
- Full instructions stay out of the initial prompt. A **load-skill tool** fetches them on demand.
- Maps example: the model picks map navigation, calls load-skill, receives the instructions, then calls a "show JS" tool to draw the map in the app.
- Loading instructions and rendering are separate steps. Being inside an app is what makes rendering useful — a skill can carry JavaScript the app runs.
- **On-device inference is not the same as offline.** A skill that fetches external information still needs a network.

### Write, test, and share a skill · 08:59
- Skills can be written by hand from the authoring guide, but the team also uses Gemini CLI and Claude Code. They had made roughly 80 skills.
- Distribution loop:
  1. Create the skill (by hand or with a coding agent).
  2. Test it through **ADB** (Android Debug Bridge), using a CLI skill that drives a connected device.
  3. Publish the files to a personal GitHub repository.
  4. Load it into Gallery from a URL via the app's ellipsis menu.
  5. Share it in GitHub discussions.
- Community skills showed up within days of launch. Users can try a new skill without rebuilding Gallery.

### Package and deploy your own tiny model · 10:18
- LiteRT-LM runs a single-file model format and is open source and cross-platform. At talk time, C++ and Java APIs were available; Swift and JavaScript were coming.
- Deployment flow: start in **Transformers**, export with **LiteRT-Torch**, evaluate with the desktop reference runtime or Gallery, then integrate LiteRT-LM into the app.
- For tiny models, model choice and task design are tightly coupled. Use a prebuilt fixed-function model (vision-language or transcription) when one fits.
- Otherwise, narrow the task and fine-tune. At 100–200M parameters the model needs one focused job; the team used synthetic training data for this.
- Examples: a Qwen model on a desktop GPU via `LiteRT-LM run`, and Apple's **FastVLM-0.5B** optimized for a Qualcomm NPU. The 0.5B figure describes the language-model part; FastVLM also has a vision encoder.
- Two separate choices matter: which model fits the task, and which hardware executes it.

### Teach a small model your app's functions · 13:13
- **FunctionGemma** is a 270M-parameter model based on Gemma 3 technology, published with the DeepMind team in December. It calls functions robustly — provided it is fine-tuned for the task.
- Reported speed on a Pixel 7: nearly 2,000 prefill tokens per second and 140 decode tokens per second. (**Prefill** processes the input context; **decode** generates the output.) The talk does not give the backend, quantization, or input/output lengths.
- Uses include text-to-function and voice-to-function, turning a request into an action the app can run.
- A **FunctionGemma Tuning Lab** on Hugging Face lets you define functions, upload data, and fine-tune.
- App-intents example: adding a calendar item or an email. Out of the box, success was about **46%**. The team generated a synthetic dataset with **Flash** and trained on it.
- After fine-tuning: **over 90% for eight of ten functions**, with the other two in the 80s. The talk does not reconcile the earlier seven-function figure or provide a held-out split or scoring protocol. These are the team's reported results for one task, not a single aggregate accuracy number.
- The gain comes from specializing the model for the app's actions, at the cost of dataset generation and training.

### Chain specialized models for offline transcription · 15:52
- **Eloquent** is a transcription app that personalizes vocabulary generic tools often miss (jargon, names, preferred keywords). At talk time it was iOS-only and not yet in Europe.
- Pipeline built from tiny models:
  - **ASR** (automatic speech recognition): a Gemma-3-based engine turns speech into text.
  - **Text polishing:** a second Gemma-3-based model cleans the transcript, removing fillers like "um" and "ah".
- Each model is a few hundred million parameters. Chained, they give offline transcription with a personal dictionary.
- Rather than one model doing everything, the app combines models with distinct jobs.
- Brick frames it as proof that specialization can hold up in production.

### Q&A: skill limits and LiteRT-LM migration · 17:41
- Q: how many skills before a tiny model degrades? Brick answers for the 4B-parameter model (not the sub-billion ones), which he says chooses reasonably well among about **eight default skills**. The team had only weeks of experimentation.
- Across turns the model works reliably; picking one skill per turn is robust.
- Calling **multiple skills within a single answer** works only sometimes and is still being investigated.
- Q: where does LiteRT-LM fit relative to MediaPipe `.task` files? For stock LLMs, the LiteRT-LM format effectively replaces the `.task` bundle (a change made the previous year). `.task` files remain useful for other tasks like face mesh. LiteRT-LM also bundles the tokenizer.
- On CPU performance, Brick offers no number here. He points to the **Gemma LiteRT-LM model card** (updated with new platform numbers) and to a colleague's session the next day at 2:30 for platform benchmarks.

## Resources

- [Google AI Edge Gallery](https://github.com/google-ai-edge/gallery)
- [Gallery skill authoring guide](https://github.com/google-ai-edge/gallery/blob/main/skills/README.md)
- [LiteRT-LM runtime](https://github.com/google-ai-edge/LiteRT-LM)
- [FunctionGemma 270M](https://huggingface.co/google/functiongemma-270m-it)
- [FunctionGemma Tuning Lab](https://huggingface.co/spaces/google/functiongemma-tuning-lab)
- [FastVLM: Efficient Vision Encoding for Vision Language Models](https://arxiv.org/abs/2412.13303)
