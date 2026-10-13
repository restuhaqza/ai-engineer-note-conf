# Your Coding Agent Should Do AI Systems Engineering

| Field | Details |
| --- | --- |
| **Speaker** | Ben Burtenshaw |
| **Affiliation** | Hugging Face |
| **Type** | Talk |
| **Date** | 2026-05-21 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=JomVvNDjGb8) |
| **Tags** | `evals` `coding-and-developer-tools` `rag-context-and-search` `data-and-model-adaptation` |

## Summary

Coding agents are now ordinary tools for everyday software work. Ben Burtenshaw asks what happens if we point them at harder problems instead — GPU kernels, model training, and research — rather than only application code. His answer is that agents can handle this work, but only when the results live in standard, reusable repositories.

He describes three levels of rising autonomy. First, a person writes a CUDA kernel interactively with an agent. Second, the agent fine-tunes a language model from a single prompt. Third, a team of agents runs an automated research lab that proposes, implements, and tracks experiments.

The thread tying the three levels together is measurability. Writing a kernel or training a model gives the agent an outcome it can check, so the work can be verified and improved. Burtenshaw argues this only holds up when the underlying building blocks — kernels, metrics data, compute — stay open and inspectable instead of hidden behind abstract APIs.

The practical payoff is less time setting up each task and more time testing ambitious ideas.

## Key Takeaways

- **Three autonomy levels.** Interactive kernel writing, prompt-driven fine-tuning, and a multi-agent research lab.
- **Kernels are a solvable target.** Agents can produce valid, optimized GPU kernels; distribution is the harder problem.
- **Memory, not math, often dominates.** Keeping GPUs busy means doing more computation per memory read (called arithmetic intensity).
- **Kernels ship like models.** Hugging Face Kernels turns a kernel into a Hub repository with a compatibility config.
- **Skills are file-based context.** Reference examples and scripts move an agent from zero-shot to few-shot.
- **AutoLab splits research into four roles.** Researcher, planner, worker, and reporter replace one agent doing everything.
- **Open primitives matter.** A dashboard helps, but raw metrics data should stay reachable so agents can build their own views.
- **Verifiable experiments are the enabler.** If the result can be measured, agents can run the loop.

## Detailed Notes

### Three levels of agent autonomy · 00:24
- Agents are accepted in everyday coding, but that success prompts a question: what harder problems should we hand them next?
- Burtenshaw proposes moving "closer to the silicon" — toward systems and machine-learning engineering.
- The levels are framed as three "bosses": interactive kernel writing, prompt-driven fine-tuning, and a multi-agent lab.
- Each level gives the agent more responsibility while keeping a concrete task whose outcome can be checked.
- A second, less exciting requirement: results must land in standard repositories, often on the Hugging Face Hub.

### Writing a CUDA kernel and the GPU memory wall · 01:58
- Custom kernels once looked unreachable for agents because they need specialized languages and real hardware to test.
- Hackathons (GPU MODE, a recent AMD event) and the KernelBench paper show agents can write valid, optimized kernels.
- A **kernel** is the actual GPU work behind a model operation; a custom one exploits specific hardware for a math operation.
- Efficiency splits into three costs: compute (the math), memory (moving tensors), and overhead (Python, PyTorch dispatch).
- The surprise: memory is usually the bottleneck, not compute. The GPU often waits for tensors to arrive.
- On an H100, example figures are roughly one petaflop/s of compute versus about three terabytes/s of bandwidth.
- **FlashAttention** is the poster child for raising arithmetic intensity — do more useful math per read and write.

### Making kernels reusable with Hub repos and skills · 05:14
- Hugging Face **Kernels** is a distribution layer: a TOML config declares supported hardware and required software versions.
- The kernel becomes a Hub repository, so publishing a kernel mirrors publishing a model.
- Consumers can read the compatibility info to check whether a kernel fits their GPU or laptop.
- **Skills** are file-based context: examples and instructions an agent can open, close, version, and source-control.
- Loading a skill usually moves a task from zero-shot to few-shot, because the agent gets concrete examples.
- Hugging Face keeps skills inside projects so maintainers own them, with a separate repo for more experimental ones.
- The kernel skill includes testing and benchmarking scripts, linking generated code to evidence.

### Measure the kernel, then evaluate the skill · 07:52
- Reported result: an agent-generated kernel for **Qwen3-8B** on H100, with a 94% speed-up.
- The companion benchmark scopes that number: roughly 1.94× average speedup on isolated RMSNorm operations, BFloat16 on an H100 80GB HBM3, across eight tensor shapes.
- It is not a full-model inference improvement, and Burtenshaw says it is not a state-of-the-art result.
- The real opportunity is the **compatibility matrix**: models tuned for one hardware generation can leave speed on the table on another.
- Agents make it practical to hunt that low-hanging fruit, especially when your cheap cloud GPU is not the model's ideal target.
- **UPskill** generates skills plus evals, then lets you compare models on the same skill.
- In his comparison, GPT-OSS was slightly less accurate at similar token use, while Kimi and Haiku were more accurate with fewer tokens.

