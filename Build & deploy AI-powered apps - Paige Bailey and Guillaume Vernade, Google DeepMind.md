# Build & deploy AI-powered apps

| Field | Details |
| --- | --- |
| **Speaker** | Paige Bailey, Guillaume Vernade, Ian Ballantyne |
| **Affiliation** | Google DeepMind |
| **Type** | Workshop |
| **Date** | 2026-05-23 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=ns9f1fjLD7Y) |
| **Tags** | `creative-and-generative-media` `infrastructure-and-deployment` `rag-context-and-search` `robotics-and-world-models` `speech-and-audio` `industry-applications` |

## Summary

This workshop is a hands-on tour of building and shipping AI apps with Google DeepMind's tools, split across three presenters. Paige Bailey starts with prompt-driven app building in AI Studio and a playable Genie world assembled from several generative models. She then builds "Shelf Scan", an app where a user signs in with Google, photographs a bookshelf, and keeps a searchable library tied to their account. The arc is honest about failures: recognition worked before saving did, and the app only worked after the save path was repaired.

Guillaume Vernade follows with a book-to-media pipeline. From a public-domain novel he generates character portraits and chapter illustrations, animates them with Veo, scores chapters with Lyria music, and narrates scenes with two configured text-to-speech voices. A recurring theme is picking the right model for each output and making the pipeline observable, instead of letting one long conversation carry all the context.

Ian Ballantyne closes by running Gemma 4 open models on hardware the developer controls: phones, laptops, and a local server. He demonstrates small coding loops on a phone, serving a 26B model through a local endpoint, and pointing tools like OpenCode at it. His final message is to test the actual artifact — a generated webpage that looks right is not the same as a generated game that runs.

Why it matters: the workshop pairs impressive model output with concrete engineering caveats about cost, capacity, context, and the gap between a plausible result and a working app.

## Key Takeaways

- **Don't build around a temporary constraint.** Early 8K/16K context windows drove vector databases; longer contexts changed the calculus. Bailey argues opinionated, customer-built use cases endure even as models improve.
- **Model choice is a policy choice.** In compare mode, Flash-Lite returned bounding boxes quickly while Flash Preview built a mask to verify them — same output, more cost and tool calls.
- **A working app is a full loop.** Shelf Scan only counted as done after recognition, save, and reload all worked; a plausible extraction was not enough.
- **Track characters explicitly.** Vernade fixed a mismatched character by giving each chapter a list of names and attaching only those reference images, rather than trusting chat history.
- **Cost adds up fast.** A light Veo notebook run was about €1, a separate Veo notebook ran about $20, and service tiers trade price for latency.
- **Open models run on hardware you own.** Gemma 4's E2B/E4B target phones and edge devices; the 26B mixture-of-experts model ran on an M4 Mac in about 22 GB with context.
- **Keep generated code reviewable.** Vernade instructs the app builder to split files by feature and add logs, so failures are diagnosable instead of hidden.
- **Test the artifact, not the screenshot.** The webpage reconstruction looked close, but the small Omniforge editor that actually ran interactions was the more meaningful result.

## Detailed Notes

### From scientific Python to model infrastructure · 0:00:15
- Paige Bailey started machine learning around 2009–2010 and contributed to NumPy, SciPy, Matplotlib, and scikit-learn.
- Her background spans geophysics and applied math; at Chevron she worked on velocity modeling, drilling optimization, and large-scale compute with Cloudera and Spark.
- Geoscience gave her early CUDA experience; she then helped TensorFlow with CPU, GPU, and TPU execution paths, single-node and distributed.
- Open source work led to Google, then a year at GitHub on VS Code and early Copilot UX, then back to the PaLM 2, Gemini, and Gemma teams.
- Her point on open source: it lets you test whether a team is worth joining and makes your own work discoverable.

