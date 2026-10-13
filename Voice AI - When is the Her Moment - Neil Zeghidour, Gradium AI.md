# Voice AI: When is the "Her" Moment?

| Field | Details |
| --- | --- |
| **Speaker** | Neil Zeghidour |
| **Affiliation** | Gradium AI |
| **Type** | Talk |
| **Date** | 2026-05-09 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=P_RI1kCkRbo) |
| **Tags** | `voice-ai` `full-duplex` `latency` `moshi` `paralinguistic` `conversational-ai` |

## Summary

Neil Zeghidour, CEO of Gradium AI and a key researcher behind Moshi, the first full-duplex voice model, delivers a candid assessment of why the long-promised "Her" moment in voice AI remains elusive. Gradium AI, a spin-off from a non-profit lab funded by philanthropists like Eric Schmidt, Rodolphe Saadé, and Xavier Niel, aims to unlock the full potential of voice AI by providing foundational models for speech-to-text, text-to-speech, and speech-to-speech applications. Zeghidour highlights that despite significant advancements, current voice AI systems, even the most impressive demos from companies like ElevenLabs and Gradium itself, still fall short of the seamless, natural interaction depicted in the 2013 film "Her."

The core challenges are multifaceted, extending beyond mere latency. Zeghidour argues that the architectural limitations of cascaded systems (Speech-to-Text -> LLM -> Text-to-Speech) inherently prevent truly human-like conversation due to high latency, especially with tool calls. Furthermore, even advanced speech-to-speech models are predominantly half-duplex, meaning they cannot handle simultaneous speaking or crucial backchanneling, which is vital for natural human interaction.

While Moshi demonstrated the solvability of full-duplex communication, it lacked the intelligence and utility of modern agents. The speaker emphasizes that achieving the "Her" moment requires not only solving full-duplex and latency but also developing paralinguistic understanding—the ability to interpret non-verbal cues like tone and hesitation—which is a fundamental scientific problem. Finally, the economic reality of high Text-to-Speech costs at scale poses a significant barrier to widespread consumer adoption, leading Gradium AI to develop on-device, CPU-based TTS solutions like Gradion Phonon to address cost and privacy concerns.

## Detailed Notes

### Introduction to Gradium AI and Mission

- **Origin**: Gradium AI is a spin-off from a non-profit lab established two years prior, funded by philanthropists including Eric Schmidt, Rodolphe Saadé, and Xavier Niel. The lab focused on open research in speech, developing models like Moshi (first speech-to-speech model for conversation), speech-to-speech translation, and Pocket TTS (a CPU model).
- **Mission**: To unlock the unrealized potential of voice AI by training and providing core voice models (speech-to-text, text-to-speech, speech-to-speech, transformation, translation) as building blocks for developers creating voice agents and solutions. Gradium AI does not focus on orchestration or specific verticals.
- **Demo**: The talk opened with a voice clone of a famous podcaster (Joel Rogan) created with Gradium AI's technology, showcasing high-fidelity voice cloning from just 10 seconds of audio.

### The "Her" Moment Analogy and Current State

- **"Her" Movie Relevance**: The 2013 movie "Her" is an overused but highly relevant analogy, as its depiction of AI interaction still anticipates the future of voice AI.
- **Current Demos**: Zeghidour showcased recent demos from ElevenLabs and Gradium AI's own Rich Mini integration, demonstrating improved naturalness.
  - **ElevenLabs Demo**: An AI helper for government services, providing information on starting a new business.
  - **Gradium Rich Mini Demo**: An AI gym bro named Logan, adopting a specific personality and voice.
- **Shortcomings**: Despite advancements, these demos still exhibit high latency, lack the ability to handle simultaneous speaking, and are fundamentally "glorified text models with a voice," meaning anything not explicitly in the text cannot be leveraged.

### Challenges of Cascaded Systems

- **Architecture**: The classic cascaded system involves Speech-to-Text (STT) -> Large Language Model (LLM) -> Text-to-Speech (TTS). Gradium AI uses streaming STT, streaming TTS with voice cloning, and semantic VAD (Voice Activity Detection).
- **Inherent Latency**:
  - **Human Conversation Benchmark**: A human conversation requires the entire stack (understanding, producing an answer, pronouncing it) to be around **200 milliseconds**.
  - **TTS Latency**: Gradium's fast TTS alone is still **more than 200 milliseconds**.
  - **Tool Call Latency**: The biggest bottleneck is tool calls or external API interactions, which can introduce latencies between **500 milliseconds and 4 seconds**. Zeghidour notes that fighting for 10-20ms in TTS becomes less relevant when tool calls introduce such significant, unpredictable delays.
- **Solution for Tool Call Latency: Fillers**: To mitigate tool call latency, the LLM can split its response. It sends the tool call and, while waiting for results, generates natural conversational fillers to keep the interaction flowing. Once the results are retrieved, they are inserted naturally back into the conversation. A live demo of a travel agent using this technique was shown.

