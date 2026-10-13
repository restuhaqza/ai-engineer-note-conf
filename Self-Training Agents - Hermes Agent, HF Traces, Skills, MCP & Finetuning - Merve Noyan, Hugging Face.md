# Self-Training Agents: Hermes Agent, HF Traces, Skills, MCP & Finetuning

| Field | Details |
| --- | --- |
| **Speaker** | Merve Noyan |
| **Affiliation** | Hugging Face |
| **Type** | Talk |
| **Date** | 2026-05-13 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=OV56RddyFuU) |
| **Tags** | `rag-context-and-search` `evals` `infrastructure-and-deployment` `apis-mcp-and-protocols` `vision-and-video` `reasoning-and-models` |

## Summary

Merve Noyan, on Hugging Face's open-source team, frames the talk as putting an AI engineer at your fingertips. The goal is to give a coding assistant more than code writing: let it pick a model, adapt it to your data, and run the infrastructure underneath. She tours the open ecosystem that makes this possible — open-weight and open-source models, the Hugging Face Hub, benchmark datasets, and inference providers.

The session is a walkthrough rather than a single result. Noyan shows how to find and serve a model, connect the Hub to a local or hosted coding agent, and use agent skills so an assistant can launch a training job for you. She then demonstrates Hermes Agent, a persistent assistant with memory and messaging integrations, and Agent Traces, a Hub feature for uploading and inspecting coding-agent sessions.

The closing workflows tie the pieces together: an MCP (Model Context Protocol — a standard way for agents to call tools) route lets an assistant search the Hub and call AI apps, and a colleague's project used the same approach to OCR 30,000 research papers. She also introduces Hugging Face Buckets for storage that repeated training and processing jobs can reuse.

## Key Takeaways

