# Beyond Transcription: Building Voice AI That Understands Conversations

| Field | Details |
| --- | --- |
| **Speaker** | Hervé Bredin |
| **Affiliation** | pyannoteAI |
| **Type** | Track keynote |
| **Date** | 2026-06-05 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=mFLlVpnGpds) |
| **Tags** | `speech-and-audio` |

## Summary

Hervé Bredin, chief science officer and co-founder of pyannoteAI, argues that speech-to-text alone misses much of what matters in a conversation. Transcription answers what was said, but not who said it — and a single flat stream of words hides interruptions, backchannels (short acknowledgments like "mm-hmm" made while someone else is still talking), pauses, and delivery. He walks up a ladder: from what was said, to who said what, to who said what and when and how.

The core technical topic is **speaker diarization**: tracking which speaker is active in a recording. It is built from three steps — voice activity detection, segmentation (finding speaker changes and overlaps), and speaker assignment. It is harder than it sounds because the number of speakers is unknown, labels are anonymous and interchangeable (speaker one, speaker two), turns can be very short or overlap, and speaking time is often uneven. Bredin demonstrates the open-source Community-1 model on a 30-second phone call in a Python notebook, explains diarization error rate (DER), and shows the premium Precision-2 model improving the result.

The second half explains why turning a transcript plus diarization into speaker-attributed text is not a simple timestamp join. Speech-to-text models are mostly trained on single-speaker audio and degrade on meetings — the same NVIDIA Parakeet model reports 11.4% word error rate on the AMI meeting dataset with headset microphones, while his team measures 26% with the shared table microphone. Timestamps disagree, coverage disagrees, and overlap is hard. He closes with a demo of pyannoteAI's cloud orchestration, which correctly interleaves an overlapping exchange.

## Key Takeaways

- **Transcription answers "what", not "who".** Speaker-attributed transcription adds the missing tags; pairing Whisper with pyannote made that combination popular.
- **Diarization means "who spoke when".** Voice activity detection, then segmentation of turns and overlaps, then consistent speaker labels.
- **Labels are anonymous and interchangeable.** What matters is consistent grouping, not whether a speaker is called one or two.
- **DER is measured in time, not words.** Speaker confusion + false alarm + missed speech, divided by reference speech duration: about 5% for Community-1 and 3% for Precision-2 on the demo call.
- **No single number describes diarization.** Roughly 8% DER for two-person phone calls, but about 41% for a noisy restaurant conversation among friends.
- **Benchmark conditions matter.** The same model on the same AMI dataset reports 11.4% WER on headset microphones and 26% on a shared table microphone.
- **Joining transcript and speakers is not trivial.** Boundaries disagree, gaps leave words unattached, and overlap offers two candidate speakers for one recognized word.
- **Exclusive diarization is one disclosed trick.** During overlap it selects the speaker most likely to be transcribed, simplifying reconciliation — and the orchestration is designed to wrap any STT model, including privately fine-tuned ones.

## Detailed Notes

### Transcription tells you what was said, not who said it · 00:16
- Bredin spent his career in academia before co-founding pyannoteAI. The open-source pyannote.audio toolkit focuses on speaker diarization.
- When OpenAI released Whisper, users had good transcription but no speaker tags, so they naturally paired Whisper with pyannote. He points to an inflection in the project's GitHub star history around that release.
- His birthday is about a week after the talk and the project is approaching 10,000 stars; he asks the audience for a star as a birthday gift.
- The simple distinction: STT (speech-to-text) maps recorded or streaming audio to a sequence of words. It does not tell you where one participant's contribution ends and another's begins.

### From "what was said" to "who said what" · 02:29
- **Speaker-attributed transcription** attaches speaker tags to the words, changing the question from what was said to who said what.
- For a meeting notetaker, that is often enough to summarize the discussion and to assign an action item to the person who actually accepted it. A correct sentence attached to the wrong participant produces an incorrect record.
- Applications named: video translation and dubbing need a consistent voice per participant; meeting notes need commitments attached to the right people; podcast intelligence needs to connect speech to a recurring person across episodes or shows.
- These applications ask for different amounts of speaker information. Keeping labels consistent inside one recording is narrower than recognizing a person across recordings.

