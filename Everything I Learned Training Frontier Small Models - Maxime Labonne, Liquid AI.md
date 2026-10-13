# Everything I Learned Training Frontier Small Models

| Field | Details |
| --- | --- |
| **Speaker** | Maxime Labonne |
| **Affiliation** | Liquid AI |
| **Type** | Talk |
| **Date** | 2026-04-29 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=fLUtUkqYHnQ) |
| **Tags** | `infrastructure-and-deployment` `reasoning-and-models` `rag-context-and-search` `safety-and-governance` `architecture` `coding-and-developer-tools` |

## Summary

Maxime Labonne, head of pre-training at Liquid AI, shares lessons from building
small language models (350 million to 24 billion parameters) meant to run on
phones, in cars and on other edge devices. His central claim is that small
models are not just scaled-down large models: they are memory-bound,
task-specific and latency-sensitive, so their architecture and training must
spend a limited budget on the capabilities the deployment actually needs.

He compares the embedding overhead of Gemma 3 270M and Qwen3.5-0.8B with
Liquid's own LFM2 architecture, which pairs gated short convolutions with
grouped-query attention and keeps embeddings to around 19% of parameters. He
then walks the LFM2.5 training recipe, including the surprise that a 350M model
was pre-trained on 28 trillion tokens — far beyond the classic compute-optimal
point.

Much of the talk tackles a distinctive failure mode: "doom loops," where a small
reasoning model repeats itself endlessly. Liquid attacks this during preference
training and reinforcement learning, cutting the loop rate from roughly 15% to
almost nothing. Labonne argues that many small-model limits can also be worked
around by the surrounding system, such as web search and Python-based recursive
environments.

## Key Takeaways

- **On-device models face three constraints:** memory (which caps size and knowledge), task specificity (narrow beats general-purpose), and latency (fast throughput matters).
- **Embeddings can dominate a tiny model:** they make up 63% of Gemma 3 270M and 29% of Qwen3.5-0.8B, leaving few "effective" parameters for reasoning.
- **LFM2 keeps embeddings near 19%** and picks operators by profiling them on the actual target hardware, not just in theory.
- **More pre-training still helps at tiny scale.** Liquid trained a 350M model on 28 trillion tokens, beyond Chinchilla's compute-optimal estimate.
- **Target capabilities, not averages.** LFM2.5-350M was optimized for data extraction and tool use, not for leading code or math scores.
- **Doom loops need targeted fixes.** A tiny model + reasoning mode + a very hard task is the worst case, and the rate can exceed 50% in some models.
- **Fix loops in preference training:** sample diverse rollouts plus one greedy (loop-prone) rollout, then let an LLM jury pick chosen/rejected pairs.
- **Small models shine as agents.** Give them web search, and use Python-based recursive environments to work around weak long context.

## Detailed Notes

### What fits on a phone or in a car? · 00:15
- Labonne is Liquid AI's head of pre-training; the models span 350M–24B parameters across text, vision and audio, and are on Hugging Face.
- On-device deployment imposes three connected constraints:
  - **Memory:** hardware caps model size, which caps how much knowledge it can store.
  - **Task specificity:** limited capacity makes specialization valuable — summarization or tool use beats general chat.
  - **Latency:** interactive workloads need fast inference and enough throughput.
- **Core lesson:** small models are not just scaled-down large models; they have unique challenges.

### Where the parameter budget goes · 02:28
- Small members of large families are already hybrid: Gemma 3 270M pairs sliding-window attention with grouped-query attention (GQA); Qwen3.5-0.8B pairs gated DeltaNet with gated attention.
- Labonne's concern is the embedding layer, not the sequence blocks: it holds 63% of Gemma 3 270M's parameters and 29% of Qwen3.5-0.8B's.
- Everything else is the "effective" parameters available for reasoning and knowledge.
- He traces the large embeddings to distillation from teachers with big vocabularies — the student inherits a big embedding matrix even as the rest shrinks.
- The opportunity: spend the same memory on more non-embedding network instead of copying the teacher's proportions.

### Choose operators on the target hardware · 03:59
- LFM2 is also hybrid: gated short convolutions plus GQA, with embeddings at about 19%, leaving more budget for the network.
- The team picked its operators by implementing candidates and profiling them on real target devices, not by theory alone.
- The gated short-convolution block won on latency — cheaper in the presented comparison than sliding-window attention, gated DeltaNet, gated linear attention and GQA.
- CPU comparison (AMD Ryzen AI Max+ 395 and a Samsung Galaxy S25 Ultra; `llama.cpp`, 4-bit, 2K input) showed faster inference and lower memory, though the compared models differ in size.
- GPU comparison (H100, BF16, SGLang) showed strong aggregate throughput at high concurrency — serving throughput, not single-request latency.

