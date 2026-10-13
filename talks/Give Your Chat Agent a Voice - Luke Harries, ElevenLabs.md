# Give Your Chat Agent a Voice

| Field | Details |
| --- | --- |
| **Speaker** | Luke Harries |
| **Affiliation** | ElevenLabs |
| **Type** | Talk |
| **Date** | 2026-05-09 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=DCZZ3AJKzuc) |
| **Tags** | `voice-ai` `chat-agents` `elevenlabs` `tts` `stt` `sdk` `tool-calling` |

## Summary

Luke Harries from ElevenLabs presents a compelling argument that while 2025 was dominated by text-based chat agents, this interaction paradigm is already beginning to feel dated. He asserts that the crucial next upgrade for every existing chat agent isn't merely about refining prompts or enhancing RAG capabilities, but rather integrating a robust voice layer. Voice offers a significantly faster, more intuitive, and inherently more accessible user experience, particularly for individuals facing challenges with traditional keyboards or dyslexia.

The addition of voice also unlocks entirely new interaction paradigms that text-only agents cannot touch. This includes enabling agents to participate in phone calls for customer support, join Zoom meetings to provide real-time information, function as screen readers, or power ambient interfaces. ElevenLabs addresses this need with their new "Voice Engine" product, designed to seamlessly wrap any existing chat agent in a voice interface without requiring a complete rewrite of the underlying agent logic.

The session highlights how this Voice Engine acts as a sophisticated wrapper, handling complex elements like turn-taking, accurate Speech-to-Text (STT), high-quality Text-to-Speech (TTS), and even emotion-aware interruption detection. Developers are provided with intuitive client and server SDKs, alongside Shadcn-based UI components, to facilitate rapid integration. A live demonstration illustrates the ease with which a functional chat support agent can be converted into a voice-enabled agent using a single prompt, proving that adding voice is now a wrapper problem, not a rebuild problem, and preserves all existing tool-calling capabilities.

## Detailed Notes

### The Evolution to Voice and the Limitations of Chat
Luke Harries begins by reflecting on 2025 as "the year of the chat agents," citing examples like Linear, PostHog, and gov.uk integrating chat interfaces as their primary interaction model. While chat is effective for declarative interactions, tool calling, and RAG, it's quickly becoming apparent that text-in/text-out feels dated. Voice is presented as the "natural medium" for the future, offering several key advantages:

- **Speed and Interactivity**: Voice interactions are inherently quicker and more dynamic than typing.
- **Accessibility**: It significantly improves accessibility for users who struggle with keyboards or dyslexia.
- **Omnichannel Capabilities**: Voice unlocks new interaction paradigms such as:
  - **Phone lines**: For customer support agents.
  - **Video calls (e.g., Zoom)**: Agents can join calls to provide real-time corrections or information.
  - **Screen readers**: Enhancing accessibility.
  - **Ambient interfaces**: Enabling hands-free interaction.

The core message is that existing chat agents need to be upgraded to voice agents to meet these evolving demands.