### Limitations of Speech-to-Speech (STS) Models

- **Definition**: STS models aim to reduce latency by taking speech as input and directly outputting speech, bypassing the intermediate text steps.
- **Half-Duplex Problem**: Almost all existing STS models, including advanced ones like OpenAI's or Cezanne, are **half-duplex**. This means the model is either listening *or* speaking, but never both simultaneously.
- **Impact on Naturalness**: Half-duplex systems cannot handle the ambiguity and overlap of human conversation. This prevents crucial "backchanneling" (e.g., saying "mhm," "uh-huh" to show active listening), which is a sign of politeness and engagement in many cultures (e.g., up to **20% overlap** in Japanese conversations). The speaker demonstrated how annoying and unnatural a half-duplex interaction becomes when attempting backchanneling.

### Full-Duplex with Moshi

- **Moshi's Breakthrough**: Moshi, developed by Gradium AI, was the first full-duplex system, demonstrated almost two years prior.
- **Capabilities**: Moshi allows for constant overlap between the user and the AI. Even if the model anticipates and starts answering before the user finishes, the user can talk over it, and the model still processes and considers the full input. This makes it highly robust to noise and multiple speakers, creating a very natural conversational flow. Nvidia's Personal Plex model is based on Moshi.
- **Moshi's Limitations**: Despite its full-duplex capability, Moshi was "very stupid" and "useless" as an agent. It lacked tool-calling abilities, observability, and true paralinguistic understanding, making it impractical for production use.

### Paralinguistic Understanding

- **Definition**: This refers to the AI's ability to understand the non-verbal cues embedded in speech, such as tone, hesitation, discomfort, and cultural signals.
- **"Her" Example**: In the movie, the AI understands the character's discomfort from his tone.
- **Current Gap**: In cascaded systems, this crucial information is entirely stripped when speech is transcribed to text. Even in STS models where the audio information is technically present, it's often not exploited if the model isn't specifically trained to capture and leverage these cues (e.g., if trained only on factual Q&A datasets).
- **Nature of the Problem**: Zeghidour stresses that this is a **"science problem, not a prompt engineering problem,"** requiring fundamental research to solve.

### Scalability and Cost Economics

- **High Cost of Voice AI**: Voice processing, particularly Text-to-Speech, is very expensive. Most hyperscalers run their voice models at a loss, using them as a marketing tool.
- **Cost Breakdown**: While LLM costs are becoming "almost nothing," and STT and diarization are affordable, TTS is the primary cost driver.
- **Real-World Impact**: Companies developing consumer voice apps often "burn their fundraising in TTS bills" before they can even grow a user base, making profitability challenging.
- **Privacy Concerns**: As AI becomes more integrated and handles private data, users desire more control and privacy, preferring local processing over cloud-based solutions susceptible to hacks.

### Gradion Phonon: On-Device TTS Solution

- **Goal**: To enable the creation of scalable consumer voice applications without prohibitive API fees.
- **Definition of On-Device**: For Gradium AI, "on-device" means running efficiently on a smartphone CPU, not just a gamer GPU.
- **Specifications**: Gradion Phonon is a very small model, less than **100 million parameters**, yet offers high-fidelity neural speech. It outperforms existing on-device models like Kokoro (which lacks voice cloning).
- **Benefits**: Eliminates API fees, provides local processing, ensures total privacy, and offers smooth, quiet performance without reliance on servers or cloud latency.
- **Availability**: Gradium AI has opened a private beta for Gradion Phonon.

## Key Technical Concepts

- **Cascaded Systems**: A common architecture for voice AI where different tasks are handled sequentially by separate models: Speech-to-Text (STT) converts audio to text, a Large Language Model (LLM) processes the text and generates a response, and Text-to-Speech (TTS) converts the response back into audio.
- **Speech-to-Speech (STS) Models**: End-to-end models that directly transform input speech into output speech, aiming to reduce latency and preserve more vocal nuances by avoiding intermediate text representations.
- **Half-Duplex vs. Full-Duplex Communication**:
  - **Half-Duplex**: A communication mode where only one party can transmit at a time (either listening or speaking). Most current voice AI systems are half-duplex, making natural overlap impossible.
  - **Full-Duplex**: A communication mode where both parties can transmit and receive simultaneously. This is essential for natural human conversation, allowing for backchanneling, interruptions, and overlapping speech. Moshi is a pioneering example.
- **Paralinguistic Understanding**: The ability of an AI to interpret and respond to non-lexical cues in speech, such as tone of voice, pitch, rhythm, hesitation, and emotional inflection. This goes beyond simply understanding the words spoken and is crucial for truly empathetic and human-like interaction.
- **Semantic VAD (Voice Activity Detection)**: An advanced form of VAD that not only detects when speech is present but also incorporates semantic understanding to improve accuracy in complex conversational contexts, such as distinguishing relevant speech from background noise or non-verbal sounds.
- **On-Device TTS**: Text-to-Speech models optimized to run locally on a device's processor (e.g., a smartphone CPU) rather than requiring cloud-based computation. This approach significantly reduces latency, eliminates API costs, and enhances user privacy.