### A 350M model trained on 28 trillion tokens · 06:07
- The LFM2.5 recipe is conventional: pre- and mid-training, supervised fine-tuning (SFT), preference alignment, then reinforcement learning (RL).
- The surprise is scale: a 350M-parameter model pre-trained on 28 trillion tokens (a training-token count, not 28 trillion unique tokens).
- Chinchilla's compute-optimal study would suggest far fewer, but performance kept improving as pre-training tokens grew.
- Labonne cites Roberts et al. on test-time scaling; by those newer laws, even 28T tokens is below the proposed optimum for this model.
- The comparison is post-trained LFM2-350M versus LFM2.5-350M, so it does not isolate extra pre-training. Gains span knowledge (GPQA Diamond), instruction following (IFBench), data extraction (CaseReportBench) and tool use (BFCL, tau2-bench).
- Priorities were data extraction and tool use — code and math were not the target. **Choose capabilities instead of aiming to be average at everything.**

### Adapt the training stages to limited capacity · 08:31
- The stages match large models; how you use them is what differs.
- **SFT:** keep it narrow. Taking a model and fine-tuning it to call one particular function is an excellent fit.
- **Preference alignment:** Liquid uses an on-policy, length-normalized form of direct preference optimization (DPO) for broad quality gains that go beyond benchmark scores.
- **RL:** effective even at tiny scale, but with a narrow learning signal; use many environments and tasks to generalize.
- Small models are sensitive to cold start, so the SFT mixture should include examples resembling the later RL tasks.
- If RL will not train: check the SFT mixture, consider whether the task is too complex, add cold-start examples, rerun SFT, then retry RL.

### Doom loops: fix them with preference training and RL · 10:41
- A **doom loop** is endless repetition of a sequence instead of finishing. Risk rises when a tiny model, reasoning mode and a too-hard task combine.
- Intervention 1 is in preference alignment. Start from roughly one million prompts; the policy model generates five temperature-sampled rollouts (for diversity) plus one greedy rollout at temperature zero (expected to loop). An LLM jury scores all six; the highest becomes `chosen` and the lowest `rejected`. A looping answer tends to land on the rejected side.
- The selection step does not itself detect repetition — it relies on the jury scoring failed answers worse. A tied best and worst gives no signal.
- Intervention 2 is RL with verifiable rewards plus an n-gram repetition penalty. A math rollout that never emits a final answer earns no correctness reward.
- Reported result for LFM2.5-1.2B-Thinking on hard tasks: roughly 15–16% doom loops after mid-training; SFT barely moves it; DPO cuts it substantially; RL makes it almost nonexistent. The full prompt set and loop detector are not specified.
- He reports more than 50% doom loops for Qwen3.5-0.8B in reasoning mode on hard tasks — evidence that some tiny models behave like scaled-down big ones.

### Put knowledge and context in tools · 15:28
- Memory limits mean low stored knowledge and more hallucination; web search turns recall into lookup.
- From his experience, tiny models are very good at agentic tasks. The critical ability is reasoning well enough to use tools reliably: decide what to look up, interpret the result, and continue.
- Long context is also weak, but a recursive language-model environment lets the model use Python to handle input outside its immediate window — a route around the limit, not a measured Liquid implementation.
- Edge-model design is interesting scientifically and in production: limitations can be addressed by the surrounding system, not only the weights.
- At the time of the talk, Liquid was developing LFM3.

### When to deploy locally — and what distillation may miss · 17:59
- Three reasons to use a local small model: **offline operation** (a car cannot count on connectivity), **latency** (local wins for time-sensitive work), and **privacy** (finance, healthcare and other regulated settings).
- Asked about his personal workflow, Labonne separates making these models from using them: he builds them for other people.
- On whether a large model's anti-loop RL behavior can be distilled into a small one, he says experiments are needed and suspects not, because distillation resembles SFT — the stage that barely reduced looping.
- The distillation formulation may matter, but he offers no tested recipe and expects additional training batches to still be needed.

## Resources

- [LFM2.5-350M model and usage examples](https://huggingface.co/LiquidAI/LFM2.5-350M)
- [Test-Time Scaling Makes Overtraining Compute-Optimal](https://arxiv.org/abs/2604.01411)
- [Chinchilla: Training Compute-Optimal Large Language Models](https://arxiv.org/abs/2203.15556)
- [Google's introduction to Gemma 3 270M](https://developers.googleblog.com/introducing-gemma-3-270m/)
- [Qwen3.5-0.8B model card](https://huggingface.co/Qwen/Qwen3.5-0.8B)
- [LFM2.5-1.2B-Thinking release](https://www.liquid.ai/blog/lfm2-5-1-2b-thinking-on-device-reasoning-under-1gb)
- [Recursive Language Models](https://arxiv.org/abs/2512.24601)