### The Challenge for Existing Agent Implementations
ElevenLabs initially focused on building world-class Text-to-Speech (TTS) models. However, they were quickly drawn into developing full agent solutions, which typically involve a "voice engine" (handling TTS, turn-taking, Speech-to-Text) combined with "agent orchestration" (LLMs, RAG, tool calling, integrations). While this full platform approach works well for companies starting from scratch (e.g., Revolut's customer support), many enterprises had already invested heavily in building their own sophisticated chat agent orchestration layers, complete with extensive evaluations and transcriptions. These companies expressed a strong desire to leverage their existing agent logic rather than completely rebuilding it for voice.

### Introducing ElevenLabs Voice Engine
To address the need for integrating voice into existing agents without a full rebuild, ElevenLabs is launching "Voice Engine" – an early preview product designed as a "first-class primitive." This product specifically encapsulates the voice engine component (STT, TTS, turn-taking) as a wrapper, allowing it to be easily added to any existing chat agent.

### Core Capabilities of Voice Engine
The Voice Engine is engineered to provide a seamless and natural conversational experience:

- **Best-in-Class Models**: It combines ElevenLabs' most accurate models:
  - **Speech-to-Text (STT)**: Powered by "Scribe," noted for its high accuracy.
  - **Text-to-Speech (TTS)**: Utilizes "V3" models for natural-sounding voices.
- **Advanced Turn-Taking**: Features an "emotion context aware" system that can:
  - Detect pauses in speech.
  - Understand "semantic bad" (e.g., filler words, hesitations) to manage interruptions naturally.
- **Voice and Language Diversity**: Supports thousands of different voices and languages.

### Developer Experience and Tooling
ElevenLabs has prioritized a developer-friendly experience for Voice Engine integration:

- **Server SDK**:
  - Allows developers to create a client and then instantiate the Voice Engine.
  - A small wrapper is attached to the existing chat agent.
  - For each new session, the Voice Engine kicks off a loop that proxies all communication to and from the existing chat agent, requiring minimal changes to the agent's core logic.
- **Client SDK**:
  - Extremely simple, requiring approximately three lines of code to add a voice widget to a website.
  - Automatically enables telephony and CSaaS (Customer Service as a Service) capabilities out-of-the-box once the agent is wrapped.
- **UI Components**:
  - A suite of beautiful, well-thought-out UI components are provided.
  - These are based on the popular Shadcn and Vercel design styles, allowing for quick and aesthetically pleasing integration.
- **Rapid Conversion**: The platform aims for a "one prompt" conversion process, where a "skill" analyzes an existing codebase, identifies the chat agent, and generates the necessary wrapping and deployment logic.

### Seamless Tool Calling Integration
A key concern for developers is how adding a voice layer impacts existing tool-calling capabilities. The Voice Engine wrapper is designed to be non-intrusive:

- **Preserves Existing Logic**: The chat agent typically handles the majority of tool calling on its backend. The Voice Engine wrapper operates independently, proxying the conversation without interfering with the agent's tool-calling mechanisms.
- **Client-side and Server-side Tools**: ElevenLabs supports both client-side tools (e.g., for manipulating the DOM) and server-side tools. While most tool calling will be handled by the wrapped agent, there will be future capabilities to proxy specific tool calls through the wrapper if needed.

## Key Technical Concepts

- **Voice Engine (Wrapper)**: A modular component from ElevenLabs that encapsulates Speech-to-Text (STT), Text-to-Speech (TTS), and advanced turn-taking logic. It's designed to be a lightweight wrapper around existing chat agent backends, allowing them to gain voice capabilities without rewriting their core logic.
- **Speech-to-Text (STT) with Scribe**: The process of converting spoken language into text. ElevenLabs uses its "Scribe" model, noted for its high accuracy, to transcribe user input.
- **Text-to-Speech (TTS) with V3**: The process of converting written text into spoken language. ElevenLabs leverages its "V3" models to generate natural and expressive voice responses.
- **Emotion-Context Aware Turn-Taking**: A sophisticated system that manages the flow of conversation. It not only detects pauses but also understands the emotional context and "semantic bad" (e.g., filler words, hesitations) in human speech to enable natural interruptions and prevent the agent from talking over the user.
- **Tool Calling Wrapper**: The Voice Engine is designed to allow the underlying chat agent to continue executing its existing tool-calling functions (e.g., fetching data, performing actions) without modification. The voice layer simply acts as an input/output proxy.
- **SDKs (Server & Client)**: Software Development Kits that provide pre-built functions and libraries for developers to easily integrate Voice Engine into their backend services (Server SDK) and frontend applications (Client SDK).

## Practical Applications

- **Rapid Voice Enablement**: Engineers can quickly add voice capabilities to any existing text-based chat agent with minimal code changes, leveraging the provided SDKs and UI components.
- **Omnichannel Expansion**: Easily extend chat agents to new interaction channels like phone lines for customer support, voice assistants for smart devices, or real-time participation in video conferencing.
- **Enhanced Accessibility**: Improve the user experience for individuals with disabilities (e.g., dyslexia, motor impairments) by offering a voice-first interaction model.
- **Preservation of Existing Logic**: Avoid costly and time-consuming rewrites of complex agent logic, RAG pipelines, and tool-calling implementations by using the wrapper approach.
- **Leveraging Advanced AI**: Gain access to state-of-the-art STT (Scribe) and TTS (V3) models, along with intelligent turn-taking, without needing to develop these complex systems in-house.

## Notable Quotes

- "Chat's cool, but it doesn't feel like you're building the future though. And I really think voice is this natural medium."
- "You've spent a ton of time building these complex chat agents... but to actually add voice to that is then remarkably simple."
- "I think these chat agents will either die chat agents or start adding voice."

## Takeaways

- **Adding voice to existing chat agents is now a wrapper problem, not a rebuild problem.** ElevenLabs' Voice Engine allows developers to integrate voice by simply wrapping their existing agent logic, preserving all prior development effort.
- **The key engineering challenge is handling turn-taking and interruptions naturally.** Voice Engine addresses this with emotion-context aware turn-taking, detecting pauses and semantic cues to enable fluid, human-like conversations.
- **Voice agents can preserve all existing tool-calling capabilities.** The wrapper design ensures that the underlying chat agent's ability to execute tools remains fully functional and unaffected.
- **The integration surface is minimal: SDK + UI components.** Developers can quickly deploy voice agents using straightforward server and client SDKs, complemented by Shadcn-based UI components for a polished user experience.
- **Voice unlocks new interaction paradigms.** Beyond traditional text, voice enables agents to operate across phone lines, video calls, screen readers, and ambient interfaces, significantly expanding their utility and reach.

## Timestamps

- **0:00** - Introduction and the rise of chat agents
- **0:20** - Why voice is the natural next step and its benefits (speed, accessibility, omnichannel)
- **1:00** - ElevenLabs' journey and the problem with existing full agent solutions for developers
- **1:20** - Introducing Voice Engine: the wrapper solution
- **1:40** - Technical details of Voice Engine: Scribe STT, V3 TTS, emotion-aware turn-taking
- **2:00** - Developer experience: Server SDK, Client SDK, UI components
- **2:30** - Live demo setup: converting a chat support agent to voice
- **3:00** - Demo of chat agent conversion and code overview
- **3:40** - Summary of developer options (Voice Engine vs. full platform) and prediction
- **4:00** - Q&A: How Voice Engine handles tool calling

## Speaker Links

- LinkedIn: https://www.linkedin.com/in/luke-harries