- Openness has layers: open-weight models with noncommercial licenses, commercially usable releases (MIT, Apache 2.0), and fully open code plus a harness.
- Control matters: with weights and the harness in hand you can inspect changes, shrink, quantize, or fine-tune a model, and keep inference local.
- Noyan argues open models have caught up to closed ones, citing a talk-time Intelligence Index chart — not a permanent ranking.
- Use Benchmark Datasets (SWE-bench, Humanity's Last Exam, AIME) to shortlist, then still test on your own task.
- Inference Providers adds hosting options plus a Tool Use column to spot models that can call tools in an agent.
- Skills let a coding agent run real work, such as fine-tuning a vision-language model, after asking a few configuration questions.
- Hermes Agent adds persistent memory and messaging (Slack, WhatsApp) and can run on a local model or Inference Providers.
- Agent Traces turns saved sessions from Codex, Claude Code, and Pi into inspectable data you can review, redact, and optionally train on.
- Local serving starts from your application and the model's format and hardware fit (a 4-bit Gemma 4 fits a 24 GB L4 GPU).

## Detailed Notes

### Open models: control, capability, and the case for them · 00:00
- Noyan opens with "having an AI engineer at your fingertips": an assistant that can also choose a model, adapt it to your data, and run the infrastructure.
- Openness comes in layers: open-weight models with noncommercial licenses, commercially usable releases (she cites DeepSeek and licenses like MIT and Apache 2.0), and fully open code and harness.
- She ties visibility to reported declines in Claude's performance: if you control the model and harness, changes and regressions don't happen without your knowledge.
- Weight access lets you shrink, quantize, or fine-tune; deploying to an edge device or browser can keep data local — but only if the surrounding app and tools don't send it elsewhere.
- She argues open models have caught up, pointing at an Artificial Analysis Intelligence Index chart; this is a talk-time comparison, not a claim about today's leaderboard.

### Find a capable model, then choose how to serve it · 02:02
- She uses GLM 5.1 in her own coding setup as an example of a strong open model.
- The Hugging Face Hub is the infrastructure layer: she estimates close to three million models, plus datasets and Spaces.
- Filtering for agentic models narrows the field; vision-language models can double as computer-use agents that read screenshots and decide where to click.
- She points to Gemma 4 (omni and agentic), Qwen 3.5, and Kimi K2.5 as examples of vision arriving in a model's first release, and expects that to spread.
- Serving is a separate choice: vLLM, MLX, and llama.cpp's `llama-server` are options; for a compatible Hub repo, one command like `llama-server -hf <id>` can be enough.

### Narrow the choice with benchmarks and provider options · 04:21
- The Hub's Benchmark Datasets filter is a more focused start than browsing trending models.
- She names SWE-bench Pro, Humanity's Last Exam, and AIME; in a SWE-bench coding comparison, GLM 5.1 sits top of the displayed open-model list.
- A strong benchmark is only an input — still try the model on your own task.
- Inference Providers offers hosted access through providers such as Groq, Cerebras, and Novita, and exposes cost and speed options.
- Its Tool Use column helps find models suitable for an agent that must call tools.

### Connect the Hub to a coding agent · 05:48
- The Hugging Face MCP server connects an assistant to Hub capabilities, while skills supply procedures for tasks like training.
- Local coding agents are another entry point using Hub-distributed models; Noyan favors Pi for simple setup.
- In the arrangement she describes, llama.cpp serves the model and Pi consumes that endpoint; Inference Providers is a possible remote route, though not established in the demo.
- `llama-agent`, a llama.cpp binary, can start an agent directly from a model ID; the local-agent docs are the reference for that path.

### Hermes: memory, setup, and integrations · 07:38
- Hermes Agent adds a persistent assistant around the model choice; Noyan prefers its memory management to OpenClaw's.
- She draws a line between learning via saved memory and reusable skills versus fine-tuning: retaining knowledge or a procedure doesn't change model weights.
- Hermes can use a local model or Hugging Face Inference Providers; its setup wizard collects credentials and configures integrations such as Slack or WhatsApp.
- She recommends GLM 5.1 from experience: after a Slack integration failed, she asked Hermes running that model to repair it and reports it succeeded.
- Her next experiments (Gemma 4, a rumored MiniMax model) were still prospective, not demonstrated results.

### Turn saved sessions into inspectable training material · 09:17
- A session produces data, and the Hub's Agent Traces accepts sessions from Codex, Claude Code, and Pi.
- It arrives as a repository type for traces; in the walkthrough, opening the traces column in Dataset Viewer shows a parsed session you can explore rather than a raw log.
- Content capture has rules: review and redact secrets, private code, and personal data before publishing traces.
- Uploading traces does not train anything by itself. Saved session files come first, then inspection, with an optional training step.
- Hermes trace support is something she expects, not something she demonstrates as available.

### From a training request to a model on the Hub · 12:03
- Hugging Face Skills give coding agents procedures across the ecosystem: the CLI skill manages repos, jobs, and demos; other skills cover the LLM trainer, Gradio, and datasets.
- To find a local model, start from the application: the Hub's `Other → Apps` filter shows models supported by LM Studio, Jan, llama.cpp, and similar tools.
- A model repo shows format and hardware fit — GGUF (a file format used by llama.cpp, Ollama, LM Studio, and others) and, for example, a larger Gemma 4 quantized to four bits fitting an L4 GPU with 24 GB VRAM. She notes this isn't a full memory budget.
- "Use This Model" then lists supported local apps and gives install or serving commands.
- Her concrete request — train Qwen2-VL on the LLaVA-InstructMix vision-language dataset — still required choices: which compute instance, estimated VRAM for the batch size, the validation split, and other training options. The agent resolves these, launches the job, and puts the model on the Hub.
- The same approach extends to object detectors and the Segment Anything Model, including handling different bounding-box formats.

### MCP discovery and the 30,000-paper OCR workflow · 15:00
- MCP exposes another route: search models, datasets, and Spaces (she calls Spaces the App Store of AI), including semantic search for Spaces.
- Jobs provide one-off compute that ends on success or failure, billed by running time; an assistant can discover a resource, call an app, or launch work through MCP tools.
- Demo: "Generate image of a baklava made of yarn" calls a remotely hosted Qwen-Image Space and returns the output. Enabling Dynamic Spaces (described as experimental) broadens discovery to MCP-compatible Spaces.
- The capstone: Hugging Face hosts paper pages, but not every paper has Markdown suitable for indexing, so a colleague's project OCR'd 30,000 papers using Codex, open OCR models, and Hugging Face Jobs through prompting.
- Model choice came from olmOCR-Bench, where Chandra OCR appears first in her displayed results; she notes a newly released skill can also recommend OCR models, including smaller ones for fine-tuning.
- The agent writes the processing script, estimates the instance, and calculates the job cost — connecting model choice to an operational plan.
- Because jobs rerun, storage matters: Hugging Face Buckets are mountable storage (compared with S3) that jobs can reuse. She calls them cheaper and faster without giving comparison conditions.

## Resources

- **Hermes Agent** (repository) — https://github.com/NousResearch/hermes-agent
- **Hugging Face Skills** (repository) — https://github.com/huggingface/skills
- **Agent Traces** (documentation) — https://huggingface.co/docs/hub/en/agent-traces
- **Hugging Face MCP Server** (documentation) — https://huggingface.co/docs/hub/en/agents-mcp
- **Local Agents with llama.cpp** (documentation) — https://huggingface.co/docs/hub/en/agents-local
- **How we OCR'ed 30,000 papers** — article by Niels Rogge. *(related)*