### Which constraints will survive the next model? · 0:04:42
- Bailey cautions against sprinting to build what the model will soon absorb — her test is that everyone rushing the same way often signals the wrong bet.
- Example: early 8,000- and 16,000-token context windows pushed developers toward vector databases; longer contexts reduced that need.
- Similar story with language-specific fine-tunes, which broader multilingual support made less necessary.
- She expects models to absorb more agent-framework behavior, sees a shift to Markdown-based "skills", and imagines automatic skill creation from activity.
- Audience pushback: bioinformatics may stay niche. Bailey cites MedLM and Med-PaLM, now largely replaced by Gemini with retrieval, because medical text entered general training.
- She accepts stochasticity as unavoidable, but argues an opinionated use case built directly with customers stays valuable as models improve.

### A toolbox of models and app surfaces · 0:08:07
- A short computer-use agent runs on screen before Bailey disconnects it to avoid showing private material.
- Conversation and reasoning: Gemini 3.1 Flash Live, Gemini 3.1 Pro, and Flash-Lite. Augment Code reportedly defaulted to Pro for agent infrastructure on performance-to-cost.
- Images and retrieval: Nano Banana 2 for image generation and editing; Embeddings 2.0 places video, audio, images, code, and text in one shared embedding space.
- Generated media and worlds: Lyria 3 (music), Genie 3 (interactive worlds), and Veo 3.1 Lite (video).
- App surface and open models: AI Studio's full-stack runtime adds databases, OAuth, and custom API keys; Gemma 4 ships under Apache 2.0.
- Bailey separates accepting multiple input modalities from producing them — Gemini can output text, code, images, interleaved text and images, and audio tokens.

### Prompt demos: video tables, verification, and a bookshelf app · 0:11:51
- In the AI Studio playground she picks Gemini 3.1 Flash-Lite for speed and cost ($0.25 per million input tokens, $1.50 per million output at launch).
- She attaches a YouTube Carnotaurus video, selects 0–300 seconds, enables Google Search grounding, and asks for a table of timestamps, dinosaur kinds, and a fun fact.
- The input estimate was roughly one frame per second and about 30,900 tokens; the model sees real frames, not just the video's metadata.
- The answer identifies Carnotaurus ("meat-eating bull"), Triceratops, and notes that Pteranodons are flying reptiles, not dinosaurs.
- **Get Code** exports the model, video URL, offsets, and prompt into TypeScript or Python.
- In compare mode, the same Lego image goes to Flash-Lite and Gemini 3 Flash Preview with code execution; both draw green-brick bounding boxes, but Flash verifies its coordinates against a mask — slower and costlier, yet still pennies.

### A world generated as pixels, and the Shelf Scan test · 0:28:56
- Project Genie combines models into an interactive experience where frames are generated from actions, not from a Unity or Unreal scene with a physics engine.
- A live attempt asks for Regent's Canal on a sunny day, dolphins, pirate flags, and a pink sparkly squirrel player; a generated first frame acts as a checkpoint.
- The public prototype limits exploration to 60 seconds; the squirrel walks on water and the dolphins stay still — a gap between plausible motion and explicit physical rules.
- Shelf Scan (AI Studio Build) lets a user sign in with Google, upload a bookshelf photo (WebP), and keep a catalog enriched via Search grounding.
- The first upload recognized about seven books but the library stayed empty; the real error was a Firestore permission problem tied to save-path validation.
- After the repair the catalog populated with titles, authors, and descriptions — including details not visible on the spines — and survived logging out and back in.
- Bailey's standard: the recognition–save–reload cycle is the test, not a plausible response.

### Book-to-media pipeline: characters, animation, music, narration · 0:43:11
- Guillaume Vernade frames multimodality as both understanding different inputs and responding through different outputs; he notes a rough pace of one generative-media model or capability every five days.
- Model choices by output: Nano Banana 2 (aspect and size controls), Nano Banana Pro (search vs image grounding), Veo 3.1 Lite (prompt testing, billed per second with audio), and Lyria 3 / Lyria real-time (songs and steerable continuous music).
- The notebook runs on Kenneth Grahame's *The Wind in the Willows* from Project Gutenberg; setup installs the SDK, sets an API key, and enables retries because Nano Banana 2 had capacity contention.
- He compares API surfaces: consumer Gemini (easiest, least control), Vertex AI (enterprise and regional control, more setup), and AI Studio / Developer API (fast start, fewer enterprise controls).
- Structured output constrains responses to usable fields, but chat history alone is a weak reference strategy — as the cast grows it gets fragile.
- The fix: each chapter records a name, a prompt, and an explicit character list, and the app attaches only those portraits; missing references raise errors before generation. Toad then matched his earlier portrait.
- Animate and score: the largest Veo model uses the latest image as a start frame; a longer continuation prompt did not produce a better clip, so results must be inspected. Gemini writes chapter prompts that `generate_content` sends to Lyria for mood-appropriate music.
- Narration rewrites an excerpt as a play: a narrator label (voice `Sulafat`) and a shared character label (voice `Fenrir`), with pace, accent, and emotion creating the audible difference.
- To turn the notebook into an app, Vernade pastes it into AI Studio; his coding rules are to split files by feature, add workflow logs, and account for quota with a paid key. One build took roughly 1,000 seconds.

