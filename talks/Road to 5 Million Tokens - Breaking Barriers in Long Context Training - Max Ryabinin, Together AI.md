# Road to 5 Million Tokens: Breaking Barriers in Long Context Training

| Field | Details |
| --- | --- |
| **Speaker** | Max Ryabinin |
| **Affiliation** | Together AI |
| **Type** | Talk |
| **Date** | 2026-06-08 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=TUnPNY4E2fw) |
| **Tags** | `architecture` `rag-context-and-search` |

## Summary

Max Ryabinin, VP of R&D at Together AI, explains what it actually takes to train a model on a multi-million-token sequence. His team set a concrete target: fit a 3-million-token training run on a single node of eight H100 GPUs, then push to 5 million. The talk is a guided tour of where GPU memory goes and how to shrink it, one allocation at a time.

The core idea is that long context hits two different walls. Attention work grows quadratically, because every sequence position interacts with every other. Separately, the activations that must stay in memory grow linearly with sequence length. Neither wall falls to a single fix, so Ryabinin stacks techniques: parameter sharding, context parallelism, checkpointing, CPU offload, sequence tiling, and finally a new method called UPipe.

UPipe (Untied Ulysses) is the team's main contribution. It processes attention heads in smaller groups over time and reuses one working buffer instead of reserving space for every head at once. That saving is what makes 5-million-token training possible on one node without a large throughput penalty.

The practical takeaway is that long-context training is a memory-budgeting problem where the bottleneck keeps moving. Fix one allocation and another becomes the reason you run out. Ryabinin recommends profiling to find that next allocation instead of guessing.

## Key Takeaways

- **Two bottlenecks, not one.** Attention compute scales quadratically with sequence length; stored activations scale linearly. Fixing one does not fix the other.
- **Shard parameters first.** FSDP (fully sharded data parallelism) splits model weights across the node, but attention activations then dominate memory.
- **Context parallelism helps.** DeepSpeed Ulysses gives each GPU the full sequence for a subset of attention heads, enabling optimized attention kernels.
- **Shrink saved activations two ways.** Activation checkpointing recomputes values during the backward pass; CPU offloading moves block inputs to host memory and prefetches them.
- **Tile element-wise work.** MLP and loss computations can be chunked along the sequence because they do not mix positions.
- **UPipe reuses an attention buffer.** Processing heads in groups means a smaller allocation that gets reused, enabling 5M-token training.
- **5M tokens was measured on Llama3-8B**, and it shows training feasibility and speed, not reasoning or retrieval quality.
- **Profile to find the next wall.** The dominant allocation shifts as you optimize, so use tools such as PyTorch Profiler.

## Detailed Notes

### What does it take to train on a much longer context? · 00:15
- Ryabinin is VP of R&D at **Together AI**, an "AI-native cloud" offering GPU clusters, fine-tuning and reinforcement learning, and inference across 200+ models.
- Two forces drive demand for long context: **agents** that want to use a growing history, and **video** generation, where several frames per second fill the window fast and consistency needs events from minutes earlier.
- Even workloads that never reach millions of tokens can gain: understanding where memory goes may free capacity for speed.

### Quadratic work and growing live memory · 03:42
- Extending a transformer's sequence exposes **two separate constraints**.
- **Quadratic computation:** attention compares every position with every other, so compute grows with the square of the sequence.
- **Linear memory growth:** the activations that depend on sequence length must be stored, and long tensors still exhaust the GPU.
- Avoiding a stored quadratic attention matrix does not remove the linear-sized tensors elsewhere in training.

### Three million tokens on one node · 05:11
- The worked target is a **3-million-token** training sequence on one node with eight H100 GPUs.
- The talk calls the model **Llama 3B**, while the matching paper's Figure 2 identifies it as **Llama3-8B** — a distinction that matters when reading the memory budget.
- First fix: **FSDP** shards parameters across the eight GPUs instead of replicating them. Model memory drops, but training still runs out of memory because of attention activations.
- Second fix: **DeepSpeed Ulysses** (from Microsoft) parallelizes across the context using all-to-all communication.
- Each GPU ends up holding the full sequence for an assigned subset of heads; a reverse redistribution returns results to the original layout.
- This lets each device use optimized attention kernels such as **FlashAttention** (the measured run uses version 3), giving roughly an 8× memory reduction — still short of fitting on one node.

