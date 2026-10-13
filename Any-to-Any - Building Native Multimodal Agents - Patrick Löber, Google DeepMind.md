# Any-to-Any: Building Native Multimodal Agents

| Field | Details |
| --- | --- |
| **Speaker** | Patrick Löber |
| **Affiliation** | Google DeepMind |
| **Type** | Talk |
| **Date** | 2026-05-20 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=GIRpQEfYf3U) |
| **Tags** | `agent-engineering` `vision-and-video` `rag-context-and-search` `apis-mcp-and-protocols` `speech-and-audio` |

## Summary

Patrick Löber, a member of technical staff on the Gemini API and AI Studio at Google DeepMind, explains how to build "any-to-any" agents that understand and produce more than text. The goal is a small NotebookLM-style app: feed in several sources — a PDF, images, a lecture video, voice memos — and have an agent turn them into learning materials such as a study guide, an infographic, and a short spoken explanation.

The pattern has two phases. First, multimodal understanding: Gemini reads mixed inputs together and produces one synthesized study guide. Second, agentic generation: Gemini acts as a reasoning model that decides which supporting asset to create, then calls specialized models for image and speech generation through function calls.

Löber covers the practical details — one generation request that holds all the sources, audio and video token budgets, context caching for repeated questions, and function declarations. He then adds the Live API for real-time audio-to-audio conversation with a camera feed, and points to related models for multimodal embeddings, local understanding with Gemma, and video generation with Veo.

## Key Takeaways

- **Any-to-any means many capabilities, not one model.** Gemini 3 understands multiple modalities but returns text; separate models handle image and speech generation.
- **An agent, not a fixed workflow, chooses the next asset.** A reasoning model decides whether another diagram or audio clip would help and calls the tool itself.
- **Cross-modal understanding is the point.** The model should connect information across sources, not summarize each file on its own.
- **One request can hold all the sources.** Upload files, then place them together in `contents` with an instruction that explains what they are.
- **Media costs context.** Audio runs about 1,920 tokens per minute; video is roughly an hour at default resolution, more with lower resolution or timestamp limits.
- **Context caching helps repeated queries.** Löber reports 90% cost savings when reusing long loaded files for repeated questions.
- **Tool descriptions advertise; the prompt sets the criteria.** Function declarations define what a tool does, and the prompt says when to call it.
- **Native models understand the world.** Shared training lets the image and speech models use meaning and context, not just surface appearance.

## Detailed Notes

### What does an any-to-any agent need? · 00:31
- Löber works on the Gemini API and AI Studio at Google DeepMind; the session builds a small NotebookLM clone.
- **Gemini** is natively multimodal: it accepts code, images, audio and video, plus URLs and Google Search, and can output text, images, speech, video, function calls and code.
- The capability diagram describes an API ecosystem, not one model that does everything yet.
- In the current lineup, **Gemini 3** handles multimodal understanding and returns text; specialized models handle native image and speech generation.
- **Nano Banana** is named as the image-generation model family based on Gemini; **Gemma** is offered as a local option with text, image and video input (and audio on smaller models).
- He organizes the talk around four things: multimodal understanding, native image generation, native speech generation, and the Live API.

### Synthesize the sources, then choose the assets · 02:56
- NotebookLM is the product reference: feed in several sources, then generate an audio overview or an infographic.
- Here the goal is different: the agent, not a hard-coded pipeline, decides what to create.
- **The distinction between an agent and a fixed workflow is who chooses the next asset.**
- The architecture has two phases:
  1. **Understand the sources:** turn mixed inputs into a synthesized study guide.
  2. **Generate supporting materials:** give Gemini the guide plus tools for images and speech; after a tool returns, the model judges whether enough material exists.
- The output is a mix of text, visuals and audio, not a fixed requirement that every source produce every type.
- The example topic is "Attention Is All You Need", with the paper, images, a lecture or tutorial video, and voice memos as inputs.
- The useful capability is **cross-modal understanding**: the model connects information across the sources.