## Practical Applications

- **Implement LLM Fillers for Latency Masking**: For cascaded systems, design LLM agents to generate conversational fillers or contextual remarks while waiting for tool call results. This masks unpredictable latency (500ms to 4s) and maintains a smoother user experience.
- **Prioritize Full-Duplex for Conversational Agents**: When building voice agents intended for natural, human-like dialogue (e.g., virtual assistants, customer service bots), invest in full-duplex capabilities to enable simultaneous speaking and backchanneling, which are critical for perceived naturalness and engagement.
- **Explore Paralinguistic Data for Richer Interactions**: Beyond transcribing speech to text, investigate methods to capture and utilize paralinguistic features (tone, emotion, hesitation) from audio. This requires a shift towards scientific research in model training to enable AI to understand *how* things are said, not just *what* is said.
- **Leverage On-Device TTS for Scalable Consumer Apps**: For consumer-facing voice applications where cost and privacy are paramount, adopt on-device TTS solutions like Gradion Phonon. This eliminates API fees, ensures data privacy, and allows for scalable usage without burning through fundraising on cloud infrastructure.
- **Design for Robustness in Real-World Environments**: Acknowledge that voice AI will be used in noisy, unpredictable environments. Develop models that are robust to background noise, interruptions, and overlapping speech, moving beyond quiet-room demos.

## Notable Quotes

- "This is mostly just a glorified text model with a voice around it and so you know anything that is not in the text will not be able to be leveraged."
- "We are fighting for latency of the TTS, trying to grab 10 milliseconds, 20 milliseconds, and then you have a tool call or open router that is going to have a latency between 500 milliseconds and 4 seconds. So in a way, sometimes you feel like we're fighting for something that is not the most relevant anymore."
- "Voice is very expensive. Everybody in this room probably knows it. The voice mode of most hyperscalers is run at a loss."
- "It's a science problem, not a prompt engineering problem."

## Takeaways

- The path to natural voice AI requires solving full-duplex, paralinguistic understanding, and cost simultaneously. Full-duplex is necessary but not sufficient; it must be combined with the intelligence, reliability, and personalization found in cascaded systems.
- Cascaded architectures have a hard ceiling on conversational feel due to inherent latency (especially from tool calls) and their inability to handle simultaneous speech, making them architecturally incapable of truly human-like interaction.
- Paralinguistic signals are the most underrated dimension of voice AI quality. Understanding the non-verbal cues in speech is a fundamental scientific challenge, not merely a prompt engineering task, and is crucial for achieving the "Her" moment.
- Cost economics of TTS at scale remain a real constraint, often leading companies to exhaust funding before establishing a user base. On-device TTS solutions are critical for enabling scalable, profitable consumer voice applications.
- The "last mile" in voice AI is the most challenging, demanding deep scientific breakthroughs and engineering innovation to bridge the gap between current capabilities and truly natural, empathetic AI.

## Timestamps

- **0:00** - Introduction to Gradium AI and its mission
- **1:00** - Joel Rogan voice clone demo
- **2:00** - The "Her" movie analogy and current voice AI demos (ElevenLabs, Gradium)
- **4:00** - Explanation of cascaded systems (STT->LLM->TTS)
- **4:30** - Latency challenges in cascaded systems (TTS > 200ms, tool calls 500ms-4s)
- **5:30** - Solution: LLM fillers for masking tool call latency (live demo)
- **7:00** - Definition of Speech-to-Speech (STS) models
- **7:30** - The half-duplex problem in most STS models
- **8:00** - Importance of backchanneling in human conversation (e.g., Japanese culture, up to 20% overlap)
- **8:30** - Demo illustrating annoying half-duplex interaction
- **10:00** - Moshi: The first full-duplex system (demo with co-founder Alex)
- **11:30** - Paralinguistic understanding (Her movie example, Moshi's limitations)
- **12:30** - Moshi's strengths (flow, robustness) and weaknesses (lack of intelligence, observability)
- **13:30** - Scalability and Cost: Voice is expensive, TTS is the main cost driver
- **14:30** - Privacy and the need for on-device solutions
- **15:00** - Introduction to Gradion Phonon: On-device TTS (smartphone CPU, <100M params, private beta)
- **16:00** - Gradion Phonon demo
- **17:00** - Conclusion: Voice is not a commodity, science & engineering are key for the "Her" moment

## Speaker Links

- Twitter: https://x.com/neilzegh
- LinkedIn: https://www.linkedin.com/in/neil-zeghidour-a838aaa7/
