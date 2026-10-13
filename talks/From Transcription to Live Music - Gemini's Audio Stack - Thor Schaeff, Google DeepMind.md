# From Transcription to Live Music: Gemini's Audio Stack

| Field | Details |
| --- | --- |
| **Speaker** | Thor Schaeff |
| **Affiliation** | Google DeepMind |
| **Type** | Talk |
| **Date** | 2026-06-09 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=Bc6Ojl2XS1w) |
| **Tags** | `speech-and-audio` |

## Summary

Thor Schaeff, who works on developer experience for the Gemini API and Google AI Studio at Google DeepMind, demonstrates Gemini's audio capabilities end to end. He moves from understanding a recording, to generating directed speech, to holding a live camera conversation, and finally to generating a song on request. Each demo is framed around the DeepMind model families behind it.

The first app, EchoScript, pulls far more than a transcript out of one API call: a summary, timestamps, speaker labels, languages, emotion labels, and English translations for non-English speech, all shaped by a response schema. Schaeff is candid that the labels are fallible — the German and French emotion tags vary between runs, and the Japanese line is mistranslated.

He then shows speech generation as direction rather than selection. A small set of about 30 base voices can be steered toward accents and delivery styles, like an Irish pub storyteller or a Singaporean hawker-center pitch. A live conversation demo adds camera input and a persistent system instruction, including the quirk that the requested Irish accent carried into German.

The finale combines the pieces: a Gemini Live "DJ" gathers a song request, clarifies it, and calls a Lyria tool to generate a track.

## Key Takeaways

- **Audio understanding goes beyond text.** Speech carries emotion, pacing, and context, and Gemini is described as handling overlapping speakers and language switches.
- **One request can fill a whole interface.** EchoScript returns summary, timestamps, speakers, languages, translations, and emotions from a single call.
- **Structured output is not accuracy.** A schema shapes the response but does not make every transcription correct.
- **Direction beats a huge voice catalog.** Roughly 30 base voices can be steered by audio profile, scene, and director's notes.
- **Native audio models differ from cascades.** In a cascade, reasoning happens on text; in a native audio model it happens inside the audio model.
- **System instructions can over-apply.** The Irish accent bled into the German poem, so instructions need to be scoped by language.
- **Live video is limited.** At the time of the talk, Live input accepted video frames at up to one frame per second.
- **Music can be a tool call.** The Live Jukebox model handles the spoken request while Lyria generates the song.

## Detailed Notes

### Start with a multilingual recording · 00:40
- Thor Schaeff works on **developer experience** for the **Gemini API** and **Google AI Studio** at Google DeepMind. He says he joined the day before **Gemini 3** launched.
- He records his own test material: his name, an apology for his French, and a Japanese greeting. A previous attempt was ruined when he recorded over it.
- Model families behind the demos: **Gemma 4**, an open model with multimodality (including audio understanding) for devices and the edge; **Veo** (including **Veo 3.1 Lite**), which adds audio generation to video; and **Gemini 3.1 Flash Live**, a full-duplex, sound-to-sound real-time conversational model that ingests text, voice, and vision.
- Gemini 3 is the foundational layer these audio experiences build on. His date references belong to the conference's April 2026 setting.

### The information a plain transcript leaves behind · 03:01
- Speech carries context, emotion, and pacing, not just words. Audio understanding therefore reaches past transcription to reason across languages, dialects, accents, and modalities.
- Schaeff describes Gemini handling **overlapping speech** and **switching between languages**. These are capability claims; the demo shows a narrower multilingual example.
- **EchoScript**, built with Google AI Studio and available in its gallery, uses **Gemini 3 Flash Preview** to analyze a recording and populate a structured interface.

### One recording, several kinds of output · 04:40
- A single model request returns a summary, timestamps, speaker labels, languages, emotions, and English translations for non-English speech. The summary recognizes his name, so the interface can label his speech.
- Results show both richness and error. The German passage gets a **happy** label (earlier attempts said angry). His French apology is rated **neutral** rather than sad. The **Japanese** line misses the intended introduction. The **Mandarin** output says he lives in the United States, which he accepts though nobody in the room verifies the Chinese.
- Emotion labels are **model interpretations of the performance**, not measurements of the speaker's feelings.
- Important caution: a fully populated interface looks equally convincing whether a given transcription is right or wrong.
- The extraction contract he states in the prompt: identify distinct speakers (by name when context allows), return timestamps and language, translate non-English speech into English, assign an emotion from happy/sad/angry/neutral, and put a brief summary at the start.
- He supplies a **response schema**, turning the request into data the UI can consume. Schema conformance and accuracy remain separate concerns.

