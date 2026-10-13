# Scaling the Next Paradigm of Heterogeneous Intelligence

| Field | Details |
| --- | --- |
| **Speaker** | Adrian Bertagnoli |
| **Affiliation** | Callosum |
| **Type** | Talk |
| **Date** | 2026-05-24 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=WRBNDpUhsJQ) |
| **Tags** | `infrastructure-and-deployment` `evals` `coding-and-developer-tools` `vision-and-video` |

## Summary

Adrian Bertagnoli, a founding engineer at Callosum, argues that AI systems should stop scaling as one model on a fleet of identical chips. Instead, he says, the work should spread across different models, workflows, and hardware — what he calls heterogeneous intelligence.

His starting point is that real problems are multi-step and open-ended. Their subproblems need different kinds of capability, so pouring more of one kind of intelligence into the whole task is inefficient. He formalizes this with a "principle of maximum heterogeneity" and argues the right move is to match the mix of skills to the actual workload.

The talk gives two worked examples. The first is recursive language models, which keep a long context outside the prompt and let a coding agent search it with Python before reasoning over selected pieces. The second is visual web navigation, where cheap models handle visual preparation (such as zooming) and stronger models do the reasoning, improving both speed and cost. He closes with a vision of compute where workloads map onto whichever chips suit them.

## Key Takeaways

- **Homogeneous scaling is fading.** One big model on identical chips is being replaced by specialization at three layers: mixture-of-experts architectures, multi-agent workflows, and split prefill/decode hardware.
- **Match the mix to the task.** Problems decompose into subproblems with different capability needs, so diversity should fit the workload rather than be maximized blindly.
- **Long context is an environment, not a prompt.** Recursive language models store context in a file, let a coding agent search it with Python, and reason only over the relevant pieces.
- **Not all long-context tasks are equal.** Retrieval of a fixed fact scales fine, but aggregation over many rows degrades at roughly 30–60% context occupancy.
- **Route each piece to the right model and chip.** Callosum reports large cost and latency gains by mapping subcontexts to different models on different hardware.
- **Split visual from textual reasoning.** In visual web navigation, offloading simple visual subtasks (like zooming) to smaller models beats single-model baselines.
- **Decomposition exposes choices.** Once a task is broken up, an orchestrator must decide which model and hardware serve each part.
- **Mind the caveats.** The benchmark numbers are Callosum-reported; the exact evaluation splits and baselines are not fully specified.

## Detailed Notes

### From one model on identical chips to coordinated specialists · 00:16
- Bertagnoli opens with the prevailing paradigm he calls homogeneous intelligence: scaling single models on a fleet of identical chips.
- Neural scaling laws — more data and more parameters yield better models — made that era productive, but he says it is rooted in training, while the industry is moving toward inference.
- He sees mild heterogeneity already appearing at three layers: mixture-of-experts replacing dense models (architecture), multi-agent systems replacing single calls (workflow), and prefill/decode disaggregation on hardware.
- A stronger level assigns different models to different GPUs and combines architectures such as state-space models and diffusion models on hardware that suits their computation.
- The endpoint he proposes is co-evolution: models, software, and hardware designed as one vertically integrated system.

### Match collective capability to the problem's demand · 03:24
- Real-world problems are complex, multi-step, and open-ended, so they split into subproblems that need different types of intelligence.
- Scaling one kind of intelligence to cover all of them is inefficient. Solving them, he argues, needs models of different sizes and architectures working together over long horizons.
- There is also an infrastructure problem: new silicon needs an interface to contribute to the existing compute stack; making an accelerator available does not by itself show agents how to use it.
- The "principle of maximum heterogeneity" models this as a distributed production function — a profile of what a group of agents can supply, based on their skills and how they communicate.
- That profile can fit one problem's demand but serve another poorly.
- In his picture, replicating a specialist only raises one peak, while replicating broad generalists spreads capacity thin; neither fills the gaps.
- He says the result holds across neuroscience, economics, and ecology, and is bounded by skill distributions, communication, resources, and demand. The design rule: match diversity to the workload and its communication topology.

### Three places to optimize · 05:56
- Orchestration spans three connected decisions.
- **Hardware:** choose where an agent runs based on its computational demands.
- **Agents:** choose the models and capabilities that do the work.
- **Workflows:** decide how agents interact and how their work is composed.
- He develops two examples: heterogeneous recursion at the workflow layer, and a mixture of multimodal models for visual web navigation at the agent layer.
- In both cases the opportunity comes from decomposing a task before deciding which resources should execute each part.

