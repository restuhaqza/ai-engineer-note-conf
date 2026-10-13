# Accelerating AI on Edge

| Field | Details |
| --- | --- |
| **Speaker** | Chintan Parikh and Weiyi Wang |
| **Affiliation** | Google DeepMind |
| **Type** | Talk |
| **Date** | 2026-05-05 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=Lm8BLHkxiAo) |
| **Tags** | `edge-ai` `on-device` `gemma` `litert` `google-ai-edge` `hardware-acceleration` `npu` |

## Summary

Chintan Parikh and Weiyi Wang from Google DeepMind present a comprehensive overview of accelerating AI on edge devices, focusing on the practical deployment of smaller, more capable models directly onto user devices. They highlight how this shift enables a new era of on-device AI workloads, moving beyond simple inference to sophisticated agent capabilities that support reasoning, tool calling, and structured output generation. The core message emphasizes the critical tradeoffs addressed by edge AI, including significantly reduced latency, enhanced privacy by keeping data local, optimized cost by offloading cloud compute, and robust cross-platform deployment.

The talk delves into Google's specific contributions to this field, particularly the Gemma 4 edge models (2B and 4B parameters) and the underlying Google AI Edge stack, powered by LiteRT. LiteRT serves as a unified, cross-platform runtime built on TensorFlow Lite, designed to execute models efficiently across various hardware accelerators like CPUs, GPUs, and NPUs. This framework supports a wide array of devices, from mobile (Android, iOS) and desktop (macOS, Linux, Windows) to web and IoT platforms, ensuring broad accessibility and deployment flexibility for developers.

Through live demonstrations and performance metrics, Parikh and Wang showcase the advanced capabilities of Gemma 4 edge models, such as on-device agent skills, local tool calling, and the ability to manage complex workflows without server round trips. They underscore the importance of NPU acceleration for achieving substantial performance gains and energy efficiency, making real-time, privacy-sensitive AI applications practical and scalable. The presentation ultimately paints a picture of a future where AI agents operate autonomously and intelligently directly on the device, driven by Google's robust edge AI ecosystem.

## Detailed Notes

### Introduction to Edge AI and Gemma 4
- **Speakers and Focus**: Chintan Parikh (Product Manager for LiteRT, part of Google AI Edge) and Weiyi Wang. The talk focuses on Gemma 4's 2B and 4B edge models, optimized for on-device deployment
- **Evolution of Models**: Google's Gemma family includes models as small as 270 million parameters, available on Hugging Face for fine-tuning. The significant evolution with Gemma 4 is the shift from basic chatbot capabilities to more autonomous agents supporting reasoning and sophisticated features

### Benefits of On-Device AI
- **Latency**: Critical for real-time applications like camera filters, background replacement in video calls, and any scenario where immediate response is paramount. On-device processing eliminates network round-trip delays
- **Privacy**: Essential for handling sensitive data, such as summarizing private documents or personal information, as data remains on the device
- **Offline Use**: Enables functionality in environments with poor or no network connectivity
- **Cost Efficiency**: Offers a hybrid approach to balance cloud and edge processing, reducing token usage and overall operational costs associated with cloud inference

### Gemma 4 Edge Models: Specifications and Capabilities
- **Gemma 4 E2B**:
  - RAM Usage: Approximately 1-2 GB (after quantization)
  - Use Cases: Suitable for voice interfaces, summarization, and low-latency local processing
- **Gemma 4 E4B**:
  - RAM Usage: Higher, designed for more demanding platforms like laptops or IoT devices
- **New Agent Capabilities (Gemma 4 E2B and E4B)**:
  - **Function Calling**: Built-in support for tool calling, allowing models to interact with other local APIs. The core inferencing remains on the edge, but external APIs can be invoked
  - **Structured JSON Output**: Native support for generating structured JSON output, integrated directly into the model architecture rather than relying solely on prompt engineering
  - **Chain of Thought**: Features a "thinking mode" (demonstrated in the gallery app) that helps users understand the model's internal thought process
  - **Hardware Optimization**: Models are optimized for native hardware support, ensuring seamless execution across multiple platforms and various hardware accelerators
- **Availability**: Gemma 4 edge models are Apache 2.0 licensed and available on Google's Hugging Face page

### Google AI Edge Stack: LiteRT
- **Core Framework**: LiteRT is Google's on-device framework, built upon the established TensorFlow Lite framework
- **Trusted Foundation**: It leverages a widely deployed foundation, with over 100,000 apps, billions of active users, and numerous daily interpreter invocations
- **Model Format**: Utilizes the TensorFlow Lite (TF Lite) model format, which is inherently cross-platform. Existing TensorFlow Lite models are compatible with LiteRT
- **Multi-Framework Support**: LiteRT accepts models from various frameworks, including TensorFlow Lite, PyTorch (via conversion), and JAX (via conversion)
- **Conversion Path**: The LiteRT Torch tool facilitates the conversion of PyTorch and JAX models into the TF Lite format for deployment
- **Model Explorer Tool**: A utility to explore the model graph, enabling developers to make informed decisions about quantization (e.g., mix-precision) and graph optimization

