# Why MLX

| Field | Details |
| --- | --- |
| **Speaker** | Not stated in the source |
| **Affiliation** | Neywa Labs |
| **Type** | Talk |
| **Date** | 2026-05-11 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=zTLJNHj0DeQ) |
| **Tags** | `infrastructure-and-deployment` `speech-and-audio` `vision-and-video` `observability-and-reliability` |

## Summary

This talk argues for running AI agents on the device you own instead of in the
cloud. The speaker is an MLX contributor, and the drive is personal. In 2020 his
father lost his sight, and because his father lives in Africa, where connections
are unreliable, a cloud-only reading assistant would often be out of reach. That
made on-device inference a requirement, not a preference.

MLX is an array framework for Apple silicon — the layer that runs models on
Macs, iPhones, and iPads, roughly what PyTorch or TensorFlow is for other
hardware. Around it, projects like MLX-VLM (vision) and MLX-Audio (speech) let a
device see, hear, and talk.

The demonstrations cover real-time object detection, a local multimodal chat,
on-device video generation, and a small robot with a cloned voice. A closing Q&A
explains that MLX uses the GPU rather than Apple's Neural Engine, how to watch
GPU activity with mactop, which omni models to try, and how TurboQuant shrinks
the memory cost of long contexts.

## Key Takeaways

- **On-device is an accessibility choice.** Work that must keep running without
  a reliable connection has to run locally.
- **MLX is the Apple-silicon layer for models.** The speaker reports 1.5
  million-plus downloads, 4,000-plus ported models, and day-zero support for
  open models such as Gemma 4.
- **Local does not mean tiny.** He reports large models on an early M1 MacBook
  and Gemma 4 26B on an iPhone using storage, while noting that fitting a model
  and running it well differ.
- **Vision and speech compose.** MLX-VLM handles seeing; MLX-Audio covers
  text-to-speech, speech-to-text, and speech-to-speech.
- **Voice pipelines come in two shapes.** A native speech-to-speech model, or a
  modular chain of speech recognition, language model, and speech synthesis.
- **Inference runs on the GPU.** Neural Engine use stays flat because MLX uses
  the GPU; Core ML is the Neural Engine route, and he calls it rough for
  developers.
- **Watch the hardware with mactop,** a terminal tool that shows CPU and GPU
  activity in real time.
- **TurboQuant shrinks the KV cache.** He reports roughly a 4× smaller cache in
  the displayed test, which matters as context grows.

## Detailed Notes

### What happens when an agent cannot depend on the internet? · 00:17
- The goal: run AI agents, including voice agents, fully on a phone or MacBook,
  offloading subscription cost so you pay only your energy bill.
- In 2020 his father lost his sight, around the time Apple released its first M1
  chip. His father is a voracious reader, so the speaker promised to help him
  read again; because his father lives in Africa, where plans are poor, a
  cloud-only solution did not fit.
- **On-device inference was an accessibility requirement,** not a preference.

### From an array framework to accessible perception · 02:27
- In 2023 a GitHub search led him to **MLX**, an array framework for Apple
  silicon — think PyTorch or TensorFlow for Apple hardware. He began
  contributing; three years later he reports over 1.5 million downloads, 4,000+
  ported models, and day-zero support for open models, including Gemma 4.
- Vision came first. A 2021 hackathon project used goggles to describe nearby
  objects; **MLX-VLM** reworked the idea for a phone camera, so you can point and
  ask what the device sees.
- **Omni models** add audio input, letting someone who cannot easily type speak
  to control the camera and ask about the scene.
- He challenges the idea that local means small: he reports models with hundreds
  of billions of parameters on an early M1 MacBook, and Gemma 4 26B on an iPhone
  using storage. The caveat: fitting a model and getting an acceptable
  interactive experience are separate questions, and no throughput is given.

### Assembling a voice agent for the available hardware · 05:23
- Audio followed vision, both for accessibility and for a JARVIS-style wish to
  control a computer by speaking to it.
- **Text-to-speech (TTS, text into audio):** their model **Marvis** is reported
  to generate audio in under 100 ms; no model version, hardware, or measurement
  boundary is given.
- **Speech-to-text (STT, transcribing speech):** real-time transcription supports
  dictation apps; he mentions Wispr Flow and superwhisper as examples.
- He suggests pointing Claude Code or Codex at **MLX-Audio** to build something
  comparable "in about ten minutes" — an estimate, not a shown build.
- **Speech-to-speech** lets the computer answer aloud. Python came first for
  development speed; Swift support enables native apps combining audio and
  vision.
- Two assembly options: **native speech-to-speech** (one model handles the
  interaction) or a **modular pipeline** of speech recognition → language model →
  speech synthesis, where each component is chosen separately. The modular route
  lets one app adapt from a first-generation M1 to newer machines by swapping
  components.

### Detecting objects and masking a background · 08:00
- A Python command starts a camera-analysis app running Roboflow's **RF-DETR**
  detection model locally, in real time. He turns the internet off during the
  demo, and processing continues.
- Lifting a glass, the detector labels it as wine — a joke about being sober.
- The demo runs on a Mac; phone execution is also described as possible, with
  Swift support coming. A second variation applies the same perception to
  background blur.
- A real-time **person mask** (a pixel-level segmentation of the speaker)
  separates the foreground from the background.
