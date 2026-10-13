# Building Conversational Agents: From Stateful Tool Loops to Live Audio

| Field | Details |
| --- | --- |
| **Speaker** | Thor Schaeff, Philipp Schmid |
| **Affiliation** | Google DeepMind |
| **Type** | Workshop |
| **Date** | 2026-04-30 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=cVzf49yg0D8) |
| **Tags** | `speech-and-audio` `apis-mcp-and-protocols` `enterprise` `coding-and-developer-tools` |

## Summary

Thor Schaeff and Philipp Schmid, developer-experience engineers at Google
DeepMind, split this hands-on workshop into two parts. Philipp builds a small
coding agent — read files, write files, run shell commands — on the new
Interactions API, which unifies models and agents and can keep conversation
history on the server. Thor then builds real-time voice experiences with the
Live API: a Gemini "DJ" that generates music through Lyria, camera and screen
understanding, and browser apps that reach the model either through a proxy
server or a direct WebSocket using a short-lived token.

A recurring message is how much the application still has to do itself. Even
with server-side state, the client runs the tool loop: it inspects outputs, runs
the requested functions, submits results and repeats. And having a tool
available is not the same as using it correctly.

The demos are refreshingly honest. Weather grounding and some custom tools fail
and are left unresolved, while other effects (an alert, a green background)
work. The presenters turn those gaps into a lesson: test end to end, and treat
enjoyable demos as starting points rather than proof that every configured
capability is production-ready.

## Key Takeaways

- **The Interactions API (public beta) unifies models and agents.** A request names either a model like Gemini 3 Flash or an agent like Deep Research.
- **Typed content blocks** give every input and output an explicit `type` — text, image, audio, video, function call, thought signature — making app code easier to dispatch on.
- **Server-side state** means a follow-up sends new input plus the previous interaction ID; you can also manage history client-side when you need to edit it.
- **Keeping history intact helps caching.** Philipp estimates cached input tokens at ~90% cheaper, with 2–3× better cache-hit rates among some startups (their figures, not a controlled test).
- **An agent is four parts:** a model, tools, context (instructions and constraints) and a loop that runs until no more tools are requested.
- **A function call is a request to act;** the function result records what the app actually did. Tool availability and instructions about when to use tools are separate.
- **The Live API is a stateful WebSocket** for streamed audio, text and video (max ~1 fps), processing sound directly with no speech-to-text → LLM → text-to-speech cascade, and supporting barge-in.
- **Two browser architectures:** proxy media through your own server (easier, extra hop, worse latency) or mint a short-lived ephemeral token and connect the client directly.

## Detailed Notes

### Setup and the two halves of the workshop · 01:43
- Both presenters work on developer experience at Google DeepMind, covering the Gemini API and AI Studio.
- To use the API you need a secret key: in AI Studio via `ai.dev`, open Get API key (also `ai.dev/api-key`), create or import a project, then create a key.
- Only a Google account is needed; the exercises fit the free tier and need no credit card. Treat the key as a secret — do not commit it. Philipp deletes his.
- Using Antigravity does not require this key setup.
- Attendee projects set the scene: one runs Gemini with OpenClaw on Meta Ray-Ban glasses (VisionClaw keeps the Live WebSocket on the phone); another is building a first AI customer-support agent.

### One interaction surface for models and agents · 08:39
- The **Interactions API** launched in public beta in December 2025, intended as the successor to `generateContent`.
- A request names either a **model** (e.g. Gemini 3 Flash) or an **agent** (e.g. Deep Research, which plans and researches for roughly 10–15 minutes across hundreds of sites).
- It follows conventions familiar from Chat Completions, Anthropic and OpenAI, and aims to chain models: Nano Banana for images, Flash for processing, Lyria for audio, and maybe Veo for video.
- **Typed content blocks:** every input and output carries an explicit `type` (text, image, audio, video, function call, thought signature), which is easier to inspect than a protobuf-style structure.
- Bringing or defining custom agents is future work.

### Server-side history and implicit caching · 11:24
- Server state lets the client send new input plus a previous interaction ID; the server attaches it to retained history.
- Long agent work can use `background: true` and polling instead of holding a connection open; streaming uses server-sent events, and webhooks are forthcoming.
- Server-managed vs client-managed history: server continuation is simple, while client management lets you remove or rewrite parts of the context.
- Retaining history helps **implicit caching**. Philipp estimates cached input tokens are ~90% cheaper and reports 2–3× better cache-hit rates among some startups using the API.
- Rebuilding history on the client invites tiny changes — dropped line breaks, stripped whitespace — that can disrupt cache reuse.

### The agent loop and function calls · 14:31
- An agent has four parts: a **model** chooses actions, **tools** interact with the environment, **context** describes the task and limits, and a **loop** repeats until no more tools are requested.
- Even with server-side state, the client runs the tool loop.
- A **function declaration** gives its type, name, description and parameter schema. When the model needs action, the app inspects outputs, finds function-call blocks, runs the local functions, submits results, and repeats.
- **A function call is a request to act; the function result records what the application actually did.**
- The same continuation can cross models: research with Deep Research, then a visual generated by Nano Banana.

