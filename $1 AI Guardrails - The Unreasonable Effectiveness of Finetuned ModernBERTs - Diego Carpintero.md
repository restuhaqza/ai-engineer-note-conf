# $1 AI Guardrails: The Unreasonable Effectiveness of Finetuned ModernBERTs

| Field | Details |
| --- | --- |
| **Speaker** | Diego Carpintero |
| **Affiliation** | Independent |
| **Type** | Talk |
| **Date** | 2026-04-16 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=YZHPEkfy2kc) |
| **Tags** | `architecture` `rag-context-and-search` `safety-and-governance` `apis-mcp-and-protocols` |

## Summary

Diego Carpintero argues that LLM attacks have moved from isolated prompt tricks to a broad, evolving landscape that spans prompts, retrieved context, model internals, tools, and autonomous agents. The root cause is that LLMs have no native boundary between developer instructions and untrusted data: both are concatenated into what the model treats as a single document.

He walks through concrete cases. The "Sydney" incident showed a Bing Chat system prompt could be extracted with plain-language requests, exposing its codename and dozens of internal rules. Indirect injection placed malicious prompts in Wikipedia pages and websites, including an AI advertising-review case where the content under review overruled the review. Other attacks exploit model math (adversarial suffixes that steer the model away from refusal), poison retrieval data, hide instructions in MCP tool descriptions, or turn agents into actors that download and run code.

The defense he proposes is zero trust: verify at every boundary, not just the user input and the model output. He lists options such as rule filters, canary tokens, constrained decoding, and an LLM acting as a judge, then focuses on a discriminator — a fine-tuned **ModernBERT** encoder that classifies text as safe or unsafe in about 35 milliseconds.

Most of the talk explains why an encoder fits. It reads the whole sequence in one forward pass, is cheap to retrain as attacks change, and can be self-hosted so internal prompts and responses never leave your infrastructure. He covers ModernBERT's efficiency tricks and then fine-tunes it on roughly 75,000 labeled examples, reaching about 85% accuracy on unseen prompt-safety benchmarks for under a dollar.

## Key Takeaways

- **LLMs have no native control/data boundary.** System instructions and user or retrieved content are processed together.
- **Prompt injection is now the baseline**, not an edge case, and it spreads through retrieval (RAG), MCP tools, and agents.
- **Alignment is a probabilistic preference, not a hard constraint**, so it cannot serve as a security boundary.
- **Zero trust applies to LLMs too:** check inputs, outputs, retrieved documents, MCP calls, memory, and agent plans.
- **Classification, not generation.** A bidirectional encoder is a good fit for repeated safe/unsafe checks.
- **Latency and cost compound.** Many checks per request make an LLM judge expensive; a ~35 ms discriminator stays cheap.
- **Retrainability and privacy favor self-hosting.** An encoder can be retrained in hours on your own hardware.
- **About 85% accuracy at ~35 ms is a useful baseline, not a guarantee** — one layer among several.

## Detailed Notes

### When instructions and untrusted data occupy the same space · 01:14
- **Direct prompt injection** is crafted user input meant to override system controls and exfiltrate data; it can work in one request or across several steps.
- The **Sydney** case: one day after Bing Chat's preview, a Stanford student used a simple "ignore previous instructions" query to reveal the system prompt, the codename Sydney, and 40+ confidential rules.
- The same extraction was reproduced through impersonation, and a later bypass survived a vendor fix; this affected many providers, not just Microsoft.
- The cause: user input is concatenated with the system prompt, so the model sees both as one document.
- LLMs therefore lack a native separation between trusted instructions and untrusted data, against normal security practice.
- **Indirect injection** places the malicious instruction in external content (HTML, a URL, a public page, or an email inbox) and waits for an application to fetch it.
- Case: a Wikipedia page was edited to tell an AI to search for a code as an "emergency"; the code led to an attacker-controlled malware site.
- A real case (reported March 26) involved websites embedding prompts to manipulate AI advertising-review systems into approving non-compliant content — the data being judged overruled the judge.

