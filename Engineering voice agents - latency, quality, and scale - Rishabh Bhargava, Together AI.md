# Engineering voice agents: latency, quality, and scale

| Field | Details |
| --- | --- |
| **Speaker** | Rishabh Bhargava |
| **Affiliation** | Together AI |
| **Type** | Expo session |
| **Date** | 2026-05-31 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=N7b1PJc7SFc) |
| **Tags** | `infrastructure-and-deployment` `safety-and-governance` `speech-and-audio` |

## Summary

Rishabh Bhargava leads the voice AI team at Together AI. In this session he explains what it takes to run a voice agent — a system that handles a phone call in real time — and treats latency as a first-class part of the product requirements rather than an afterthought.

He walks through the **cascading pipeline**, today's dominant design: audio comes in, speech-to-text turns it into words, an LLM decides what to do and may call tools, text-to-speech turns the reply back into audio, and chunks stream back to the caller. Each component has its own key metric — word error rate and turn detection for recognition, time to first token (TTFT) for the LLM, time to first audio and real-time factor (RTF) for speech synthesis.

The engineering challenge is an "AND" problem: the agent must be fast, smart, natural-sounding, and reliable at the same time. Bhargava shows how model choices fit inside a latency budget, why colocating models can cut a real chunk of delay, and how scaling rules differ from ordinary async services because calls hold long-lived, stateful connections.

He closes with the emerging alternative — a single speech-to-speech model — plus audience questions on tool-call evaluations, where guardrails belong, and how to keep observability when the transcript disappears from the main path.

## Key Takeaways

- **Latency is a product requirement.** Bhargava's heuristics: humans reply in ~300 ms, AI above 500 ms feels noticeable, and 1–2 second pauses make callers hang up.
- **Voice is an AND problem.** Fast but incapable, or capable but fragile under load, still fails the caller.
- **The cascading pipeline dominates.** Speech-to-text → LLM (with tools) → text-to-speech, coordinated by an orchestrator like Pipecat, LiveKit, or a homegrown system.
- **Recognize the end of a turn, not just the words.** Turn detection is still partly unsolved; starting too early turns a decision into an interruption.
- **Streaming beats batch.** Whisper-style 30-second chunks need wrappers to fake streaming; native streaming encoders reuse cached activations.
- **Budget the whole conversation.** LLM takes most of the latency and cost, then text-to-speech, then speech-to-text.
- **Network time counts.** Colocating models cut ~75 ms of network latency to ~5 ms in his example, about a 30% overall reduction.
- **Check before speaking.** A guardrail must run before text-to-speech, because spoken offers cannot be taken back.

## Detailed Notes

### Why voice matters · 00:16
- Bhargava introduces himself as Together AI's voice AI lead; before Together he co-founded and led Refuel, which Together acquired.
- Together provides training compute and inference infrastructure; he cites a million-plus developers and customers including Cursor and Decagon.
- Billions of calls a year are still handled by humans — checking an order, changing a reservation, booking a doctor's appointment.
- Voice is also a new interface to computers: people talk before they read, and that habit is extending to tools like Cursor and Claude Code.
- He frames rich, dependable conversations as an engineering problem in 2026, not a research one.

### Four requirements at once · 02:42
- **Fast:** real-time response, with the latency heuristics above.
- **Smart:** handle complicated instructions, resolve ambiguity, and call tools that reach the real world.
- **Natural:** the right language and accent, correct name pronunciation, and appropriate emotion.
- **Reliable:** the same experience at 100, 1,000, or 10,000 concurrent calls.
- These must all hold together; solving one alone does not produce a usable agent.

### The cascading pipeline · 04:20
- An orchestrator (Pipecat, LiveKit, or homegrown) coordinates the streaming path.
- Steps: receive audio chunks → speech-to-text → LLM decides tools and response → text-to-speech → stream audio back.
- Each component is separate, but the caller experiences their combined behavior as one conversation.