### The smallest stateful coding agent, tools and instructions · 17:29
- Install the Gemini API **skill** first; the docs distinguish general, Live API and Interactions skills. This exercise uses Interactions, installed project-locally (e.g. `.agents/skills/gemini-interactions-api`).
- The skill is deliberately small: it prevents unreliable choices (such as an obsolete model) and points to Markdown documentation the agent can fetch. Ask the coding agent which skills it can use to verify discovery.
- The implementation defines an `Agent` class in `workshop.py`; `run` sends input and stores the previous interaction ID, and a name-recall test confirms memory.
- Then add `readFile` and `writeFile`: each tool needs a JSON schema (the arguments to generate) plus a Python function (what to execute), linked in a tools map. The app checks a requested tool exists before dispatching.
- A `stdin` loop makes it interactive. A thumbs-up SVG request initially writes nothing; making the role explicit ("a coding assistant that can use tools with the local filesystem") and adding system instructions produces the `writeFile` call.
- **Tool availability and instructions about when to use tools are separate.** A shell tool uses Python's `subprocess`, returning standard output; Philipp explicitly sets security aside for that example.

### Branching history, expiry, and context limits · 42:55
- Because IDs live on the client, you can branch: select an earlier interaction and continue differently — for example, use one research result as the base for several parallel requests.
- `interactions.get` retrieves an interaction and its previous ID, so following those links walks history backward, which is useful for export.
- He states default retention of one day on the free tier and 55 days for paid usage. Storage duration is separate from context size: he describes a million-token limit that produces an error when exceeded.
- Compaction remains the client's responsibility here. After retention expires, an old ID can no longer be found — expiry of stored interactions, not a daily conversation reset.
- Caching works at the object level: a 4,000-token PDF might be cached while a ten-token text input is not; whitespace changes and request timing affect whether a hit occurs.

### Live API: a DJ, the socket, and voice grounding · 49:02
- **Gemini 3.1 Flash Live** is a newly released native-audio model tuned for latency and scale; the Live API remains separate from Interactions for now.
- **Live Jukebox** gives the conversational model a DJ persona and a Lyria 3 music tool (clip model makes ~30-second songs with lyrics). It requires a paid key because it generates music.
- The DJ clarified a request, then produced a German-techno-schlager track with lyrics about London, robots in bowler hats and builder's tea; a later request (Swahili nursing song plus hardcore techno) played but with the wrong vibe.
- The **Live API** is a stateful WebSocket accepting text, streamed audio and video frames (max ~1 fps) from a camera, canvas or screen share. It processes sound tokens directly, with no speech-to-text → LLM → text-to-speech cascade.
- Thinking levels change how much reasoning precedes a response; roughly 97 languages are supported in preview; automatic voice activity detection enables barge-in.
- The direct transport is WebSocket-only here. Applications wanting WebRTC use integrations such as LiveKit, Pipecat, Fishjam, Vision Agents or Voximplant; GPT Realtime ships WebRTC directly.

### Voices, grounding failures, and browser architectures · 59:24
- In AI Studio's Live view, webcam answers identified a green jacket, blue T-shirt and black cap. The base voice is **Puck**, and roughly 30 base voices exist; a system instruction asked for a friendly Irish accent.
- Weather is a grounding test: the model first said about 9°C and cloudy. Challenged, it named April 8, 2026 and gave 5–13°C with rain. Thor disputed it and suspected a grounding or UI problem; no fix was established. **A fluent spoken answer does not prove the intended source was used.**
- Two browser architectures: a **server proxy** (browser → FastAPI server → Live session; easier but adds a hop and latency) versus an **ephemeral token** (a small backend mints a short-lived token the client uses to connect directly).
- The proxy demo showed worse latency; the direct demo kept the long-lived key on the backend and used raw WebSockets in the frontend.
- The direct-connection demos produced visible custom-tool effects (a Hello World alert, a green background) but then a refusal to restore white, and the weather lookup still failed. Another attendee also reported Search grounding failing.

### Longer sessions, production choices, and speaker authorization · 1:25:07
- Live session limits (uncompressed): about 15 minutes for audio-only and two minutes for audio-video. A go-away notification precedes termination; **context-window compression** extends sessions by sliding a window over older context, so dropping video frames frees room for conversation.
- Production examples: Shopify Sidekick (in production), Stitch voice-directed design (a Google product), Waymo integration prospective, and Ato, an elder-companion device from an Argentina-based startup.
- **Native audio** gives natural conversational flow with less pre-speech control; a **cascading pipeline** exposes intermediate stages you can inspect or rewrite — sometimes a better business fit.
- Live sessions do not offer a retrievable transcript archive; the application must store the events it needs (LiveKit and Pipecat help here).
- **Recognizing speech is not authorizing its speaker.** Thor offers no reliable enrolled-speaker-only capability; proactive audio filters irrelevant talk but is not dependable authorization. An attendee's NVIDIA Parakeet experiment is not a demonstrated Live feature.
- For coding dictation, grounding matters more than continuous chat: send source identifiers as text alongside audio (for example `main.py` and `util.py`), or send one image then several seconds of audio rather than resending frames (~1,200 tokens each).
- Adapting to a user happens within the active context window, which differs from durable memory. On evaluation, Thor points to requirements such as HIPAA and SOC 2 plus partner infrastructure. On `store=false`, disabling storage removes server-side continuation and background execution, but is not a universal privacy guarantee. Vertex timing has no date.

## Resources

- [VisionClaw](https://github.com/Intent-Lab/VisionClaw)
- [Gemini API skills](https://github.com/google-gemini/gemini-skills)
- [Interactions API launch](https://blog.google/innovation-and-ai/technology/developers-tools/interactions-api/)
- [Current Interactions API documentation](https://ai.google.dev/gemini-api/docs/interactions-overview)
- [Building with Gemini 3.1 Flash Live](https://blog.google/innovation-and-ai/technology/developers-tools/build-with-gemini-3-1-flash-live/)
- [Gemini Live API overview](https://ai.google.dev/gemini-api/docs/live-api)
- [Gemini Live API examples](https://github.com/google-gemini/gemini-live-api-examples)
