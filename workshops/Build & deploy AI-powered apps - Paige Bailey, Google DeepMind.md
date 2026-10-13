# Build & deploy AI-powered apps

| Field | Details |
| --- | --- |
| **Speaker** | Paige Bailey |
| **Affiliation** | Google DeepMind |
| **Type** | Workshop |
| **Date** | 2026-04-29 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=G_bHFmEAarM) |
| **Tags** | `enterprise` `vision-and-video` `speech-and-audio` `apis-mcp-and-protocols` `robotics-and-world-models` `rag-context-and-search` |

## Summary

Paige Bailey, a developer-relations engineering lead at Google DeepMind, runs a
demo-heavy workshop on prototyping AI applications quickly in Google AI Studio.
Instead of slides, she builds things live: a dinosaur video turned into a cited
table, model-written Python running in a sandbox, a spoken conversation with
screen and camera input, and a full app with login and storage.

The guiding idea is to match the model and the tool to the job. Gemini is
multimodal both ways — it accepts video, images, audio, text and code, and can
return text, code, audio, images or interleaved images and text. When a question
needs information newer than the training data, Bailey adds grounding tools such
as Google Search, Maps or "URL context" (feeding it specific web pages to read).

She also shows how cheap small models can be. A tiny Flash Lite model drew
bounding boxes on a photo inside a hosted Python sandbox for a fraction of a
penny. The session closes on architecture, including a robot that plans with a
model but does not let that model drive its motors directly.

## Key Takeaways

- **Gemini is multimodal in and out.** It reads video, images, audio, text and code, and can output text, code, audio, images, or mixed images and text.
- **Give the model tools.** Search, Maps and URL context supply facts that are newer than training data or private to a page.
- **Small models plus tools go far.** Flash Lite solved the Lego bounding-box task quickly and cheaply by writing and running Python.
- **Get Code bridges demo and product.** It exports the selected model, prompt and settings as Python, TypeScript or Java.
- **Gemini Live unifies input.** Screen, video and audio join one conversation; language, accent and style can change mid-chat, while system instructions fix required behavior.
- **AI Studio Build makes real apps.** Bailey generated a bookshelf cataloger with Firestore storage and Google login, and it remembered books across sign-out and sign-in.
- **Generated worlds are pixels.** Project Genie (Genie 3) makes explorable environments, but not exportable 3D meshes.
- **Separate plan from action.** In the robot example, Gemini Live plans and delegates to local models; it does not control motors itself.

## Detailed Notes

### The model lineup and how to choose · 01:35
- Bailey separates models by the job each does, not by a single benchmark score.
- **Realtime:** Gemini 3.1 Flash Live. **Flagship and lighter tiers:** Gemini 3.1 Pro and Gemini 3.1 Flash Lite.
- **Media and structure:** Nano Banana 2 for images, Lyria 3 for music, Genie 3 for worlds, Veo 3.1 Lite for video.
- **Open and embeddings:** Gemma 4 is the open family, and a shared multimodal embedding space lets one search return a cat video, purring audio and books about cats.
- She notes Augment Code adopting Gemini 3.1 Pro for performance and cost, then points to a later Augment post describing adding Gemini alongside other providers.
- **Thinking levels** (minimal, low, medium, high) trade tokens for reasoning; she keeps minimal or low for interactive speed.

### Ground a dinosaur video into a cited table · 05:48
- AI Studio (ai.dev / aistudio.google.com) runs on a personal Gmail account and the free tier.
- Media input options: Drive, uploads, recorded audio, camera, samples, or a YouTube link — a URL avoids downloading and re-uploading.
- She enables Google Search grounding, adds a five-minute clip (sampled at one frame per second, roughly 27,600 tokens), and asks for a table of dinosaur types with timestamps and a fun fact each.
- The video supplies the visuals; Search supplies the facts, with inline citations.
- The result names Rexy, Brachiosaurus, Velociraptor and Pteranodon, and correctly notes that pteranodons are pterosaurs, not dinosaurs.
- **Get Code** exports the same experiment to code.

### Give smaller models executable vision tools · 13:06
- **Code execution** gives the model a hosted Python sandbox with data-science libraries, so the generated code runs there, not on the developer's machine.
- In compare mode she pits Flash Lite against Gemini 3 Flash on the same Lego photo, asking for bounding boxes around green bricks.
- Both succeed; Flash Lite answers first and cheaply (well under a fraction of a penny), while Flash takes longer and costs slightly more but still under a penny.
- The color thresholds and area filters are implementation choices; touching bricks can merge into one region.
- Related uses include segmentation masks and counting selected entities.
- Takeaway: enabling tools can close much of the gap between a small and a large model.

