# How to talk to statues

| Field | Details |
| --- | --- |
| **Speaker** | Joe Reeve |
| **Affiliation** | ElevenLabs |
| **Type** | Talk |
| **Date** | 2026-06-01 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=u-rJwPPU3QA) |
| **Tags** | `speech-and-audio` `coding-and-developer-tools` `vision-and-video` |

## Summary

Joe Reeve works in the growth team at ElevenLabs, an audio AI company, and he opens by showing a side project that went viral in London: an app that lets you photograph a statue and then talk to it out loud. The talk spends the rest of its time on how the app works and on the design questions it opens up.

The photo pipeline is short and simple. The app identifies the object from the image, uses OpenAI to research its identity and imagine what its voice might have been, generates a matching voice with ElevenLabs' Voice Design API, and wraps the persona in an agent you can speak to. Reeve says the whole loop takes about 30 seconds. He built it in a weekend, and the video plus a follow-up about how fast it was built drew museum groups, auction houses and eventually an internal hackathon.

The interesting part is what the demo exposes. Reeve argues that curators, not web search, should own an object's story and voice. He wants voice paired with a visual interface people can steer and edit, controls that let listeners skim a spoken answer, and a better model for interruption. After a cut-in, he argues, an agent should treat the conversation as what the listener actually heard — not the full text it generated.

He closes on the unglamorous half of a viral demo: making the video is its own craft, and a strong hook, captions and music decide how many people ever see the work.

## Key Takeaways

- **The app is mostly glue.** Reeve combines managed services — image identification, research, voice design and a hosted agent — instead of building models.
- **Voice design is underused.** You can describe a voice in words rather than pick one from a list.
- **Curators own the narrative.** Photo plus web research is a prototype shortcut; museums should decide the content and the voice.
- **Voice and visuals belong together.** One idea: talk to a "product manager" agent while a companion screen shows and lets you edit what was understood.
- **Interrupting is socially awkward.** People are too polite with voice agents; a push-to-talk control supplies a signal that raw audio misses.
- **Spoken answers need navigation.** Listeners want forward/back controls that move between concepts, not between sentences.
- **Heard speech is not the same as generated text.** If a user interrupts mid-reply, trim the unheard remainder using playback timestamps before continuing.
- **Distribution is part of the work.** Short hooks, captions and music decide whether a good demo ever reaches an audience.

## Detailed Notes

### Assembling audio parts into a statue app · 00:36
- Reeve introduces himself as part of ElevenLabs' growth organization and takes a quick show of hands for who knows the company.
- He lists its audio services: text-to-speech, transcription, music generation, sound effects, voice creation and editing, and a managed agents platform.
- He singles out **voice design** — describing a voice in text and generating one — as a capability he thinks people underuse.
- A demo video shows British Museum objects introducing themselves: Pharaoh Amenhotep, Demeter, Hoa Hakananai'a, Hans Sloane, a Guardian Lion and others.

### From a photograph to a conversation · 02:22
- The app runs four steps: take a photo to identify the statue, let OpenAI research its history and imagine a voice, generate that voice with the Voice Design API, then create an agent and start a call.
- Reeve reports roughly 30 seconds from photo to conversation as his end-to-end example.
- A QR code points to a blog tutorial; the updated tutorial uses GPT-5.2 with `web_search_preview` and a browser WebRTC call, not a phone line.
- The durable architecture is the sequence itself: identify, research, design a voice, build an agent.

### A prototype, then museums come calling · 03:22
- He built the prototype in Cursor in about two hours on a Sunday and published the build prompt on the ElevenLabs blog.
- The first video got 50,000 impressions; after he reposted it framed as an example of vibe coding, the next day reached 1.5 million.
- Three museums or museum groups reached out; one CEO said a team of ten had spent a year on something similar. Bonhams and Christie's wanted item-to-person conversations, and the response fed into ElevenHacks.
- On a production question, Reeve separates the layers: the voice and agent APIs are designed to scale, user accounts can come from a third party, and the remaining work is finishing the museum experience. Generated code still has to be checked.

### Curators own the story and the voice · 07:05
- Asked about evals, he says photograph-and-research is not the long-term museum workflow; **curators** should decide the narrative instead of pulling random facts from a search.
- Museums treat their collection databases as core intellectual property, and some expose them through public APIs (he mentions the V&A).
- No dedicated curator interface exists yet — the available route is the ElevenLabs dashboard, editing the system prompt and knowledge files. He thinks text is the better medium for managing information even when the final experience is spoken.
- Choosing a voice is an interpretation. He cites Jago Cooper's question of how an inanimate object should sound — a stone quarried in China, carved in Vietnam, then 200 years in a British museum — and notes the same question now applies to everyday objects such as lifts.

