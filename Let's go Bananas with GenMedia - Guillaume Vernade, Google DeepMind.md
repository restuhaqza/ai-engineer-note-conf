# Let's go Bananas with GenMedia

| Field | Details |
| --- | --- |
| **Speaker** | Guillaume Vernade |
| **Affiliation** | Google DeepMind |
| **Type** | Workshop |
| **Date** | 2026-05-18 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=BcWFc3H7Khg) |
| **Tags** | `creative-and-generative-media` `vision-and-video` |

## Summary

This hands-on workshop builds a generative-media pipeline around a public-domain
book — Kenneth Grahame's *The Wind in the Willows*. Gemini reads the book and writes
prompts; image, video, music, and speech models turn those prompts into character
portraits, chapter illustrations, clips, songs, and narrated dialogue.

Google DeepMind developer advocate Guillaume Vernade walks through a runnable
notebook, showing how context, reference images, and modality-specific prompts shape
the results. A recurring lesson: more specific prompts produce better media, and the
same models behave differently depending on how you feed them context — for example,
chat history versus an explicit set of reference images.

He also stresses the practical side. The media models are paid, so bound the work,
and regional availability plus safety rules constrain what you can actually build.

## Key Takeaways

- **Changing an image model should mean changing a model name**, not the API — a long-running complaint Vernade raises.
- **Pick a model per output:** Nano Banana 2 for images, Veo 3.1 Lite for cheap video iteration, Lyria for music, Lyria RealTime for live steering.
- **Bound the workload first.** A whole book expands into many paid requests; the notebook caps characters and chapters and asks for payment confirmation.
- **Ask for structured output.** A schema with `name` and `prompt` (later plus a character list) lets downstream code consume fields instead of parsing prose.
- **Prefer explicit references over chat history.** Sending only the relevant character images is more precise and avoids re-sending the book every turn.
- **A still-image prompt is not a video script.** Give Veo an image as the first frame, but describe the next few seconds separately.
- **For Lyria, almost everything lives in the prompt** — instruments, scale, BPM, structure, and lyrics; you can also feed an image as musical input.
- **Two TTS voices can sound like a bigger cast** by writing per-character delivery directions into the transcript.

## Detailed Notes