### URL context and model tiers · 16:20
- URL context is a lightweight form of retrieval for pages you already know: supply the URLs and let the model read them.
- She feeds blog posts about Gemma 4 and Genie 3 — both released after the training cutoff — and asks for similarities and differences.
- The model lists its sources and cites them inline as it makes assessments.
- Vertex offers retrieval over internal documents without setting up your own vector database.
- Tier positioning: 3.1 Pro (largest, slower, pricier), Gemini 3 Flash (production workhorse), 3.1 Flash Lite (gaining users from earlier Flash models).

### Live screen, speech, and camera · 19:34
- **Gemini Live** accepts screen share, video and audio in one dynamic conversation, with custom function calls and Search grounding.
- Bailey shares her screen and asks what the model sees; it describes the Lego search page.
- An Italian-speaking attendee verifies responses in Italian, including London's weather that day.
- A "Texan accent" poem shows style can change on request.
- Practical example: a bank kiosk that replies in the language a visitor starts with. For a fixed language or tone, set **system instructions**.
- Camera input identifies two fingers, then a thumbs-up — one integrated interaction instead of stitching together speech-to-text, an LLM, text-to-speech and video handling.

### Build a bookshelf catalog app · 24:51
- **AI Studio Build** compares to v0 or Lovable: create, edit, share and deploy, with databases, Google login and securely held custom API keys.
- Requirements were dictated by voice: upload a bookshelf photo, identify titles and authors (with Search descriptions and categories), require Google sign-in, and save per user so it persists.
- The data model keeps owner separate from book metadata to make the per-user rule explicit; enforcement also needs database access rules.
- Selecting Firestore and Firebase Google auth sends Gemini 3.1 Pro Preview off to plan the app, database and authentication in the background.
- At this point the app is requested behavior, not a finished product.

### Generated worlds and media · 29:17
- **Project Genie** (Genie 3) generates frames dynamically as you explore; there is no Unity, Unreal or physics engine underneath.
- Movement is WASD and the view uses arrow keys; learned behavior mimics collisions.
- She creates a Big Bend National Park scene with Lego rock formations and a pink ostrich, and the generated world can differ from the prompt.
- **Key limit:** the world is pixels, not meshes — nothing to import into Unity, though the audience asks exactly that.
- Nano Banana 2 adds image-search grounding to place a supplied dog in a supplied park with a found branded can.
- A Lego-to-SVG test shows recognisable but distorted bricks; adding code execution produces an inspectable geometric intermediate (a blue brick with outline points), not a finished improved SVG.
- For video, Gemini writes an expanded prompt for Veo 3.1 Lite (720p; 16:9 or 9:16; four-, six- or eight-second clips; 4K needs a higher tier). Veo is a paid model.

### Open models, persistence, and robotics · 50:34
- **Gemma 4** is Apache 2.0; AI Studio and its API let you evaluate the models for free before downloading, and modality support differs by variant (native audio only in E2B and E4B).
- Chrome's built-in language model is **Gemini Nano**, not Gemma — a naming distinction Bailey flags.
- **Lyria 3** (Pro and Clip previews) provided a Spanish theme song for the food-truck concept, using a gallery app whose source is inspectable.
- The finished bookshelf app recognised titles and authors, and books persisted across sign-out and sign-in. She made it public and shared a QR code, but declined the one-button Cloud Run deploy to conserve quota.
- **Robotics:** Pupper, an open-source Raspberry Pi quadruped, uses Gemini Live to interpret flexible instructions and build a plan, then calls local robot models to act. It does not directly control movement.
- In Q&A she says an AI Studio app is planned, points to using Gemini APIs with OpenClaw for existing workflows, and notes free tiers for Gemma and some Gemini models.

## Resources

- [Gemini video understanding guide](https://ai.google.dev/gemini-api/docs/video-understanding)
- [Build full-stack apps in AI Studio](https://ai.google.dev/gemini-api/docs/aistudio-build-mode)
- [Gemini code execution](https://ai.google.dev/gemini-api/docs/code-execution)
- [Gemma 4 launch announcement](https://blog.google/innovation-and-ai/technology/developers-tools/gemma-4/)
- [How Project Genie works](https://blog.google/innovation-and-ai/models-and-research/google-deepmind/project-genie/)
- [Chrome's built-in AI APIs](https://developer.chrome.com/docs/ai/get-started)
- [Stanford Pupper control software](https://github.com/stanfordroboticsclub/StanfordQuadruped)