### Deployment and Acceleration
- **Cross-Platform Support**: LiteRT offers unified architecture for deployment across Android, iOS, macOS, Linux, Windows, Web, and IoT devices (e.g., Raspberry Pi, demonstrated with a robot)
- **Hardware Acceleration**:
  - **CPU and GPU**: Universal support for these common accelerators
  - **NPU (Neural Processing Unit)**: A key focus for acceleration. Integrations are complete with Qualcomm and MediaTek, with ongoing efforts for additional partners. NPUs offer significant performance and energy benefits
- **Compilation Options**: Supports both Ahead-of-Time (AOT) and Just-in-Time (JIT) compilation for deployment flexibility

### Tools and Resources
- **AI Edge Portal**: A cloud-based benchmarking service designed for broad Android deployment. It helps developers determine optimal compilation strategies (AOT vs. JIT) and assess model reliability across diverse device fleets
- **Gallery App**:
  - A Google-created playground app to demonstrate the capabilities of Gemma 4 edge models
  - Showcases agent skills, audio scribe, ask image, and chat experiences, all running on-device
  - Provides sample code and is forkable for custom development
  - **Demonstrated Use Cases**:
    - **Augmenting Knowledge**: A skill to query Wikipedia for encyclopedia questions
    - **Journal Entry with Score & Comment**: A sleep tracking agent that analyzes mood trends based on user input
    - **Expanding Core Capabilities**: Pairing photos with music, demonstrating image understanding and on-device music generation
    - **Managing Complex Workflows**: An example involving local calls, showcasing navigation of multiple apps and complex workflows
  - Allows users to select the desired accelerator (CPU, GPU, NPU support coming soon) within the app
- **GitHub Repository**: The sample app is open-source on GitHub, allowing users to fork it, make changes, and share custom skills with the community
- **CLI Tool**: A new command-line interface tool with Python binding support for easier deployment

### Performance Metrics
- **NPU Performance**: Up to a 13x performance boost in certain use cases
- **iOS Performance**: Approximately 56 tokens per second
- **Desktop/IoT Performance**: Detailed performance numbers, including quantization specifics, are available on Google's Hugging Face page
- **LiteRT vs. Llama Runtime**: Up to 35 times faster on mobile, at par on desktop, 3 times faster on IoT
- **Coverage**: Gemma models tested and shown to run on Android, iOS, Linux, and Raspberry Pi

## Key Technical Concepts

- **On-device AI / Edge AI**: Running AI models directly on the user's device rather than in the cloud
- **LiteRT**: Google's unified, cross-platform on-device framework for executing AI models, built on TensorFlow Lite
- **Gemma 4 Edge Models**: Google's family of smaller, efficient LLMs (2B and 4B parameters) optimized for on-device inference
- **Quantization**: Reducing precision of model weights and activations for smaller size, faster inference, and lower memory/power consumption
- **Function Calling / Tool Calling**: A model's ability to identify when an external tool or API needs to be invoked and generate correct parameters
- **Structured JSON Output**: Generating responses in a predefined, machine-readable JSON format
- **Chain of Thought**: A reasoning technique where the model explicitly shows intermediate reasoning steps
- **Hardware Acceleration (CPU, GPU, NPU)**: Utilizing specialized hardware to speed up AI model inference. NPUs are designed specifically for neural network operations
- **AOT vs. JIT Compilation**: Ahead-of-Time compilation optimizes for a specific target before runtime; Just-in-Time compilation occurs during runtime for dynamic optimizations
- **Cross-platform Deployment**: Deploying the same AI model and runtime across multiple OS and device types without significant modifications

## Practical Applications

- **Real-time Camera Effects**: Applying filters, background replacement, or AR overlays in video calls with minimal latency
- **Privacy-Sensitive Data Processing**: Summarizing confidential documents, analyzing health data, or performing sentiment analysis on private communications on-device
- **Offline Productivity Tools**: AI-powered text summarization, language translation, or smart replies without internet connectivity
- **Voice Interfaces and Local Agents**: Responsive voice assistants, personal journaling agents, or mood trackers processing commands locally
- **Smart Home Security**: Local AI on Raspberry Pi for facial recognition on home security cameras, reducing cloud costs and enhancing privacy
- **Image Understanding and Content Generation**: Generating music based on a photo's "vibe," or complex image analysis entirely on-device
- **AR/VR**: Real-time updates and interactions within AR/VR environments by processing camera feeds locally
- **Complex Workflow Management**: Orchestrating multi-step agent tasks, such as navigating multiple local applications

## Notable Quotes

- "Real-time latency is king over there."
- "On-device always offers this hybrid approach where if you're looking for like how do I really offset running things on the edge versus on cloud and where can I get the best balance? I think there's a there is something here."
- "The NPU is going to give you at least like 3 to 10x improvement in performance and it's going to be a game changer in terms of the amount of energy you're going to use and the amount of performance you're looking for to unlock those use cases."

## Takeaways

- On-device AI is now practical for real agent workloads, not just inference demos
- Cross-platform deployment requires a unified runtime like LiteRT
- NPU acceleration is key for performance without draining battery
- Privacy and latency are the primary drivers for edge AI adoption

## Speaker Links

- Weiyi Wang LinkedIn: https://www.linkedin.com/in/weiyiwang1993
- Chintan Parikh LinkedIn: https://www.linkedin.com/in/chintansparikh