### Attacks exploit geometry, retrieval, tools, and agents · 06:02
- **Model geometry:** the attacker appends gibberish suffix tokens to a harmful request, shifting the next-token distribution out of the refusal region.
- Once the model begins with an affirmation like "sure, here is how," autoregressive continuation carries it into the prohibited answer.
- The **greedy coordinate gradient (GCG)** method searches for such a suffix iteratively: start with placeholder tokens (about 20 exclamation marks), define a loss that favors an affirmative opening, compute gradients, and test candidate replacements.
- Computing those gradients needs an open-weight model, but suffixes found across open models can transfer to closed models that share similar refusal boundaries.
- **Retrieval poisoning:** **RAG** (retrieval-augmented generation — fetching documents to ground answers) adds a poisoning path. In the cited **PoisonedRAG** study, inserting 5 malicious chunks into a knowledge base of 8 million documents was enough to force an attacker-chosen answer for one target question.
- Two conditions must hold for poisoning: the content has to be retrieved (easy to satisfy by appending the likely query), and it must sound convincing enough to influence generation.
- **MCP:** (Model Context Protocol — a standard for agents to call tools) creates an asymmetry: a human approves a tool by its name and short summary, while the model reads a longer description that can hide instructions.
- Example: a user approves an "add two numbers" call, while the model exfiltrates a private key and MCP credentials through a hidden function-call parameter; the visible result still looks normal. Other research describes further protocol exploits, including exfiltration of WhatsApp chat histories.
- **Agentic attacks** turn text into action. In one computer-use proof of concept, a web page told the agent to download a file; the agent downloaded it, located it, made it executable, and enabled a remote-code-execution path.
- An agent that can write, compile, and run programs might build a malicious executable itself instead of retrieving one.
- **Supply chain:** a variant began with a malicious npm package and a public GitHub issue whose title instructed a coding agent to install it; the title was interpolated straight into the agent's prompt, and roughly four to five thousand developers were affected.
- Conclusion: the attack surface is distributed, diverse, and mutable, and it self-escalates if nothing is done.

### Zero trust means checkpoints at every boundary · 14:29
- **Zero trust** follows one rule: trust nothing, verify everything. LLM applications break it by merging controls with data and by treating alignment as a hard constraint.
- The "iceberg effect": what a human reviewer sees may not match what the model actually receives.
- Consequences fall into three areas:
  - **What is told:** leaks of personal data and health records; false grounding; toxic output.
  - **What is done:** unauthorized actions such as fraud and impersonation.
  - **What is believed:** manipulation and persuasion that bias decisions.
- The speaker stresses the goal is to protect people, not merely pass a security audit.
- More autonomy and complexity means more checkpoints. At minimum, check user inputs and model outputs.
- Ideally, also check retrieved documents, MCP interactions, context memory, and agent plans before they influence consequential behavior.
- Implementation options: rule-based filtering, canary tokens, discriminative classifiers, constrained decoding, and LLM-as-a-judge where extra latency is acceptable.

### A bidirectional encoder makes repeated classification practical · 17:50
- Judging safe versus unsafe is a **classification** problem, not a generation problem.
- An **encoder** model sees all tokens in the sequence at once (bidirectional attention) and produces a condensed representation in a single forward pass.
- That representation is the `CLS` token, which is passed to a small classification head.
- The fine-tuned baseline classifies in about **35 milliseconds** with no batching or quantization, so there is room to improve.
- Why it matters: many checks per request make a generative judge add seconds, while an encoder stays fast.
- An encoder is also cheap to retrain in hours as attacks evolve, and self-hosting keeps internal requests, intermediate steps, and responses off third-party providers.
- **ModernBERT** updates the original BERT encoder for longer context and efficiency; its alternating attention plus FlashAttention was associated with roughly 70% lower fine-tuning memory in the reported setup.
- **Alternating attention** mixes close and broad reading: two local layers with a 128-token sliding window, then a global layer, supporting up to 8,192 tokens.
- This fits the problem: some attack signals are local (adversarial suffixes, GitHub issue titles), while others need broader context (retrieved documents, tool descriptions, agent plans).

### Stop wasting compute on padding and oversized layers · 23:56
- GPUs batch efficiently when tensors share a shape, but real prompts arrive at different lengths.
- Normal **padding** extends every sequence to the longest one, adding meaningless tokens; a cited experiment on the Wikipedia dataset found padding could account for up to 50% of computation.
- **Unpadding** removes placeholder tokens before embedding; **sequence packing** concatenates real tokens from multiple examples to fill the context, padding only at the end when necessary.
- A **masked-attention boundary** keeps tokens from one original sequence from attending to another, so packed examples do not mix meanings.
- **Deep and narrow** design: ModernBERT-base uses 22 layers with hidden size 768; ModernBERT-large uses 28 layers with hidden size 1,024.
- Those shapes came from a grid search over task performance and inference speed, not guesswork.
- Each layer refines the `CLS` representation once more, adding a level of semantic abstraction.
- Other efficiency choices: dimensions aligned to GPU tensor cores, gated activations, bias terms disabled except in the final decoder, and a normalization layer after embeddings.

