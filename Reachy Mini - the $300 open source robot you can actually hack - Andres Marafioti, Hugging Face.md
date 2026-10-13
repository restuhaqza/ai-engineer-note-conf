# Reachy Mini: the $300 open source robot you can actually hack

| Field | Details |
| --- | --- |
| **Speaker** | Andres Marafioti |
| **Affiliation** | Hugging Face |
| **Type** | Talk |
| **Date** | 2026-05-29 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=0jeZfjJMfmo) |
| **Tags** | `robotics-and-world-models` `speech-and-audio` |

## Summary

Andres Marafioti, who leads multimodal research at Hugging Face, introduces Reachy Mini — a small, expressive, open-source robot aimed at hackers, researchers, and students. His starting point is a gap: robotics is advancing quickly, but capable robots cost roughly $50,000 and up, which keeps them out of classrooms and garages. He also argues that copying the human body needlessly narrows design, and that a non-human, expressive form invites genuinely new kinds of interaction.

Voice AI, by contrast, is already mature. There are strong commercial and open-source speech models, so voice is a natural interface for embodiment. Marafioti demonstrates a conversation app where the robot takes photos, describes what it sees, shows emotions, and tells jokes — all through natural speech.

The technical heart of the talk is serving that conversation. He lays out a three-layer system: a speech-to-speech pipeline, the robot application, and a scaled inference backend. He then walks through optimizing Qwen3-TTS with Codex: streaming audio, a static KV cache, and CUDA graph capture lifted generation from below real time to about 5.8× real time, with time to first audio dropping to a few milliseconds.

He closes with a warning that latency is end-to-end: once synthesis is fast, infrastructure overhead can become the dominant part of the user's wait.

## Key Takeaways

- **Access is the bottleneck.** Capable robots cost about $50k and up, so few students or hobbyists can experiment with them.
- **Form shouldn't just copy biology.** A humanoid shape sets expectations but constrains hardware; an expressive non-human robot invites new interactions.
- **Voice is ready.** Good commercial and open-source speech models exist, so voice is the natural way to talk to an embodied agent.
- **Two configs.** A $300 version (no onboard Raspberry Pi, no battery) and a $450 self-contained version with both.
- **Ship it unassembled.** Building the kit teaches how it fits together and makes repairs a simple part-swap.
- **Three-layer serving.** Speech-to-speech pipeline, robot application, and a scaled backend; LLM inference scales separately from conversation workers.
- **Optimization mattered.** Streaming, a static KV cache, and CUDA graph capture took Qwen3-TTS from below real time to about 5.8× real time.
- **Latency is end-to-end.** The model's time to first audio is only part of the wait — serving infrastructure can cost as much again.

## Detailed Notes

### Who gets to experiment with robots? · 00:20
- Marafioti recalls a Zurich robotics-club humanoid boxing match — impressive and a little disturbing — plus home robots pitched to water plants or scroll TikTok.
- These robots cost roughly $50,000 and up; he puts Waymo vehicles in the mid six figures.
- Humanoid shapes look familiar, which helps set expectations, but borrowing biology constrains hardware that could be faster, more agile, or more stable (he offers a spider form as one example).
- Practical test: a high school won't buy ten $50k humanoids, enterprise machines are hard to adapt, and they don't invite connection.
- His thesis: robotics capability is outrunning access to robotics experimentation.

### Voice is ready; the interaction is open · 03:21
- Voice software is mature: commercial options such as Gradium and GPT Realtime, plus open models like Voxtral and Kokoro (an 80M-parameter model that sounds good).
- Hugging Face's speech-to-speech pipeline provides the pieces to assemble a voice agent yourself.
- Having the pieces doesn't settle how people should talk to robots; Marafioti wants that shaped by broad experimentation, more like computer hacker culture than a single company.
- Reachy Mini targets hackers, researchers, students, and dreamers, and its non-human, expressive form is deliberate — it invites new interactions instead of acting as a person replacement.

### A kit that invites modification · 05:28
- Reachy Mini ships unassembled, so owners learn how the parts fit together; that turns replacing a broken component into a familiar "order the part and swap it" job.
- Two configurations at the historical talk prices: a $300 version (no onboard Raspberry Pi, no battery) and a $450 self-contained version (with Raspberry Pi and battery).
- The cheaper one suits people who don't need onboard compute or a battery and is sold in bulk to high schools and universities.
- Community mods show the idea: a 3D-printed Halloween pumpkin, and the discovery that petting the robot and getting a reaction is enjoyable. Asked whether it purrs, Marafioti says you can make it purr.

