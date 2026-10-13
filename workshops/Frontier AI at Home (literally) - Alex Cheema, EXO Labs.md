# Frontier AI at Home (literally)

| Field | Details |
| --- | --- |
| **Speaker** | Alex Cheema |
| **Affiliation** | EXO Labs |
| **Type** | Workshop |
| **Date** | 2026-05-26 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=ESbWpPT_9-o) |
| **Tags** | `infrastructure-and-deployment` `industry-applications` `rag-context-and-search` `safety-and-governance` `agent-engineering` `evals` |

## Summary

Alex Cheema of EXO Labs runs a hands-on workshop on running frontier models on
local hardware instead of in someone else's data center. His opening concern is
ownership: if an AI becomes an extension of your working memory, what happens
when a provider cuts off access? A cybersecurity friend was locked out of
several model APIs while doing defensive penetration testing.

Most of the workshop is engineering. Inference splits into **prefill** (process
the prompt, build the KV cache) and **decode** (generate tokens one at a time).
At a single user's low batch sizes, decode is limited by memory bandwidth and
energy, not raw compute. He argues the useful unit of optimization is the whole
running system — kernels, interconnects and even the harness.

His central claim is that different devices solve different problems and should
be combined: a Mac pools memory, an RTX has more bandwidth and compute, and
NVIDIA's Spark favors prefill. He closes with live demos on four Mac Studios
connected by Thunderbolt 5, running a large GLM-5.1. The enabling change is
**RDMA** (a low-latency remote-memory transport), which reportedly cut
inter-machine communication from ~0.3 ms to single-digit microseconds.

## Key Takeaways

- **Ownership is the motivation:** local models keep access and data under your
  control.
- **Prefill is compute-bound; decode is memory-bandwidth-bound** at small batch
  sizes.
- **Three limits decide viability:** capacity, bandwidth and energy per byte.
- **The whole stack matters** — kernels, interconnects and the harness.
- **Different devices, different strengths:** combine a memory-heavy Mac with a
  bandwidth-heavy GPU or Spark.
- **Tensor parallelism needs fast communication** — RDMA moved that under a
  millisecond.
- **Local inference can generate its own batch** via multi-agent and test-time
  search.
- **Be skeptical of speed claims** — extreme quantization can hurt quality.

## Detailed Notes

### Ownership: why run frontier AI at home · 01:05
- EXO Labs works across models, software and hardware to cut the cost of local
  frontier AI.
- Most people still use data-center models; EXO argues for an open alternative
  you own.
- He cites Karpathy's line, roughly "not your weights, not your brain."
- As systems like OpenClaw act on a user's behalf, ownership includes continuity
  of access, control of data and the ability to choose what the system can do.
- The locked-out friend shows how contingent useful work can be on provider
  decisions.

### The hardware lottery and whole-stack optimization · 05:17
- After training, the question is how to execute the model; million-dollar
  hardware is a major barrier.
- Sara Hooker's **hardware lottery**: available hardware shapes which ideas get
  explored. The stack optimized for training, not inference.
- Local Qwen 3.5 on Apple Silicon ran ~50% slower than expected.
  - Cause: unnecessary kernels launched separately, each adding overhead.
  - At 150 tokens/second, each token has under 10 ms, so a few ms of launch cost
    is a large share.
  - Fusing those kernels gave a reported **30% improvement**.
- The same logic applies to orchestration, interconnects and the harness —
  Claude Code and OpenCode can differ with the same model.
- **The useful unit of optimization is the whole running system.**

### Prefill, decode, and the three limits · 10:33
- Personal inference lacks the large batches a cloud provider enjoys.

  | Phase | Work performed | Typical limit here |
  | --- | --- | --- |
  | **Prefill** | Process the prompt, build the KV cache | Compute |
  | **Decode** | Generate tokens one at a time | Memory bandwidth at low batch |

- The **KV cache** holds attention keys and values that decode reuses.
- A good harness avoids re-prefilling an unchanged prefix; Claude Code's
  `/context` command shows how much of the prompt is stable.
- Three questions decide practicality:
  - **Capacity:** does the model fit? Loading weights from disk is very slow.
  - **Bandwidth:** how fast do weights and KV cache reach the GPU?
  - **Energy per byte:** how much energy does moving that data cost?
- He pairs a ~10–15 Wh phone battery with ~10–15 W of inference to illustrate
  roughly an hour of runtime; his team's phone got too hot to hold.
- He prefers **intelligence per joule** (task quality per energy used), distinct
  from the power-based "intelligence per watt" paper.

### The appliance question: price, saturation, specialized chips · 18:50
- An audience member asks whether people will spend ~$5,000 on an inference box
  like a fridge. Cheema answers through the trajectory of useful inference.
- GLM-5.1's raw weights need roughly 1.5 TB — more than one consumer machine; he
  estimates ~$40,000 of 512 GB Mac hardware for that class, at roughly 20
  tokens/second.
- He sees about a **100× price/performance** opportunity across the whole stack,
  potentially a near-frontier ~$5,000 appliance within ~18 months — a thesis,
  not a shipped product. That needs **co-design**.
- He proposes an **S-curve per use case**: more intelligence helps at first,
  then brings diminishing utility. Transcription needs low latency, while hard
  scientific work keeps absorbing compute.