### Timing, delivery, and who is being addressed · 04:05
- Even correct attribution leaves information out. Precise timing reveals that one speaker interrupts another, and it reveals backchannels — short acknowledgments such as "mm-hmm" while someone else keeps talking, which can signal agreement without taking over the turn. Missing one removes a small but consequential contribution.
- Pauses matter too: a gap between turns can change how a response is interpreted, even though it adds no words to the transcript.
- The next layer is delivery: how something was said. A laugh might answer a joke or an awkward moment; a cough might accompany discomfort. Bredin presents these as possible interpretations, not reliable labels for someone's state of mind.
- Prosody (patterns of stress, rhythm and intonation) and disfluencies add more. His example is "The dog ate the cake" — the same words carry different implications depending on emphasis, and keeping that detail gives downstream language models more to work with.
- A step further back: who is talking to whom, and where. Addressing an audience differs from answering one person during Q&A, and a quiet room differs from a street. He narrows the technical talk to diarization — who speaks when does not by itself solve addressee detection, prosody or intent.
- As evidence of demand, he shows a Hugging Face audio-model ranking sorted by downloads: in that snapshot three of the top seven models involve speaker identity or diarization, and the rest are speech-to-text.

### How diarization works, and why it is hard · 08:42
- Three increasingly specific tasks. **Voice activity detection**: find the regions where anyone is speaking. **Segmentation**: locate speaker changes inside those regions and find overlapping turns, including interruptions and backchannels. **Speaker assignment**: give turns consistent speaker labels throughout the recording.
- Segmentation detail matters: a very short "yes" may carry the most important information in the conversation, and detecting a long speech region while losing that small turn is not enough. Finding turns is still not diarization — the system must also establish which turns belong to the same speaker.
- The number of speakers is generally unknown in advance. A meeting attendee list gives guidance, not an inventory: two people may join from the same channel, or someone who was not invited may join.
- Labels are anonymous — speaker one, speaker two, speaker three, not names. They are interchangeable as long as their assignment stays consistent. Swapping every green label to black and vice versa leaves the diarization equivalent.
- Other difficulties: overlapping speech, extremely short turns, uneven speaking time (a participant who only says "mm-hmm" gives the system much less to work with than the discussion leader), plus noise and the usual acoustic problems of speech processing.

### Evaluating diarization: errors, DER, and what accuracy depends on · 12:27
- The demo input is a roughly thirty-second telephone conversation between two people: they exchange greetings, name their locations (New Jersey and Texas), and discover that both are originally from Chicago. Bredin presents a manually labeled reference first.
- Locally, he downloads the open-source **Community-1** model from Hugging Face, runs it with PyTorch's `MPS` backend on his Mac, and compares the prediction with the reference annotation on aligned timelines.
- Errors separate into three kinds: **speaker confusion** (speech assigned to the wrong speaker), **false alarm** (speech detected where the reference has none), and **missed speech** (reference speech not detected — which includes detecting only one of two speakers during overlap).
- The last case is easy to overlook: the system can correctly detect that someone is speaking and still miss part of the conversation because a second person talks at the same time.
- **Diarization error rate (DER)** combines the three durations and divides by the reference speech duration. The quantities are durations, not word counts; the scoring convention counts speaker-time, so one second containing two reference speakers contributes two speaker-seconds when overlap is scored.
- Bredin reports about 5% DER for Community-1 on this call. The premium cloud model Precision-2 fixes a speaker confusion and reduces the other errors, for about 3% DER. These are results from the short demo recording, not dataset-wide scores, and comparisons also depend on whether overlap is included and on the boundary collar (a tolerance around reference boundaries).
- Asked how well the state of the art works today, Bredin says it depends entirely on the use case. He reports about 8% DER for conversational telephone speech between two people, but about 41% for a noisy restaurant conversation among several friends — even for the best system.
- These scenario numbers are separate comparisons, not additional scores for the two models in the notebook. The number of participants and the acoustic environment change the difficulty substantially; noisy group conversation remains far from solved.