### Treat long context as an environment · 06:52
- A large context window does not guarantee reliable reasoning over everything in it; he calls this context rot.
- The clearest distinction is by information demand. Finding one needle in a haystack is O(1): the input grows but the answer needs a fixed amount. Adding values across rows is O(N): more of the input must participate in the result.
- Constant-demand retrieval can stay effective across the whole window, he says, while linear or quadratic demands can degrade at roughly 30–60% occupancy. He gives no universal cutoff.
- Recursive language models change how the agent reaches the input: keep the long context outside the prompt in an inspectable environment.
- A coding agent uses a Python REPL (an interactive Python prompt) to search it with keywords, regular expressions, or other programmatic operations.
- It extracts relevant subcontexts and passes them to recursive calls; each call answers its question or delegates again.
- His example keeps a support log in a file while Python pulls out complete records mentioning refunds, then batches those records as subcontexts. He notes the filter must match the question — a keyword search misses records that phrase things differently.

### Route recursive work across models and chips · 08:44
- Once a workflow exposes smaller subcontexts, orchestration can choose how to process each one.
- Callosum maps subcontexts to different models and different chips, aiming to keep frontier-level reasoning quality while cutting cost and latency.
- The distinguishing step is choosing model and hardware together, not recursion alone.
- On the OOLONG benchmark (a long-context reasoning and aggregation test), he names GPT-5 and then GPT-5.2 as the baseline.
- For that baseline he reports about 2,000 seconds to run the benchmark and about $3.75 for one task. He cautions the spoken units differ, so they should not be combined into a throughput estimate.
- With Cerebras, he reports the system is 7× cheaper and 5× faster. With SambaNova, 12× cheaper and 3× faster — pushing price lower at the cost of some latency.
- These are Callosum-reported results; the exact evaluation split, timing aggregation, and accuracy comparison are not given in the talk.

### Separate visual reasoning from textual reasoning · 10:20
- Visual web navigation is another kind of heterogeneity: a browser agent must read images, reason about text, and choose actions.
- Callosum combines open and closed video action language models. Bertagnoli reports 18% over GPT-5.2 and 25% over Gemini 2.5 on the benchmark he calls Video Web Arena.
- The mechanism is decomposition: visual reasoning and textual reasoning become separate components that can use different models.
- He describes a shift in the Pareto frontier (the cost-versus-performance tradeoff), where heterogeneous mixtures beat single-model configurations including Kimi K2.5 and GPT-5.2.
- Pairing Qwen3-VL-8B-Instruct with Kimi K2.5: reported 1.3× faster than Kimi K2.5 alone and 18× cheaper than GPT-5.2 alone (note the two rows use different baselines).
- Pairing Qwen3-VL-8B-Instruct with GPT-5.2: reported 3× faster and 3.7× cheaper than GPT-5.2 alone.
- One source of savings is as simple as zooming. Preparing a closer view does not need the same capability as interpreting it, so zooming and other view prep go to less capable models.
- For those visual subtasks alone he reports 11× faster and 43× cheaper than using ChatGPT; those large subtask ratios contribute to, but do not equal, the whole-workflow gains.

### Build infrastructure around heterogeneous workloads · 12:38
- Bertagnoli frames this as a third era of compute. The CPU era made computation faster; the GPU era made it massively parallel, with NVIDIA dominating.
- The proposed third era maps multi-agent workloads onto different chips according to what each step needs.
- Scaling then depends not only on a device's capacity but on how well the system assigns work across devices.
- Callosum is working with ARIA (a UK research funder), reporting a rounded £3 million grant toward what he calls the UK's first heterogeneous co-located cluster.
- He credits homogeneous scaling for enabling all of this, and ends with an infrastructure builder's optimism: "This is the worst our infrastructure will ever be."

### How does the system choose a model? · 14:01
- An audience member asks how the system decides which task runs on the faster, cheaper model — is it hard-coded?
- Bertagnoli says the first implementation used bespoke mappings from simple subtasks to simple models.
- Callosum later added an automation layer that detects task complexity and predicts the best-suited model and hardware.
- He does not describe how that predictor works or how it is validated.
- The routing decision remains central: decomposition exposes the choices, and the orchestrator must turn them into assignments that keep quality while cutting cost.

## Resources

- **Welcome, Heterogeneous Intelligence** — Callosum's technical account of the work. *(related)* — https://www.callosum.com/blog/welcome-heterogeneous-intelligence
- **The Principle of Maximum Heterogeneity** — A distributed production model connecting agent skills to demand. *(mentioned)* — https://www.callosum.com/blog/the-principle-of-maximum-heterogeneity
- **Recursive Language Models: Original Write-up** — The authors' October 2025 introduction to inspecting context outside the prompt. *(related)* — https://alexzhang13.github.io/blog/2025/rlm/
- **Recursive Language Models (paper)** — Research on recursive inference over context held outside the model. *(mentioned)* — https://arxiv.org/abs/2512.24601
- **OOLONG: Long-Context Reasoning and Aggregation** — A benchmark testing analysis over many text fragments. *(mentioned)* — https://arxiv.org/abs/2511.02817
- **We Are Scaling Heterogeneous Compute** — Callosum's announcement of hardware partnerships and ARIA support. *(related)* — https://www.callosum.com/blog/we-are-scaling-heterogeneous-compute