### Choose a base voice, then direct its performance · 07:24
- Speech generation reverses the direction: understanding what speech sounds like lets the model produce a particular performance.
- Instead of a large TTS catalog filtered by gender, language, and accent, Gemini offers roughly **30 base voices** that can be directed toward accents and delivery styles. The chosen voice is a performer whose delivery can change.
- The **Voice Library** app in the AI Studio gallery structures direction into **audio profile** (who is speaking), **scene** (the setting), **director's notes** (how to deliver), **sample context**, and **transcript** (the words).
- Schaeff requests a high-pitched Irish male voice and uses **Gemini 3 Flash** to build the speech-generation system prompt: a character named Finian, a cozy crowded pub on the coast of County Clare, strong Irish accent.
- The live synthesis fails, so he plays a prepared sample — an animated pub story about an enormous thing and a chaotic, laughing evening — then the same voice's base sample in a standard American delivery.
- A second comparison uses **Sapphire**: the base sample is standard American English, while a Singaporean scene produces hawker-center dialogue about chicken rice and chili, with expressions like "lah" and "confirm plus chop."
- He attributes the adaptation to the model understanding what each scene sounds like.

### A conversation without a text-only middle · 11:55
- **Gemini 3.1 Flash Live** is a native audio, speech-to-speech, real-time multimodal model. A **WebSocket** connection carries real-time text, audio, and video, and it returns audio plus a text transcript.
- Schaeff cautions against over-trusting audio benchmarks and stresses an architectural difference: in a **cascading pipeline**, an LLM reasons over transcribed text and the text response becomes speech; in a **native audio model**, reasoning happens inside the audio model and a transcript is just an output.

| Architecture | Where reasoning happens | Response path |
| --- | --- | --- |
| Cascading pipeline | An LLM receives transcribed text | Text response becomes speech |
| Native audio model | Reasoning is integrated into the audio model | Audio response, with a transcript |

- A transcript can be an output without being the intermediate representation that drives the conversation. He gives no quantitative comparison.
- He tries it in AI Studio's Live interface, which at the time let people test the models without payment or a credit card.

### The camera works—and the accent instruction travels · 13:32
- He sets a system instruction for a friendly Irish accent and turns on his camera, then asks whether the model can see him and what it thinks of his outfit.
- The reply identifies his **Gemini shirt** and backwards hat while keeping the conversational style. Visual input supplies the subject; the instruction shapes the spoken delivery.
- Switching to German to request a poem, the model introduces in English and recites German verse — and the **Irish accent carries into the German**.
- A global instruction stayed active across the language change: asking in German did not narrow the accent instruction. His advice is to scope system instructions so the accent does not apply to every language.
- Screen sharing is another possible visual input. At the time of the talk, Live video input was limited to **one frame per second** — a limit on ingesting frames alongside audio, not on Gemini's general video understanding.

### Building the live connection · 15:09
- Starting points depend on where the connection lives: **Python** examples for a server-to-server connection, **JavaScript** examples for a client-to-server connection.
- He recommends the published **Gemini API coding-agent skills**, which include Live API guidance. His reason is practical: real-time audio is complex, and an agent benefits from instructions specific to those APIs rather than inferring the integration patterns.

### Let the conversation request a song · 15:55
- The final piece is **Lyria 3**, a music-generation model that can produce lyrics. Two variants: **Lyria 3 Clip** for 30-second jingles and **Lyria 3 Pro** for full-length songs. (Lyria RealTime is a separate offering, so a live interface does not imply streaming synthesis.)
- **Live Jukebox** recreates calling a radio station to request a song. Schaeff gives the Gemini Live model a **tool** that calls Lyria, so the conversational model gathers the request while the music model produces the track.
- The DJ persona greets him and asks what vibe he wants. He interrupts to request German techno schlager about the UK startup scene.
- The DJ proposes a pounding techno beat with a Hans Zimmer-style element; Schaeff approves and asks it to proceed.
- Before generating, the DJ asks about lyrics and energy. Schaeff picks **manic energy** and leaves the lyrics open, forming a brief: German techno schlager, UK startup theme, manic energy, surprise lyrics.
- The song plays: German lyrics mention a pitch deck, growth, a Series A goal, and coffee, with a refrain celebrating startup life — completing the handoff from spoken request to clarified brief to generated song and playback.

## Resources

- [Gemini Live API documentation](https://ai.google.dev/gemini-api/docs/live-api) — real-time voice and vision over WebSockets.
- [Directing Gemini speech generation](https://ai.google.dev/gemini-api/docs/speech-generation) — voices and performance prompting.
- [Generate music with Lyria 3](https://ai.google.dev/gemini-api/docs/music-generation) — clips, songs, and lyrics.
- [Gemini API coding-agent skills](https://github.com/google-gemini/gemini-skills) — installable agent guidance, including the Live API.
- [Gemini 3.1 Flash Live model card](https://storage.googleapis.com/deepmind-media/Model-Cards/Gemini-3-1-Flash-Live-Model-Card.pdf)
- [Gemma 4 on-device development](https://developers.googleblog.com/bring-state-of-the-art-agentic-skills-to-the-edge-with-gemma-4/)