### Fine-tuning a model from a prompt · 09:26
- The second level: give the agent a larger task — fine-tune a model from a prompt.
- Spoken example: ask for Qwen3-6B to be fine-tuned on a chain-of-thought dataset.
- That model name was the example as spoken, not a verified checkpoint; improving chain-of-thought is the goal, not a measured result.
- HF CLI skills connect the workflow to the Hub, including hosted GPU execution.
- An Unsloth-based route is presented as a cheaper alternative with optimized training implementations.
- The shift from level one is that skills now guide a whole training workflow instead of an interactive kernel session.

### AutoLab: turning a research loop into a team · 10:21
- **AutoLab** starts from Andrej Karpathy's autoresearch, part of the nanoGPT and nanochat lineage.
- In that setup, one agent edits the training script and runs experiments, judged by a bits-per-byte metric (lower is better).
- Burtenshaw found the single-agent loop limiting and split the work into four roles.
- **Researcher:** finds papers (via HF Papers, which has a CLI, or arXiv) and turns ideas into hypotheses.
- **Planner:** keeps the queue of jobs. **Workers:** implement hypotheses as training changes. **Reporter:** monitors jobs and maintains a dashboard.

### Git, compute, and metrics for experiments · 12:09
- The lab runs inside a Git repository: an original training script, an evolving script, and branches per experiment.
- A data structure on `main` holds the scores. The demo uses OpenCode, but the repo also has Codex and Claude versions.
- Workers launch **HF Jobs** on the needed hardware and submit patches back to the repo.
- The reporter uses **Trackio**, an open-source dashboard whose data layer is Parquet — readable independently of the dashboard.
- Because the data is open, an agent can skip the dashboard and build its own view, such as a Gantt chart of experiments.
- The demo prompt bounds the work: run one pass, propose up to two fresh single-change experiments, use a reviewer to reject duplicates or stale ideas, and keep storage in one HF bucket.
- The planner and reviewer receive templates with current state, job history, and available hyperparameters.
- Burtenshaw calls the reviewer's tabular output verbose — the shared comparison context matters more than the volume.

### Verifiable experiments and open primitives · 16:45
- **The key condition is a verifiable experiment.** Training a model or writing a kernel gives an outcome to measure and compare.
- Agents run in parallel for hours, so monitoring must show more than a final score: events, warnings, and notifications.
- On the compute side, jobs are visible in Hugging Face, and labels let you sort and inspect what agents are running.
- Reading the underlying metrics data lets you plot a Gantt chart linking each agent to when it ran and what it scored.
- Takeaway one: agents work well with **open primitives**, and abstract APIs can become a ceiling if you cannot see behind them.
- Takeaway two: the Hub already has the storage, tracking, and compute to support these workloads.
- His closing next step: go run a real experiment whose outcome you can check.

## Resources

- [Custom Kernels for All from Codex and Claude](https://huggingface.co/blog/custom-cuda-kernels-agent-skills) — walkthrough for building, benchmarking, and publishing agent-generated kernels.
- [Hugging Face Kernels](https://huggingface.co/docs/kernels/index) — docs for the kernel distribution and development library.
- [Generating and evaluating kernel-writing skills](https://huggingface.co/blog/upskill) — UPskill generation, evaluation, and model comparison.
- [We Got Claude to Fine-Tune an Open Source LLM](https://huggingface.co/blog/hf-skills-training) — agent-driven fine-tuning on hosted GPUs.
- [Training with Unsloth and Hugging Face Jobs](https://huggingface.co/blog/unsloth-jobs) — tutorial combining Unsloth and hosted compute.
- [Karpathy autoresearch](https://github.com/karpathy/autoresearch) — single-GPU training experiment loop with a fixed time budget.
- [AutoLab pre-training research project](https://github.com/burtenshaw/multiautoresearch/tree/main/pre-training) — the multi-agent research project.
- [Trackio documentation](https://huggingface.co/docs/trackio/index) — experiment logging, dashboards, and accessible metrics storage.
- [KernelBench: Can LLMs Write Efficient GPU Kernels?](https://arxiv.org/abs/2502.10517) — the benchmark cited on LLM-written kernels.
- [FlashAttention](https://arxiv.org/abs/2205.14135) — the paper behind the memory-aware attention kernel.