### Recompute, offload, then tile · 07:48
- **Activation checkpointing** stores less and recomputes the rest during the backward pass, reported as another roughly 8× cut in activation memory. Boundaries need care to avoid excessive recomputation.
- **CPU offloading** moves transformer-block inputs to host memory and prefetches them before backprop reaches that layer, overlapping transfers with useful work. Ryabinin credits **Unsloth** as the first implementation the team knew of.
- Displayed memory drops to near **37 GB**, but an MLP or loss step can still build an intermediate spanning all three million positions.
- **Sequence tiling** splits element-wise operations into chunks. MLPs and loss do not mix positions, so they can process slices; attention is left untouched.
- A PyTorch sketch combines tiling with checkpointing so expanded MLP intermediates are recomputed per tile.
- The full-sequence output and its concatenation still need storage — the saving is in the expanded intermediates.
- With tiling stacked on the earlier methods, the 3-million-token target finally fits.

### Reuse the attention buffer across head groups · 10:00
- Going past that target needs a closer look inside context parallelism, leading to **UPipe (Untied Ulysses)**.
- Key observation: **one group of heads already saturates the GPU's compute**, so reserving working space for every assigned head at once wastes memory without adding throughput.
- UPipe loops over head groups: recompute the inputs for one group, compute attention over the full sequence, store the partial result, then advance and **reuse the same buffers**.
- The allocation serves one group at a time, so it is much smaller. This chunks attention work along the **head** dimension, whereas tiling chunked along the sequence.
- The benefit depends on each smaller group still giving the GPU enough work; the team reports the memory saving with no significant throughput loss at smaller scales.

### What the five-million-token result measures · 11:46
- The team reports **competitive throughput** at both 8B and 32B scales, sometimes beating other implementations at shorter contexts.
- The **5-million-token** result applies specifically to **Llama3-8B**: 98.25 tokens/s/GPU on eight 80 GiB H100s.
- That run combines BF16, FlashAttention 3, FSDP, checkpoint offloading, Ulysses degree U=8, chunk count C=8, and 1.9 TiB of host RAM (offloading is unpinned at 5M).
- The paper's **Qwen3-32B** setup uses 16 H100s and still runs out of memory at 5 million tokens.
- These numbers demonstrate training feasibility and speed, **not** that a model reasons well over a 5-million-token input.
- Head-group size is a tuning knob:

| Heads computed together | Working memory | Throughput tendency |
| --- | --- | --- |
| Fewer | Lower | May be lower |
| More | Higher | Can be higher |

- The aim is not the smallest possible buffer but enough savings while keeping each chunk efficient.

### Find the allocation that becomes dominant next · 13:00
- Long-context training keeps **shifting the bottleneck**: shrink one allocation and a previously secondary one becomes the reason training fails.
- Ryabinin recommends a profiler such as **PyTorch Profiler** to surface unexpected costs; the paper covers profiling in depth.
- The complete stack matters — parameter sharding, context parallelism, checkpointing, offloading, tiling, and headwise buffer reuse each target different allocations.

### QKV means query, key, and value · 14:07
- A questioner asks whether "QKV" refers to quantization settings. Ryabinin corrects this: it stands for the transformer layer's **query, key, and value** matrices.
- Query–key interactions produce the pairwise attention computation; QKV is not a quantization parameter.
- Even a tensor with three million entries along a single axis is a large allocation, which is why buffer reuse alone cannot carry the whole training system.

## Resources

- [Untied Ulysses paper](https://arxiv.org/html/2602.21196v1) — memory-efficient context parallelism via headwise chunking.
- [DeepSpeed Ulysses](https://github.com/deepspeedai/DeepSpeed/blob/master/blogs/deepspeed-ulysses/README.md) — how all-to-all communication reshapes sequence partitions.
- [Unsloth Gradient Checkpointing](https://www.unsloth.ai/blog/long-context) — asynchronous activation offloading.
- [PyTorch Profiler tutorial](https://docs.pytorch.org/tutorials/recipes/recipes/profiler_recipe.html) — measuring where time and memory go.