### A conversation that reaches into the physical world · 07:34
- A keyboard is an awkward way to address a humanoid, so Hugging Face provides the Reachy Mini Conversation App for voice interaction.
- In a prerecorded demo the robot runs a lobster-themed personality: "functioning within acceptable parameters" but hoping for a good lobster joke.
- Asked to take a photo and describe the scene, it reports a person taking a mirror selfie in a white T-shirt with a yellow emoji and orange pants — then jokes about the pants.
- One conversation blends personality, a camera request, a visual description, and an emotion request, all by voice.
- Marafioti invites the audience to talk to the physical robot after the recording.

### Splitting the robot, speech pipeline, and inference · 09:58
- Everything (models, agents) is open source, but running it locally needs a GPU many owners lack, so the default service runs on Hugging Face Inference Endpoints.
- Marafioti reports 7,500 robots shipped, with conversation their most-used app.
- Three layers:
  - the **speech-to-speech** pipeline (maintained for about two years);
  - the **robot application** (microphone, speaker, echo cancellation, tool dispatch, emotions, camera, face tracking);
  - the **deployment layer** (load-balanced workers plus separate LLM endpoints).
- Turn path: voice activity detection → Parakeet speech-to-text every 150 ms (partial transcripts let the robot react early) → LLM for the reply and tool calls → Qwen3-TTS for audio.
- Echo cancellation keeps the robot from hearing its own speech as input. Owners can replace the pipeline, and the LLM (they use Qwen 3.5 27B) scales separately, because worker connection counts don't reflect real LLM load — eight chatty users differ hugely from eight silent ones.

### Making Qwen3-TTS fast enough to converse · 12:49
- Qwen3-TTS had the speech quality he wanted, but the released implementation was slower than the paper implied; the paper's numbers came from an internal serving system with compilation and CUDA graph optimizations.
- He spent roughly two weeks working with Codex to improve it.
- Obstacle 1 — output delivery: it generated the whole utterance before returning any audio, so a ten-second clip meant waiting ten seconds. Streaming lets playback start while generation continues.
- Obstacle 2 — repeated CPU–GPU coordination: about 500 steps per audio packet, each needing host involvement.
- Fix: replace the dynamic KV cache (which changed shape as input grew) with a static KV cache that allocates more memory upfront. That enables CUDA graph capture and faster generation.
- Result: generation rose from 0.8× to 5.8× real time (roughly 1,250 ms down to about 172 ms per audio second), and time to first audio fell from several seconds to a few milliseconds. He does not specify hardware or workload.

### The first audio packet isn't the whole budget · 15:19
- A second demo shows a Hugging Face Space cloning selected voices and synthesizing pasted text in real time; one sample gives the lobster a new voice.
- The demo narration claims first-token latency under 200 ms and 4× real-time generation — separate claims from the spoken figures, with no shared measurement conditions.
- The open-source implementation is FasterQwen3TTS, usable standalone or through Reachy Mini's voice agent; its repo also separates throughput from relative speedup.
- Key point: the model's time to first audio is only part of the wait. Requests and audio also pass through serving infrastructure, and Marafioti says infra overhead takes about as long as model processing in this deployment.
- A responsive voice agent therefore needs an end-to-end latency budget, not just a fast model number.

### Extending the robot: apps, hardware, and limits · 17:16
- The supplied conversation app is a starting point; he wants people to invent their own interactions and keep that work open rather than gated behind expensive robots.
- The movement app on the onstage robot was one-shotted by giving Codex the robot repo plus a description — his point that even non-coders can begin experimenting.
- Q&A on where apps run: GPU-free apps can run on the Raspberry Pi-equipped robot (within its limits), or a laptop can supply compute. He mentions Java and Python tentatively.
- Hardware extensibility needs hacking. Hugging Face tries to make its open robots stack: SO-100 and SO-101 arms and a three-wheel base called Kiwi designed to accept Reachy Mini. He hasn't seen the mobile combination implemented, but says the parts were designed to fit.
- On Qwen3-TTS gaps: the original was functionally complete but slow; users report cases his optimization didn't cover, so preserving the model's full range while keeping the common path fast is ongoing engineering work.

## Resources

- [Faster Qwen3-TTS](https://github.com/andimarafioti/faster-qwen3-tts) — streaming TTS implementation with CUDA graph acceleration (repository).
- [Qwen3-TTS Technical Report](https://arxiv.org/html/2601.15621v1) — architecture and evaluation, including serving conditions (paper).
- [Hugging Face speech-to-speech](https://github.com/huggingface/speech-to-speech) — configurable voice-agent pipeline (repository).
- [Reachy Mini Conversation App](https://github.com/pollen-robotics/reachy_mini_conversation_app/blob/main/README.md) — conversation software connecting Reachy Mini to voice backends (repository).
- Reachy Mini hardware overview — official kit, variants, and programming support (article).