### Background and the GenMedia ambition · 00:34
- An image app should swap models by changing a name; but **Imagen** and **Nano Banana** exposed different APIs.
- Vernade is a developer advocate at DeepMind: docs, code samples, demos, skills, and prompt guides, plus carrying developer feedback back to the model teams.
- His background is video-game production, then Stadia, then DeepMind; he jokes that Imagen's brand disappearing may have settled the API argument by default.
- Images, sound, and video are everywhere; a **world model** should ingest many modalities (including sensor data) and answer through many.
- The ambition is one encompassing model; shipping separate image and video models creates smaller release boundaries and reduces the risk of breaking one capability while improving another.
- Capability and exposed behavior can diverge: he recalls image understanding being suppressed in an early Gemini release and residual refusal behavior persisting until Gemini 2.0 (his explanation; Google's Dec 2023 Pro Vision launch already offered image input).
- The portfolio extends to vision-dependent robotics, agents, open models (he corrects the slide to **Gemma 4**), and research systems (**AlphaEvolve**, **AlphaGenome**, **WeatherNext**).
- He reports GenMedia releases more often than monthly and a DeepMind-wide average of one release every five days.

### Choose a model for each kind of output · 07:07
- **Nano Banana 2** adds image sizing and aspect-ratio choices, search grounding, and web-image grounding — useful when knowing what something looks like matters (architecture, animals). Some subjects, such as newer buildings, are restricted.
- **Veo 3.1 Lite** is a cheap way to iterate on video prompts before a larger result or upscaling; he cites about $0.05 per generated second and about $0.40 per video.
- **Lyria** produces completed clips (30 seconds) or full songs (up to three minutes).
- **Lyria RealTime** keeps generating while a session runs and accepts new prompts that steer it. He describes it as predictive rather than prompt-to-finished, with roughly two-second responsiveness (his figure).
- The interaction is like a DJ changing the mix while the music continues.

### Bound the job and set up the notebook · 09:43
- The workshop uses a runnable book-illustration notebook; a full run costs about $1, mostly from video, and skipping video is cheaper.
- Pipeline: a Project Gutenberg book → Gemini writes prompts → media models make portraits, chapter illustrations, videos, music, and narration.
- Setup needs a recent SDK (for the demonstrated music support) and an AI Studio API key; the main path uses paid access, with the older Nano Banana free tier for image-only runs.
- The client loads the key and enables automatic retries (five retries, two-second delay) for overload, especially when US traffic rises.
- Model choices: Gemini 3.1 Flash Image Preview (Nano Banana 2), a text model, a Lyria clip model, and a Pro TTS model.
- **Bound the workload before generating:** the notebook has a payment checkbox (accidentally left on) and caps characters and chapters to control spend and waiting time.
- The book is uploaded with `client.files.upload`.
- Know the access surfaces: consumer Gemini apps (easy, little control), **Vertex AI** (infrastructure, location, access control, more setup), the **Gemini Developer API** (API-key onboarding, you protect the key), and **AI Studio** (interactive experimentation).
- Vertex and the Developer API share an SDK, but the Developer API hides bucket creation and ACL setup; a leaked API key is a direct misuse risk.
- The notebook creates a text chat so later requests keep the book and prior instructions, and requests **structured output** — each prompt has a `name` (character or chapter) and a `prompt` string, expressed as a schema such as `NamedPrompt`.
- Service tiers described: Standard (standard price, shared queue), Flex (50% discount, can wait minutes), and Priority (twice the standard price, preferential scheduling). He says Veo support is unclear and Lyria does not support it yet.

### Give the cast a consistent visual identity · 21:46
- Leaving style unspecified yields similar defaults, so he supplies a colorful building-block style; Colab form annotations make it easy to edit.
- System instructions constrain composition: no book-cover titles and no multi-panel images — stopping the model from making a cover when it knows it is illustrating a book.
- Gemini first describes the main characters. A leftover adult-only restriction can now be removed; generating new images with children differs from editing existing ones (restricted in Europe at the time).
- A separate **image chat** receives the character prompts and sets image modality, aspect ratio, system instructions, style, and Priority.
- Keeping it separate avoids mixing text prompt-writing history with image outputs while retaining earlier images for continuity.
- Mole and Water Rat appear first, then a conspicuously oversized Toad, then Badger and Otter; generated images are appended to an array for reuse.
- Sequential generation lets each turn see the images produced before it; parallel asynchronous calls would lose that ordered dependency.

### From chat history to explicit references · 26:10
- Next: one illustration prompt per chapter, excluding multi-panel compositions and repeating character descriptions.
- The first version relies on chat history to carry appearance forward — but the SDK resends prior messages (including the book) every call, which gets slow.
- The **Interactions API** offers a different mechanism: a response returns an interaction ID that a later request references, so the server recovers prior context; the same interaction can also branch (for example, lyrics → cover image and song).
- His retention answer is tentative; documentation later specifies default retention of 55 days for paid interactions and one day for free, with shorter paid retention configurable.
- **Automatic caching** helps reused context, and ordinary calls can benefit from caching too.
- A more explicit approach expands structured output to include the characters appearing in each chapter, then looks up their saved images and sends only those references with a unary `generate_content` call.
- A dictionary lookup raises an error for a missing character instead of silently substituting an unrelated image.
- The explicit-reference images look broadly similar; in one snowy scene the model seems to show Water Rat attacking Mole, but the prompt says rescue — **preserving appearance does not guarantee the depicted action reads correctly.**

### Video: a still frame is not a script · 34:12
- The final chapter illustration goes to **Veo 3.1** as the first frame; the first attempt reuses the illustration prompt.
- He stresses how much the starting frame determines the video's possibilities. Plain Veo 3.1 is the quality option; Fast and Lite are smaller, faster variants doing less generation work.
- The generated rescue clip has sound and dialogue, but the wrong character speaks — a still-image prompt described the scene without adequately directing action or assigning speech.
- Fix: add a prompt-writing step — give Gemini the chapter image, ask for a Veo prompt describing the next few seconds, then generate from the same first frame.
- The new prompt has Water Rat lower his silver pistols, reassure Mole, and walk away; it keeps the book's old-fashioned tone and the building-block clothing details.
- He judges the second clip better, though it contains no conversation — a more directed continuation, not proof that every desired audiovisual detail is controlled.
- Warning: don't regenerate video endlessly; iteration costs become significant.

### Music: composing related soundtracks with Lyria · 39:46
- Uses Lyria's faster clip model; he cites about $0.04 for a 30-second clip and about $0.08 for a song of up to three minutes.
- Gemini again writes the prompts, asking for instrumental music that shares an identity across chapters while emphasizing what makes each different.
- He attributes Gemini's skill at media prompts partly to training: Gemini-written prompts contribute substantially to GenMedia training data, and there is internal prompt rewriting. Longer, detailed prompts generally work better than one-liners.
- The music controls live largely in the prompt:
  - **Instrumentation and feel** — instruments, playing style, scale, and BPM.
  - **Duration and development** — when the arrangement should change.
  - **Song structure** — intro, verse, chorus, outro; the chorus indicates material meant to recur.
  - **Lyrics** — supply the words or let the model invent them.
- Switching to songs with lyrics, the themes stay similar because chat history encourages reusing earlier descriptions and musical ideas.
- Lyria returns lyrics **with timing**, enabling synchronized text or a karaoke interface; a final passage can leave about 1.2 seconds of instrumental ending.
- Streaming (`generate_content_stream`) returns lyrics before audio, so a title or cover-image job can start early.
- Lyria can take an image as input: a pot-au-feu shopping list plus "an epic quest with opera voices" turns groceries into drama, with bouquet garni as a recurring chorus.

### Voices, constraints, and live steering · 47:53
- TTS trick: NotebookLM-style audio normally has two voices, but delivery instructions can make those two slots suggest more characters.
- Gemini extracts a passage and rewrites it as a play with narrator and character lines and a consistent speaking style each; the narrator gets one voice, and other characters share another.
- Parenthetical directions (long pauses, breathlessness, stutter, whisper, intensity) distinguish them; repeating a delivery across lines gives a stable cue.
- The TTS request must include an explicit reading instruction ("Read this text…"); supplying text alone wasted his time. Narrator → `Sulafat`, characters → `Fenrir`.
- Playback sounds distinguishable, though both speak slowly; a requested accent did not come through. Irish, Singlish, and German accents are directions to explore, not demonstrated results.
- At scale he would keep real character names in the transcript and a baseline prompt per character, adding local emotion per line.
- Availability: a company stuck on Nano Banana 1 because it can only use models hosted in Europe. New previews run only on global endpoints; he expects that rule to persist and is pushing for faster general availability.
- Safety: graphic material may be toned down or refused, which is why he chose children's books.
- He closes with **Lyria RealTime** and the Space DJ app: musical locations represent prompts, and movement steers the mix (post-punk, neo-soul, K-pop, calmer moods). Autopilot moves through the space automatically, and the session ends after about ten minutes — a reminder that continuous music has a lifecycle limit.
- He notes RealTime is not intended for voices but can produce vocalizations, and imagines a game steering music by region, activity, or player state.

## Resources

- [Book illustration with the Gemini Cookbook](https://github.com/google-gemini/cookbook/blob/main/examples/Book_illustration.ipynb) — the workshop notebook (mentioned).
- [Gemini API pricing](https://ai.google.dev/gemini-api/docs/pricing) — image, video, and music pricing (update).
- [Interactions API](https://ai.google.dev/gemini-api/docs/interactions-overview) — server-side conversation state, chaining, and retention (update).
- [Generate music with Lyria 3](https://ai.google.dev/gemini-api/docs/music-generation) — music generation and prompting (update).
- [Gemini Pro and Pro Vision developer launch](https://developers.googleblog.com/its-time-for-developers-and-enterprises-to-build-with-gemini-pro/) — December 2023 image-input announcement (related).
- [Space DJ](https://magenta.withgoogle.com/spacedj-announce) — a spatial interface that continuously steers Lyria RealTime (mentioned).