### Listening: recognition, turn detection, streaming · 05:06
- **Word error rate (WER)** compares the transcript against a reference; state-of-the-art models sit near 6% on open benchmarks.
- Aggregate accuracy is not enough: a wrong customer or drug name passes through the LLM and TTS unchanged, and can't be recovered downstream.
- **Time to complete a transcript:** after the caller stops, how long until the text is ready. Bhargava reports ~100 ms P90 for some models on Together (conditions not specified).
- **Turn detection** decides whether a pause means the caller finished or is still thinking. It remains partly unsolved.
- Architecture is moving from batch to streaming. Whisper was trained on 30-second clips, so wrappers chunk, pad, call, and stitch.
- A newer NVIDIA encoder uses short look-ahead (~80 ms to ~1 s) and caches activations, so heavy work happens once.

### The LLM and TTS inside the budget · 08:27
- For the LLM, the key streaming metric is **time to first token (TTFT)**; his rough target is 200–300 ms, which maps to models in the 8–30 billion parameter range.
- Trade-off both ways: a bigger model may eat the budget before speech starts; a smaller one may fail at instructions and tool calls.
- For text-to-speech, **time to first audio (TTFA)** is the startup delay, and **real-time factor** is processing time divided by generated audio duration.
- RTF below 1 means generation outpaces playback, avoiding buffering (e.g., 5 s of processing for 10 s of audio gives RTF 0.5).
- Quality still needs listening: hear samples for the specific voices you'll use, and check pronunciation and emotional control.

### Network placement and scaling · 10:57
- Rough budget ordering, for both latency and cost: LLM first, then text-to-speech, then speech-to-text.
- Engine metrics exclude network time between the orchestrator and models in different data centers.
- **Scale up early and aggressively** — queueing that's fine for a background job is dead air on a call.
- **Scale down carefully** — long-lived, stateful connections mean you should drain conversations instead of killing pods.
- Deploy near callers to cut audio travel time, and support required regions for data-residency rules such as European ones.
- Colocation example: 100–200 ms engine times plus 75 ms network becomes ~5 ms network, about a 30% total latency drop (his illustrative setup, not a general guarantee).

### Speech-to-speech and full duplex · 14:24
- A single **speech-to-speech model** replaces the STT → LLM → TTS chain, reducing orchestration boundaries.
- Examples: the OpenAI Realtime API and NVIDIA's Nemotron 3 VoiceChat (early access at the time).
- Adoption in production is limited, in his account, by weaker instruction-following and tool-calling; teams often return to a pipeline.
- The upside is preserving nuance — tone, emotion, hesitation — instead of losing it in a transcript.
- **Full duplex** lets the model generate audio while still receiving it, enabling backchannels ("I see") and easier handling of interruptions and barge-ins.

### Guardrails, evals, and auditability · 16:56
- For tool-call evals, separate **structural validity** (parsable, correct shape) from **semantic correctness** (right tool, right arguments).
- Recommend tool-call structure very close to 100%; the semantic threshold depends on the use case.
- When a small model is fast but unreliable, fine-tuning on use-case data can lift tool calling without breaking the latency budget.
- Guardrails can sit before the LLM (routing) or after generation (output checks); each adds a component needing its own latency allocation, SLA, and scaling.
- The key constraint: prevent bad offers **before text-to-speech runs**, since spoken claims cannot be revoked.
- The **thinker–talker pattern** splits work: a small model gives a quick acknowledgment and delegates heavy work to a larger model.
- Observability is not lost with speech-to-speech: you can transcribe alongside audio, and evals shift to longer full-duplex conversations.

## Resources

- [Pipecat](https://github.com/pipecat-ai/pipecat) — Python framework for assembling streaming voice agents.
- [LiveKit Agents](https://docs.livekit.io/agents/) — docs for building realtime agents in Python or Node.js.
- [OpenAI Whisper](https://github.com/openai/whisper) — speech-recognition code and weights.
- [OpenAI Realtime API guide](https://developers.openai.com/api/docs/guides/realtime) — realtime audio applications.
- [NVIDIA Nemotron 3 VoiceChat](https://developer.nvidia.com/nemotron-voicechat-early-access) — a 12B full-duplex speech-to-speech model.
- [Together AI's acquisition of Refuel.ai](https://www.together.ai/blog/together-ai-acquires-refuel-ai) — background on the speaker's previous company.