### Why a leaderboard score may not transfer to a meeting · 16:52
- Producing speaker-attributed text looks like assigning a speaker to each word. The first obstacle is that recognition itself may already be wrong: most STT models are trained on single-speaker data and generalize poorly to multi-speaker recordings with overlap, speaker changes and crosstalk.
- His example uses NVIDIA Parakeet on the AMI meeting dataset. The Open ASR leaderboard reports 11.4% WER (word error rate) using individual headset microphones — effectively single-speaker speech. His team measures 26% using the distant microphone in the middle of the meeting table.
- The dataset name is the same; the input conditions are not. Headset recordings and a shared table microphone are materially different recognition problems, and the comparison changes both microphone placement and conversational conditions — it does not isolate the effect of overlap alone.
- The difficult conditions include distant speech, speaker changes, crosstalk, interruptions, and code switching (changing languages mid-sentence). The practical rule for a meeting product: test the kind of recording the product will actually receive. A score from isolated headset speech does not establish performance on a shared room microphone.

### Speaker attribution is not a simple timestamp join · 19:07
- The second obstacle is **reconciliation**: making transcription and diarization agree about which words belong to which speakers.
- Their timestamps can disagree, and their coverage can disagree — diarization may detect speech that STT never transcribes, or STT may produce words where diarization reports no speech. During overlap, two active speakers may be represented by only one recognized word stream. A timestamp join cannot assign words that recognition never produced.
- In the second demo, Bredin applies Parakeet to the same phone call. It returns a sequence of words, each with start and end times — the word intervals needed for attribution.
- He places those words beside Precision-2's speaker turns and examines the assignments one by one. Easy case: a word interval shifted slightly left still clearly matches the yellow speaker.
- Hard cases: the word "oh" falls between two diarized turns, so it has no unambiguous turn to attach to; the word "okay" coincides with two active speakers, but transcription supplies only one word.
- These are different failure modes — imperfect boundaries, a gap between turns, and competing speakers during overlap. Treating all of them as the same interval-matching problem hides the uncertainty that reconciliation must resolve.

### Preserving the overlapping exchange, and the tools behind it · 21:26
- Bredin submits a job to pyannoteAI's cloud service that combines Precision-2 diarization, Parakeet transcription and reconciliation. The output contains words already assigned to speakers, so the application does not have to resolve each conflicting interval itself.
- The revealing example is the transition between the two introductions: one participant's trailing "um" overlaps the other's opening words. The output interleaves words from both speakers, keeping the filler as the other person begins. He reports the system gets this overlap right — a specific demonstrated success, not an explanation of the full algorithm.
- The talk tutorial lives in the `pyannoteAI/tutorials` repository. Its components have distinct roles: `pyannote.audio` for open-source diarization, `pyannote.metrics` for evaluation, and `iPyannote` for the interactive visualization widget (the "i" stands for interactive). The SDK provides access to the premium models, complementing the local open-source workflow.
- Asked where conflict resolution happens, Bredin says the complete technique is proprietary but identifies one disclosed part available in Community-1: **exclusive diarization**.
- During overlapping speech, exclusive diarization selects the speaker considered most likely to be transcribed by the STT model. That removes a competing speaker assignment from that view and makes reconciliation simpler. It is one major part of the approach, not a complete explanation of the interleaving demo.
- The intended boundary is that orchestration should work without modifying the STT model, supporting different transcription systems — including privately fine-tuned models. That lets a team keep a recognizer adapted to its own use case while adding diarization and reconciliation around it.

## Resources

- [Beyond transcription talk notebook](https://github.com/pyannoteai/tutorials/tree/main/2026-04-beyond-transcription-talk) — the exact tutorial for both live demos.
- [Community-1 speaker diarization](https://huggingface.co/pyannote/speaker-diarization-community-1) — model card for the open-source local pipeline.
- [pyannote.metrics](https://pyannote.github.io/pyannote-metrics/reference.html) — DER components, speaker-label matching and evaluation settings.
- [Parakeet TDT 0.6B v3](https://huggingface.co/nvidia/parakeet-tdt-0.6b-v3) — the exact transcription model used in the second demo.
- [pyannote.audio](https://github.com/pyannote/pyannote-audio) — open-source toolkit for voice activity, segmentation and diarization.