- **Specialized chips** need stable targets: general GPUs are flexible, Cerebras
  and Groq sit between, and Taalas is model-specific. The test is repaying chip
  development before the model goes obsolete.
- EXO evaluates rather than builds hardware, but imagines many task-specialized
  models run by a few efficient chips.

### Mac vs. GPU vs. Spark: split the model across devices · 37:52
- An audience member notes a large Mac model runs hot and drains battery; fitting
  weights is not the same as sustaining a useful workload. Cheema agrees.

  | Device | Memory capacity | Memory bandwidth |
  | --- | --- | --- |
  | RTX 5090 | 32 GB GDDR7 VRAM | ~1.5–2 TB/s |
  | Demonstrated Mac Studio | 512 GB unified memory | ~800 GB/s |

- The Mac has far more capacity but much less bandwidth and compute, so place
  different model parts on different devices.
- An ASUS NVIDIA **Spark** (~$4,000) has more compute and suits prefill; the
  Mac's bandwidth suits decode. He corrects a reversal: **Spark prefill, Mac
  decode.**
- A directly connected RTX delivering a ~3× speedup on large models was
  presented as work underway.
- Architecture matters: a quantized **dense** Gemma 4 benefits from a small fast
  pool, while a **mixture-of-experts (MoE)** model can exploit the Mac's memory.

### Clusters, batching, and local concurrency · 47:19
- EXO's workflow: a background app per device discovers peers and picks a model
  distribution.
- Each node exposes an HTTP endpoint; Cheema uses Tailscale (not bundled) to
  reach it, keeping a phone as the interface while inference runs at home.
- Cloud providers batch many users; a single user normally cannot, so local must
  generate its own concurrency: multi-agent requests (he cites Grok 4.20) turn
  one request into several calls; test-time search like best-of-N generates
  candidates and verifies one; test-time training updates weights during use.
- On idle capacity: an owned setup has no per-token bill, so an always-on
  assistant might consume much of it; he prefers selling tokens or completed
  tasks over renting hardware.

### Live demo: RDMA, GLM-5.1, and adding a laptop · 1:07:37
- Four Mac Studios run EXO, fully connected over Thunderbolt 5.
- The key recent work is **RDMA**: inter-Mac latency was ~0.3 ms, too slow to
  split work usefully.
- **Tensor parallelism** splits layers across machines and synchronizes
  repeatedly — a 60-layer model needs 120 syncs per token.
  - At 0.3 ms each that is ~36 ms per token, enough to erase any benefit.
  - RDMA reportedly cut latency ~100× to single-digit microseconds, bringing
    overhead under ~1 ms per token.
- They run **GLM-5.1** across the four machines as an "instance" (one running
  configuration). A 4-bit download of nearly 400 GB uses ~112 GB per machine; it
  needed only weight conversion to MLX because GLM-5's architecture was already
  supported (Gemma 4 was harder).
- A smaller Qwen 3.5 instance runs on two nodes (~77 tokens/second).
- An audience MacBook joins over Thunderbolt after Wi-Fi discovery fails. EXO
  uses **event sourcing** — each machine keeps an append-only log and the laptop
  replays history — which suits devices that sleep or leave mid-request. Setup
  needs Thunderbolt 5, RDMA via macOS Recovery, topology discovery and matching
  OS versions.
- Its MLX/LM Studio downloads are invisible (they live outside `exo/models`), so
  a small Qwen 0.6B is downloaded. A large checkpoint can be a terabyte-scale
  download, and each machine currently keeps essentially the full weights.
- A **heterogeneous** run puts prefill on the Spark and decode on the MacBook;
  the KV cache must stream over 10 Gb Ethernet while compute proceeds, or the
  stages fall back to running in sequence.

### Measuring quality, not just speed · 1:32:24
- Local-inference advice currently lives on Reddit and Twitter, where real
  experiments mix with misunderstood results.
- He is skeptical of extreme quantization: a one-bit large model may be worse
  than a smaller model kept at higher precision, and "Kimi on a MacBook" claims
  can hide reduced expert activation.
- EXO's planned open benchmark site (announced for roughly the next month) will
  collect thousands of results across hardware, models, quantization and pruning,
  measuring tokens/second, prefill time, task quality and energy together.
  A budget selector would show the **Pareto frontier**.
- On **autoresearch** (Karpathy's loop, via Simon Willison): he values it but
  insists on the scientific method — a reasoned hypothesis, an experiment that
  also checks quality, then iterate. An open-ended "make it faster" loop becomes
  a slot machine.
- He is skeptical of disk-backed inference: a MacBook's single SSD is too slow,
  though a large SSD array might interest if memory prices change.
- Final Spark–Mac comparison: ~**2× faster** end-to-end for a large prompt, and
  a single-paper run of ~7 seconds on the MacBook alone versus 4.8 seconds across
  both (~1.46×). The benefit grows as prefill grows.

## Resources

- [EXO](https://exolabs.net/)
- [EXO source and RDMA setup](https://github.com/exo-explore/exo)
- [The Hardware Lottery](https://arxiv.org/abs/2009.06489)
- [Intelligence per Watt](https://arxiv.org/abs/2511.07885)
- [GLM-5.1 official model card](https://huggingface.co/zai-org/GLM-5.1)
- [Search and Learn (inference-time search recipes)](https://huggingface.co/spaces/HuggingFaceH4/blogpost-scaling-test-time-compute)
