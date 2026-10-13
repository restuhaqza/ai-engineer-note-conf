# Running LLMs locally: capacity, throughput, and responsiveness on DGX Spark

| Field | Details |
| --- | --- |
| **Speaker** | Mozhgan Kabiri Chimeh |
| **Affiliation** | NVIDIA |
| **Type** | Talk |
| **Date** | 2026-04-10 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=c5-kx2bwoCk) |
| **Tags** | `rag-context-and-search` `infrastructure-and-deployment` `safety-and-governance` `evals` |

## Summary

Mozhgan Kabiri Chimeh, a Developer Relations Manager at NVIDIA, asks a practical question: how much of an LLM workflow can run on one machine under your desk? She frames the usual reasons work moves to the cloud — the model will not fit in memory, or the needed software stack is not available locally — and then runs hands-on benchmarks on DGX Spark to show what is realistic.

DGX Spark is a standalone system built on the GB10 Grace Blackwell Superchip, which combines CPU and GPU with unified memory. It has 128 GB of unified memory and FP4 support, which NVIDIA says is enough to work with models up to roughly 200 billion parameters. It also runs the same NVIDIA AI software stack used in data centers, so a workflow can move from desktop to cloud with few changes.

Her experiment serves Qwen models from 1.5B to 14B parameters with vLLM inside an NVIDIA-optimized container. Every run follows the same strict protocol so results stay verifiable: Docker isolation, three warm-up runs, background GPU logging, and a uniquely named output directory holding responses, metrics, and metadata.

The headline results are about speed. A 1.5B instruct model reached 61.73 tokens per second, while a 14B NVFP4 model (a 4-bit floating-point format) reached 20.19 tokens per second — nearly ten times the parameters at a still-usable rate. The same 14B model without that optimization dropped to 8.40 tokens per second. Kabiri Chimeh stresses these are measurements of one serving workflow on this hardware, not a universal ranking of model sizes.

## Key Takeaways

- **Two things push workloads off the desktop:** the model does not fit in memory, or the required software stack is not available locally.
- **DGX Spark pairs 128 GB of unified memory with FP4 support**, advertised as enough for models up to around 200 billion parameters.
- **The same stack runs locally and in the data center**, so prototypes can move to cloud with minimal changes.
- **Reproducibility is part of the result.** Each run gets Docker isolation, three warm-ups, GPU logging, and a saved artifact directory.
- **Time to first token (TTFT)** measures how long a user waits before a response starts; end-to-end latency measures the whole answer.
- **Quantization matters as much as hardware here.** 14B NVFP4 hit 20.19 tok/s versus 8.40 tok/s for the unoptimized 14B base.
- **14B NVFP4 was 3.4x faster to first token** than the base model (TTFT p50 of 0.07s versus 0.24s).
- **Memory capacity is not memory bandwidth.** Fitting a model and feeding it quickly are different problems.

## Detailed Notes

### Why run LLMs locally? · 00:01
- AI work increasingly demands more from developer machines, and two limits appear: running out of memory or lacking the right software stack.
- That often pushes everything to the cloud or a data center.
- As models move toward production, concerns like cost predictability, data residency, and deterministic latency take center stage.
- Iteration speed depends on shared infrastructure, and your runs compete with other workloads, which delays development.
- The guiding question: can some of that workflow move closer to where development happens?

### A desktop system with a portable stack · 01:39
- DGX Spark is a standalone AI system built on the **GB10 Grace Blackwell Superchip** (CPU + GPU with unified memory).
- It offers **128 GB of unified memory** and FP4 support, advertised for models up to around 200 billion parameters.
- The 200B figure is a capacity claim, not a measured throughput guarantee for every context length or concurrency setting.
- It runs the same NVIDIA AI software stack used in production, so work moves from desktop to data center or cloud with minimal changes.
- The point is not to replace the cloud, but to bring development closer to the developer.

### Make every benchmark run inspectable · 02:40
- Models are served with **vLLM** inside an NVIDIA-optimized container that matches the data-center environment.
- The automated harness runs Qwen models from 1.5B to 14B under one protocol.
- Protocol: isolate with **Docker**, run **three mandatory warm-ups**, then log GPU metrics every second in the background.
- Each execution gets a unique directory named from a precise timestamp and a sanitized model ID.
- The saved artifact holds full model endpoint responses, metrics, metadata, and benchmark results, so any number can be traced back to its run.

### Measuring when the response begins · 04:13
- **End-to-end latency** covers the full request; **time to first token** covers the wait before output starts.
- A fast first token makes an application feel responsive even while the rest of the answer is still being generated.
- The `stream_once` function consumes a streaming vLLM response and timestamps its first chunk.
- The timer records the first yielded chunk even if it contains no visible text, preserving the first-chunk proxy used here.
- Note: this client-side first-chunk timing differs from vLLM's server-side TTFT metric, so the two are not interchangeable.

### Completion speed versus model size and precision · 05:34
- The suite compares smaller instruct models with larger optimized variants under the same local setup.
- Reported results (speaker's DGX Spark / vLLM measurements):

| Model variant | Completion throughput |
| --- | ---: |
| 1.5B Instruct | 61.73 tokens/s |
| 14B NVFP4 | 20.19 tokens/s |
| 14B Base | 8.40 tokens/s |

- The 14B NVFP4 model has nearly ten times the parameters of the 1.5B model yet still produces 20.19 tok/s.
- **NVFP4** is NVIDIA's 4-bit floating-point quantization format; the speaker calls this a critical engineering sweet spot.
- The 14B base result shows parameter count alone cannot predict usable speed; on Blackwell, quantization choice matters as much as the hardware.
- These numbers are scoped to this checkpoint and setup, not a promise for other applications.

### Fitting a model is not the same as feeding it fast · 07:31
- Larger models generally need more computation before the first token, so TTFT rises with size.
- Reported TTFT p50: **0.07s** for 14B NVFP4 versus **0.24s** for 14B Base.
- That makes 14B NVFP4 about **3.4x faster to first token** than the unoptimized base model.
- **Memory capacity is not memory bandwidth:** capacity decides whether a model fits; bandwidth determines how fast data moves.
- A more compact representation (NVFP4) reduces the data burden, which the speaker frames as increasing "intelligence per byte."
- That is an interpretation of responsiveness, not a separate measurement of model quality.

### Keep iteration local, then scale · 09:23
- The speaker recommends DGX Spark for three uses: **steady-state workloads**, **privacy-sensitive data**, and **rapid prototyping**.
- You can build and fine-tune locally with the same software stack also used in DGX Cloud (these benchmarks measure inference, not fine-tuning).
- Start local, measure behavior, and iterate while the system is directly available.
- When the application needs more resources, carry the workflow to a data center or cloud.
- The DGX Spark playbook collection at `build.nvidia.com/spark` holds the software and workflows used here.

## Resources

- [DGX Spark playbooks](https://build.nvidia.com/spark) — practical guides for local inference, fine-tuning, and more.
- [Serve LLMs with vLLM](https://build.nvidia.com/spark/vllm) — current container serving instructions.
- [How NVFP4 represents model values](https://developer.nvidia.com/blog/introducing-nvfp4-for-efficient-and-accurate-low-precision-inference/) — the four-bit floating-point format.
- [DGX Spark hardware specifications](https://www.nvidia.com/en-gb/products/workstations/dgx-spark/) — GB10, unified memory, and memory bandwidth.
- [vLLM per-request metrics](https://docs.vllm.ai/en/latest/features/per_request_metrics/) — server-side TTFT definition.