### Positional encoding and keeping attention on-chip · 29:01
- Self-attention compares tokens but does not inherently know their positions, which matters for suffixes at the end of a request or instructions buried in a long document.
- The original approach added a fixed position vector to each token embedding; this entangles position with token meaning and caps input length at the trained size (BERT's 512 tokens).
- **Rotary positional encoding (RoPE)**, following the RoFormer paper, rotates query and key projections by angles tied to position.
- Because the attention score between two rotated vectors already encodes their relative distance, no separate learned position vector is needed and the context can be continuous.
- ModernBERT uses different RoPE scales for local and global attention: faster rotation for nearby tokens, slower for long range, to avoid distant positions appearing close.
- **FlashAttention** targets the GPU memory hierarchy. GPUs have fast on-chip memory (over 30 TB/s) and slower off-chip memory; the bottleneck is moving data between them.
- Instead of materializing the full attention matrix, FlashAttention processes the sequence in blocks, computes partial scores in fast memory, and accumulates the results.
- This is a major contributor to the classifier's low latency.

### Fine-tuning ModernBERT into a guardrail · 34:58
- The training corpus is roughly **75,000 labeled examples** from 20 open sources, associated with **InjecGuard**.
- Start with the ~150M-parameter base model to validate the pipeline and get a baseline, then switch to the large model if more accuracy is needed.
- In the reported comparison, the large model raised accuracy by almost 6 percentage points.
- Install FlashAttention to realize the attention gains; that helped reach around 70% memory savings.
- **bfloat16** (a reduced-precision number format) cut training memory by almost 40% and allowed a batch size of 64; an Adam-family optimizer is also used.
- Data prep: split into train and test with Hugging Face Datasets, keep prompt + label + source, tokenize (ModernBERT uses a modified byte-pair tokenizer with about 50,000 entries), and use `batched=True` for speed.
- Tokenized sequences start with `CLS` and end with `SEP`; for binary safety classification the `CLS` token carries the important representation.
- Fine-tuning adds a feed-forward classification head, compares the safe/unsafe prediction to the label, and updates the encoder and head through the loss.
- A pooling choice worth noting: default `CLS` pooling versus mean pooling, which averages all token representations and may help with very long inputs.
- The resulting checkpoint is published as **PangolinGuard-Large**.

### Evaluate on unseen data, then probe with attack prompts · 40:43
- Inference can run on CPU; on a GPU, enable FlashAttention to get the optimized kernels.
- On unseen prompt-safety examples, the speaker reports almost **85% accuracy** at about **35 milliseconds** per classification.
- The evaluation mixes subsets of **NotInject**, **BIPIA**, **Wildguard-Benign**, and **PINT**, testing both malicious-input detection and false alarms on benign prompts.
- This measures a component, not the security of a complete agent application; timing remains specific to the measured setup.
- A Hugging Face demo starts with a benign prompt (labeled safe), then runs the Sydney "ignore previous instructions" prompt and an impersonation prompt (both unsafe).
- It also flags the edited-Wikipedia redirection, the ad-review manipulation attempt, a harmful request with adversarial suffix tokens, and an MCP tool description hiding key exfiltration — all classified unsafe.
- The closing message: this is a baseline, not a gold standard. A self-hosted classifier on commodity hardware is one defensive layer, and AI safety is a shared responsibility.

## Resources

- [ModernBERT: Smarter, Better, Faster, Longer](https://arxiv.org/abs/2412.13663) — the architecture paper.
- [ModernBERT implementation](https://github.com/AnswerDotAI/ModernBERT) — the reference codebase.
- [PangolinGuard source](https://github.com/dcarpintero/pangolin-guard) — the speaker's guardrail repository.
- [PangolinGuard fine-tuning walkthrough](https://huggingface.co/blog/dcarpintero/pangolin-fine-tuning-modern-bert) — dataset preparation and training.
- [PangolinGuard-Large model card](https://huggingface.co/dcarpintero/pangolin-guard-large) — the fine-tuned checkpoint.
- [PangolinGuard demo](https://huggingface.co/spaces/dcarpintero/pangolin) — try the classifier.
- [InjecGuard paper](https://arxiv.org/abs/2410.22770) — dataset and over-defense mitigation.
- [PoisonedRAG paper](https://arxiv.org/abs/2402.07867) — the cited retrieval-poisoning study.