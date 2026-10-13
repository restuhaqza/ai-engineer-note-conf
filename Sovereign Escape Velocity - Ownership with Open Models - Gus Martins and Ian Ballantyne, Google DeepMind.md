# Sovereign Escape Velocity: Ownership with Open Models

| Field | Details |
| --- | --- |
| **Speaker** | Gus Martins, Ian Ballantyne |
| **Affiliation** | Google DeepMind |
| **Type** | Talk |
| **Date** | 2026-06-10 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=SS-A8sE7hkw) |
| **Tags** | `coding-and-developer-tools` `reasoning-and-models` `infrastructure-and-deployment` `data-and-model-adaptation` `evals` |

## Summary

Gus Martins and Ian Ballantyne of Google DeepMind explain why an open model family (Gemma 4) matters alongside Google's hosted Gemini. Gemini is the easiest and strongest starting point, but a hosted API cannot help when data must stay inside your own infrastructure.

Ownership here means running inference where you choose, accessing the model itself, and customizing it. Gemma 4 ships in four sizes: two small phone and IoT models (E2B and E4B, where "E" means effective), a 26B mixture-of-experts, and a 31B dense model presented as the family's strongest.

The pair demonstrate phone agents that reason about which skill to call, a local 26B run in LM Studio on an M4 Mac, and a translation pipeline that fans work out to sub-agents on one machine. They also note Gemma 4 moved to an Apache 2.0 license to reduce legal friction for institutions.

Ballantyne frames the decision to offload work as four thresholds — capability, hardware fit, latency and cost — and stresses that self-hosting transfers uptime, maintenance and energy costs to the owner. Changing the endpoint is easy; evaluating the workflow is the real work.

## Key Takeaways

- **Two families, two jobs.** Gemini is Google's strongest hosted option; Gemma supplies control a hosted API cannot.
- **"E" means effective.** E2B and E4B name an effective parameter scale, not full weight counts, so they fit on phones and constrained devices.
- **26B A4B is a mixture of experts.** Only about 4B parameters are active per token, which keeps hardware needs low.
- **31B dense is the family's strongest.** Martins estimates the two larger models placed 4th and 7th among open models on LM Arena, often against much larger rivals.
- **Licensing changed.** Gemma 4 uses Apache 2.0, which is easier for sovereign and institutional adoption than a custom license.
- **Four thresholds decide offloading.** Capability, hardware fit, latency and cost all have to hold before moving work to owned hardware.
- **Ownership changes the budget.** There is no per-token bill, but energy, scheduling and maintenance become part of the design.
- **Switching is a one-line change.** Point an OpenAI-compatible client at LM Studio or Ollama, select the local Gemma model, then evaluate on your own tasks.

## Detailed Notes

### Why own the model? · 01:31
- Martins works on Gemma and says plainly that Gemini is Google's best and easiest starting point — but there is more to the story than raw capability.
- Some situations need inference on your own hardware, a customized model, or proprietary data that cannot leave your infrastructure.
- A hosted model cannot satisfy those constraints through intelligence alone, so an open model fills the gap.
- Gemini and Gemma are complementary: one is hosted and reached through an API, the other gives control and access to the model itself.

### Gemma 4 sizes and what "effective" means · 02:18
- Four sizes launched: **Gemma 4 E2B** and **E4B** for phones, IoT and smaller devices, plus **Gemma 4 26B A4B** (a mixture of experts) and **Gemma 4 31B Dense**.
- The `E` stands for **effective**: the name describes an effective parameter scale rather than counting every parameter associated with the model.
- Martins explains E2B as roughly a 5B-parameter model whose transformer computation has a much smaller footprint; extra token-mapping parameters can live outside accelerator memory.
- So "effective" is not a memory specification — actual memory also depends on representation and runtime configuration.
- E2B and E4B accept text, vision and audio input but produce text, and support thinking, coding and function calling.

### Capability per size, and enough for the job · 03:42
- In the mixture-of-experts 26B, only part of the model works on each token — Martins describes roughly 4B-sized working components. That is active computation, not the space needed to store the whole model.
- The 31B dense model is presented as the strongest of the family.
- Martins estimates the two larger models placed fourth and seventh among open models on LM Arena, with many top-20/30 rivals two or three times larger, sometimes twenty times larger.
- Both larger models can run on a desktop or behind a server endpoint; hosting on someone else's cloud still leaves you in control of the model and its deployment.
- His stance: you may not need the most intelligent model on earth to summarize email, help with code, or search documents.
- Hardware comparison: a 31B model on one GPU versus unnamed competitors needing 200 GB and perhaps four or five GPUs (workload unspecified). The launch announcement gives a narrower condition: unquantized bfloat16 weights fit on an 80 GB H100.