- **The mechanism is the point:** a local segmentation result becomes an input to
  a product's rendering behavior, not merely an analysis output.

### Opening a multimodal chat interface · 09:27
- **Gemma 4** had arrived the previous week (published release date: April 2,
  2026). MLX-VLM's `mlx_vlm.chat_ui` entry point takes a model and opens a
  **Gradio** interface; a short launcher script passes the choice through.
- The launch stalls, then on retry tries to download model files before he
  switches the internet off again. **Local inference still needs model files
  available on the device.**
- He selects an image and asks it to describe the image in detail; the reply
  identifies the person and describes details including the biography.
- The demo machine has 96 GB of memory and runs all the models shown, at once,
  in real time. On Apple silicon this is **unified memory** shared by CPU and
  GPU; the demo uses the GPU. A planned live audio demo fails that morning over
  a Swift-branch problem, so community recordings stand in.
- **Grounded visual reasoning** turns the model into a way to inspect video: ask
  it to segment every jet, and colored overlays mark the aircraft and their
  contrails. He suggests home security on a MacBook and dashcam review.
- **MLX Video** generates video from a text prompt. A community cartoon is not
  one long clip; successive generations continue from earlier video to build a
  story. He says that chained system runs on a MacBook with 16 GB; the model,
  settings, and time are not given.

### Giving applications—and a robot—a voice · 14:27
- An audience member, Adrian, built an app called **Locally** and added
  MLX-Audio plus Marvis TTS to give it spoken output. The clip reads prose aloud,
  a native reading experience; the speaker stresses that design matters
  alongside the models.
- **Robotics** extends the same combination into a physical device: he powers a
  **Reachy Mini** robot with MLX-Audio and MLX-Vision, using its camera and audio
  input.
- A recording shows what he calls real-time cloning of the Iron Man JARVIS
  voice: after "Hey, JARVIS," the robot answers with a friendly greeting.
- That is the destination: agents that hear, see, and speak on an iPhone, iPad,
  Mac, or robot, once the libraries are composed into one interaction.

### Where inference runs, and how to observe it · 17:07
- An audience member notes that Neural Engine (Apple's dedicated AI accelerator)
  use stays at zero while MLX runs.
- The explanation: the demonstrated path uses the **GPU**. MLX also supports CPU
  execution, so it is not GPU-only. For the Neural Engine he points to **Core
  ML**, while calling its developer experience and private-API issues obstacles.
- He mentions internal work on hybrid GPU-plus-Neural-Engine inference,
  contingent on Apple fixing APIs, and hope for WWDC. He speculates some Neural
  Engine parts may move into the GPU by the M5 generation, but says Apple's
  direction is unknown. These are prospective, not demonstrated.
- To observe the current path he recommends **mactop**, a terminal monitor by
  Carsen Klock showing CPU and GPU activity as an overlay. In the demo, GPU usage
  rises after inference starts and a prompt is entered, tying output to visible
  local compute.

### Choosing an omni model without overpromising · 19:32
- For **omni models** (accepting more than one input type) he names **Gemma 4
  E2B and E4B**, whose edge variants take image, audio, and text in combinations,
  and **Qwen3-Omni**, a larger local option he calls roughly 30 billion
  parameters. The choice depends on the task: the modalities must fit the
  application, and the model must fit the device.
- **Key limitation:** do not expect these local models to match contemporary
  Claude/Opus models; his suggestion they might reach that level within six
  months is a forecast. He still reports inference over hundreds of images in
  parallel and over many documents.
- **TurboQuant** targets **KV-cache memory** — the attention keys and values a
  model keeps during inference. He says he implemented it about thirty minutes
  after seeing the paper, and a March 25 post about it drew roughly 700,000
  views. The source notes the paper was submitted to arXiv on April 28, 2025,
  while Google's announcement came March 24, 2026.
- Figures from the displayed table: full model 0.703 GB average cache with 6/6
  exact matches; TurboQuant 2.5 at 0.143 GB (4.9× smaller); TurboQuant 3.5 at
  0.185 GB (3.8× smaller). "Exact match" means agreement on the six displayed
  cases, not general quality equivalence.
- He reports almost doubled throughput at around 300,000 tokens of context; the
  workload and hardware are not specified. Caveats: a smaller KV cache does not
  cut total memory by the same factor, and cache compression does not extend a
  model's supported window — Google lists 128K for Gemma 4 edge variants and up
  to 256K for larger ones. His million-token on-device claim therefore depends on
  model and hardware.

## Resources

- *MLX* — Apple's machine-learning array framework, with Python and Swift
  interfaces and shared CPU/GPU memory.
- *MLX-VLM* — local multimodal inference and fine-tuning, with a Gradio chat
  interface.
- *Gemma 4 launch announcement* — model sizes, input modalities, and context
  windows.
- *MLX Audio* — speech synthesis, transcription, and speech-to-speech for Apple
  silicon.
- *Marvis TTS v0.2* — streaming speech model with MLX Audio examples.
- *RF-DETR* — Roboflow's object detection and segmentation project.
- *MLX Video* — image, video, and audio generation on Mac through MLX.
- *mactop* — real-time Apple-silicon system monitoring in the terminal.
- *TurboQuant paper* and *Google Research's explainer* — vector quantization for
  memory-efficient inference.