### Put the sources in one generation request · 04:50
- The SDK path starts with a free API key from AI Studio and an installed Gemini SDK.
- Upload the PDF, video and MP3 through the File API, or pass smaller files as inline data.
- He recommends the Gemini API development skills, which give a coding agent guidance for working with the API.
- The central call is `client.models.generateContent`: put the sources and the instruction together in `contents`, explain what the files represent, and ask for a synthesis; in the walkthrough he names `gemini-3-flash`.
- The prompt supplies the relationship between the files, which is what turns a pile of uploads into a request for a coherent study guide.

### Budget the media context · 06:09
- Audio understanding can also do plain transcription: ask Flash or Flash Lite directly to produce a transcript rather than building a separate pipeline.
- He gives an audio rate of 1,920 tokens per minute and a one-million-token context window, describing that as more than nine hours of audio.
- For video, he gives a rough capacity of one hour, with configuration options to go longer.
- You can narrow the task to a time interval — for example, analyse only minutes 5–15 — use the File API for larger uploads, or pass a YouTube URL directly.
- **Context caching** helps when the same long sources support repeated questions; Löber reports 90% cost savings for repeated queries over long loaded files.
- At this point the application can read a source collection and produce a summary.

### Generate assets and let the model choose · 07:56
- The second phase uses Gemini as the reasoning model with function calls connecting it to specialized generators.
- Image generation uses the same `generateContent` entry point with a different model and a prompt describing the desired picture; he names the infographic use case, where the model produces slide-style graphics.
- He calls the image model Nano Banana 2 (spoken as a "Gemini 3.5 Flash image preview" variant).
- Speech generation uses a text-to-speech model based on Gemini 2.5, configurable for two speakers so it can resemble a podcast conversation; he plays an excerpt explaining transformers in about two minutes.
- A **function declaration** tells the reasoning model what a tool does and what arguments it accepts — a name, a description, and a parameter schema.
- Both tools take one string: a detailed description of the image, and content plus delivery instructions for the audio.
- The prompt gives the criteria: call `generateImage` for concepts that need a diagram, and `generateSpeech` for sections that benefit from an audio summary. The declarations request an asset; the application connects the choice to the real generator.

### Why native understanding matters to generation · 11:18
- Löber attributes native generation's usefulness to the training shared with the main Gemini models, so the generator uses knowledge of the world and the meaning of the input.
- **A viewpoint from a map:** in a Nano Banana 1 example, arrows drawn on a map give a viewing direction, and the request is for a picture of what would be visible there — the output depicts the Golden Gate Bridge.
- **Homework corrections:** a Nano Banana 2 example returns an image of math homework with corrections, which needs understanding of the maths as well as the picture.
- The image models can also render code inside images.
- For speech, the expressive capabilities are multilingual output, accents and tone; he plays British and other accent samples and notes that not every accent is supported.
- Understanding plus generation already gives the core study application; real-time interaction sits on top.

### Add a conversation that can hear and see · 13:50
- The **Live API** adds real-time interaction, using **Gemini 3.1 Flash Live**, which he calls a native **audio-to-audio** model: audio goes in and audio comes out through a single architecture rather than a cascade of separate models.
- He says this enables more natural-sounding conversation.
- With limited time he plays a prerecorded demonstration from his colleague Thor rather than running it live.
- The interaction starts with speech, then adds a camera feed; when the demonstrator asks whether the assistant can see him, it describes a dark jacket over a blue shirt.
- The useful transition is from a conversation about an unspecified scene to an answer grounded in the current camera view; he tentatively mentions configuring the accent.
- AI Studio's Live interface is the suggested place to try it.

### Extend the same architecture · 15:22
- The study app is one use of a transferable pattern: understand heterogeneous inputs, reason over the result, and invoke the output capability the task needs.
- Three extensions:
  - A **multimodal embedding** model maps different modalities into one shared vector space, enabling multimodal search.
  - **Gemma 4** offers a local multimodal-understanding option.
  - **Veo** adds video generation with native audio.
- Each expands a different part of the application — finding source material, running understanding locally, or producing another output medium — without changing the separation between understanding, deciding and generating.

## Resources

- Attention Is All You Need — the Transformer paper used as the study agent's source topic.
- Gemini API development skills — installable coding-agent guidance for building with the API.
- Build with Nano Banana 2 — Google's introduction to the image model (also named Gemini 3.1 Flash Image).
- Gemini documentation: audio understanding, video understanding, image generation, and context caching.
- Gemma 4 model overview — open-weight multimodal models and local deployment.