### Gemma 4 on hardware you control · 1:24:26
- Ian Ballantyne introduces Gemma 4 across four models. E2B and E4B target phones, Raspberry Pis, and Jetson-class devices.
- Their effective size reflects per-layer embeddings that stay in flash and page in when needed — roughly 2B/4B effective cores versus about 5B/8B with embeddings held in RAM.
- Larger options: 26B A4B (mixture-of-experts) and a 31B dense model. Active parameters describe compute per token, not what must be stored.
- Thinking is configurable; image and video understanding are supported, but audio input is limited to the two effective models.
- Ballantyne describes performance in the range of models ten times larger, though without a benchmark; his concrete comparison is the capability mix — Gemma 2 leaned creative writing, while Gemma 3 and 4 broadened coding, function calling, and action selection.

### Serve a local model and test what it builds · 1:32:47
- Google AI Edge Gallery (Android and iOS) is a playground for device-local models; E2B selects among Agent Skills like Android Intents, JavaScript, and WebViews.
- Gemma also integrates with Ollama, LM Studio, vLLM, and SGLang. Ballantyne loads 26B in LM Studio on an M4 Mac at about 18 GB, or about 22 GB with the demonstrated context.
- LM Studio exposes the model through a compatible endpoint on port `1234`; an existing Python client only needs `base_url="http://localhost:1234/v1"` and the local model ID.
- He runs one orchestrator with ten sub-agents, each drawing a different SVG, with thinking off for speed; the reported throughput is combined across sub-agents, and this workload runs offline.
- OpenCode points at the same endpoint by editing `.config/opencode/config.json`; when local RAM is short, he points to a Cloud Run deployment guide.
- Live coding: 31B had already written a spec for "Nebula Drift", an asteroid racer. OpenCode gave it shell, filesystem, and edit tools, but the first run produced a page that did not play.
- A syntax error copied into the chat triggered a repair; a failed exact-match edit and a possible model unload ended with an explicit full rewrite of `game.js`. 31B was better at choosing sub-agents; 26B sometimes needed stronger direction.
- Follow-ups: an audience idea became "Omniforge" (build-your-own-game); a screenshot of a webpage became a single-file HTML rebuild; Nebula Drift finally ran with a star field and movable ship but no asteroids, so it stayed only partly implemented.
- Open Duck carries the same action-selection idea into a robot simulator, where E2B runs in the browser via WebGPU and picks from the robot's supported actions.
- The closing test is interaction: the webpage reconstruction looked close but its buttons were unverified, while Omniforge actually opened an editor, accepted object placement, and ran a scene with a moving triangle.

## Resources

- [Gemini 3.1 Flash-Lite launch](https://blog.google/innovation-and-ai/models-and-research/gemini-models/gemini-3-1-flash-lite/)
- [Gemini API pricing](https://ai.google.dev/gemini-api/docs/pricing)
- [Build apps in Google AI Studio](https://ai.google.dev/gemini-api/docs/aistudio-build-mode)
- [Project Genie introduction](https://blog.google/innovation-and-ai/models-and-research/google-deepmind/project-genie/)
- [The Wind in the Willows (Project Gutenberg)](https://www.gutenberg.org/ebooks/289)
- [Gemma 4 model card](https://ai.google.dev/gemma/docs/core/model_card_4)
- [Google AI Edge Gallery](https://github.com/google-ai-edge/gallery)