### Licensing and national adaptations · 07:03
- Sovereignty here means owning and adapting the model, and not depending on a provider's continued service or permission to use an endpoint.
- Through Gemma 3, the family used a custom license that Martins describes as commercially friendly — but custom terms still require legal interpretation, and he jokes about an eighteen-month procurement review.
- Gemma 4 moves to **Apache 2.0**, and he says future releases will follow; this does not retroactively change earlier models.
- Examples: Gemma used in parts of Ukraine's services, a Bulgarian-language model based on Gemma 2 (with work to move it onto Gemma 4), and a Brazilian Portuguese adaptation based on Gemma 3.
- A changing fine-tuning problem: when the base model is already strong in a language, gains are hard to find. Martins describes substantial work for maybe a one-percent gain.
- He reports the 31B model ranking top two or three in many languages on LM Arena, without naming them.

### Four thresholds for offloading agent work · 09:38
- Ballantyne picks up the angle of agentic workloads, which cost tokens through repeated instructions, intermediate outputs and actions.
- He cites OpenRouter's December 2025 State of AI report, which shows programming among the highest-token task categories when input and output are combined. The report covers traffic through OpenRouter, not all AI use.
- A laptop GPU can absorb document processing, data analysis, research, or suitable coding — but not, in his view, redesigning an entire application architecture. Bounded jobs like refactoring, analysis and small modular code are better fits.
- Four thresholds that have to hold together:
  - **Capability** — can the model do this specific task?
  - **Hardware fit** — can the intended machine run it?
  - **Latency** — can it respond within the task's time budget?
  - **Cost** — does owned or leased compute make economic sense?
- User-facing tasks may need answers within a couple of seconds; batch work can run on a different schedule.

### Phone agents and local desktop runs · 12:45
- A mobile demo (credited to Cormac's team) puts agent skills on a phone: the user can speak, show images, show the surrounding world, or chat.
- The model chooses among skills that can trigger device apps like calendars and maps, or invoke developer-defined skills.
- Ballantyne highlights reasoning about which action to take, followed by reliable defined function calls, as an improvement over the previous generation. The playground is **Google AI Edge Gallery**, available on iOS and Android.
- On desktops, the 26B runs in **LM Studio** at roughly 26 GB of RAM including context on an M4 Mac with about 48 GB of unified memory — a reminder that active parameters and runtime memory differ.
- Removing the per-token bill does not remove cost: GPU and NPU utilization consumes energy, so when work runs (immediately versus overnight) becomes a product decision.
- In the terminal demo, an orchestrator takes the Gemma 4 announcement and farms translation work out to sub-agents in separate windows on the same machine. The planned final step is a small webpage of results; the demo clearly shows translations being generated, not the finished page.
- Enterprise angle: earlier designs needing 300B+ models and multiple GPUs might now run on a single H100 or A100, or an L4 in some cases. **MedGemma** is offered as a medical family that can work with private data under an organization's control.

### Point an existing client at a local model · 18:14
- The integration is intentionally small: point an OpenAI-compatible client at LM Studio or Ollama and choose the locally served Gemma model.
- A short Python example changes `base_url` to the local server and sets `model` to the local Gemma identifier, then sends a translation request. The code shown builds one request; an orchestrator can fan the same document out to several languages.
- Compatibility supports this endpoint-and-model pattern; it does not imply complete feature equivalence.
- Ballantyne's next steps are empirical:
  1. **Drop the model into an existing workflow** and observe what it handles well.
  2. **Separate tuning needs from capability limits** — some failures are addressable through adaptation, others reflect task complexity.
  3. **Strengthen the evaluation suite for your tasks**, since general benchmarks only identify candidates.
- The endpoint change starts the experiment; task evaluation decides whether the substitution is useful.

### The responsibilities that come with control · 19:20
- Self-hosting transfers uptime, downtime, maintenance and ongoing operating costs to the owner; buying hardware adds upfront capital expenditure.
- Those obligations belong in the same decision as model quality and inference cost.
- Mobile deployment adds a support matrix — supported devices, their accelerators and available RAM — but can enable offline use and private data that never leaves the device.
- Enterprise scaling needs the same attention to infrastructure sizing and continuing costs, at a larger scope.
- Ballantyne closes by encouraging experiments on concrete tasks, using benchmarks as signals of feasibility, and asking for feedback on deployments, fine-tuning and the problems encountered.

## Resources

- [Gemma 4 launch announcement](https://blog.google/innovation-and-ai/technology/developers-tools/gemma-4/) — sizes, Apache 2.0 license and release details.
- [Gemma 4 model card](https://ai.google.dev/gemma/docs/core/model_card_4) — architecture tables, parameter definitions and evaluations.
- [Google AI Edge Gallery](https://github.com/google-ai-edge/gallery) — installable mobile playground for local multimodal models.
- [OpenRouter State of AI](https://openrouter.ai/state-of-ai) — December 2025 study of more than 100 trillion tokens.
- [LM Studio OpenAI compatibility](https://lmstudio.ai/docs/developer/openai-compat) — point an existing client at a locally served model.
- [Ollama OpenAI compatibility](https://docs.ollama.com/api/openai-compatibility) — connect OpenAI-compatible apps to local models.
- [MedGemma](https://developers.google.com/health-ai-developer-foundations/medgemma) — developer overview of the medical model family.