### Voice plus a screen, and who you're talking to · 09:50
- Reeve says today's voice interfaces are too binary: you either speak or you use another interface, and interactive or generative UI beside voice is still rare.
- His coding example: talk to a **product-manager agent** that delegates to a coding agent, so the thing you converse with is not the thing doing the work.
- A companion interface could display what the agent extracted and let the user correct it while still speaking, making the conversation inspectable.
- He notes the social problem that people are too polite to interrupt agents; he now interrupts more aggressively and finds it better, but has no settled way to grant that permission to others.
- On skills: ElevenAgents has no native skills concept, but it loads knowledge files and can make MCP (a standard way for agents to call tools) calls. Voice can be bolted onto an existing coding agent — people have connected ElevenLabs to OpenClaw and Claude Code setups.

### Putting the technology inside the object · 13:19
- Museum engagement is an unsolved design problem even for museum leaders; a tablet beside an exhibit can still feel like technology stuck onto an existing experience.
- He describes commissioning a statue with embedded speaker, phone and microphone so visitors talk directly to the object, and a K6 red phone booth where you pick up the handset to talk to a Sir Michael Caine agent.
- On consumer vibe coding, he thinks it has not gone mainstream: even tools aimed at non-technical users can feel oriented toward building B2B SaaS apps from familiar components.
- At evening vibe-coding events, participants have no software vocabulary, so the LLM entertains ideas an engineer might have steered away from. He wonders what an "Instagram filters" or TikTok moment for creation would look like.

### Distribution primitives and dense output · 17:48
- His template is Facebook Instant Games in Messenger: a small set of social and commercial primitives — user and friends, leaderboards, key-value storage, rewarded and interstitial ads — exposed through JavaScript `async`/`await`.
- He says he bought a Fruit Ninja clone for £15, wired in that API, and woke to 15 million users in Vietnam — with little revenue, which he attributes to Facebook testing in lower-value ad regions.
- The lesson: making something is only half of it; giving it a place to go — friends and sharing — is the other half.
- An audience member points out an asymmetry: speech is a convenient way to express intent, but a poor way to receive dense information. Reeve agrees he wants one voice input with parallel outputs, plus a little speech, and notes voice also supplies companionship and motivation to keep going.

### Navigating and steering spoken answers · 21:39
- Concise text and concise speech feel different; a short spoken answer can come across as abrupt or rude, so simply cutting words is not a fix.
- Reeve proposes **skim listening**: forward and back controls like podcast speed or an old iPod's scrubbing, but moving between concepts or topics rather than sentences.
- A pitfall: telling a reading agent to "go to the next one" may be treated as a brand-new prompt, producing more paragraphs about that point instead of advancing.
- The room suggests showing the answer's structure: topic jumps, a short summary you can expand by holding a button, or higher-level visual sections with speech developing each one. Reeve points to Claude's voice interface showing tappable sections.

### Turn-taking, transcript repair, and demo craft · 23:48
- Audio-only agents lose the visual cues of conversation: you cannot tell that someone is about to interrupt, and you don't know whether a reply will take 10 seconds or a minute.
- Acknowledgements ("uh-huh") and requests to go back are steering signals, not necessarily new turns.
- A developer in the audience uses a Wispr Flow-style **push-to-talk** boundary because automatic interruption is unreliable, and suggests a small indicator — ideally naming the topic — for when the agent wants to contribute.
- For deciding when to speak, Reeve proposes an asynchronous monitor that keeps re-reading the evolving transcript and asks whether there is something useful to add. He is clear this is still to be built.
- **Heard speech and generated text are different states.** Most platforms append an interruption after the whole generated answer even if the user heard only its start; he suggests using playback timestamps to trim the unheard suffix. A code example in the bundle implements that transcript edit.
- On making demos: he applies an 80/20 rule to editing (about 20–25 minutes on a phone), adds captions, front-loads the hook because median viewing time is 6–12 seconds, and treats music as a major factor. His gear was a phone, CapCut and a borrowed DJI lapel mic, and the finished video came from a second filming trip.

## Resources

- [Build the talking-statue app](https://elevenlabs.io/blog/talk-to-a-statue-building-a-multi-modal-elevenagents-powered-app)
- [Voice Design quickstart](https://elevenlabs.io/docs/eleven-api/guides/how-to/voices/voice-design)
- [V&A collection APIs](https://developers.vam.ac.uk/)
- [Eleven Music API launch](https://elevenlabs.io/blog/eleven-music-now-available-in-the-api)
