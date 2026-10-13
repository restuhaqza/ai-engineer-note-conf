# Training an LLM from Scratch, Locally

| Field | Details |
| --- | --- |
| **Speaker** | Angelos Perivolaropoulos |
| **Affiliation** | ElevenLabs |
| **Type** | Workshop |
| **Date** | 2026-05-04 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=UsB70Tf5zcE) |
| **Tags** | `speech-and-audio` `reasoning-and-models` `architecture` `evals` |

## Summary

Angelos Perivolaropoulos, who leads ElevenLabs' speech-to-text team, builds a small language model from scratch — no pretrained weights, just PyTorch and a few basic libraries. The goal is to expose the machinery: choose a tokenizer, assemble a GPT-2-style decoder, write a training loop, and generate text. The model is deliberately small enough to train on a laptop or a free Google Colab GPU, in about 15 minutes on a T4.

The first decision is the tokenizer. Because the dataset is a small Shakespeare corpus with only 65 distinct characters, he uses a character-level tokenizer. That keeps the vocabulary tiny, which matters because a large off-the-shelf vocabulary would be a poor fit for a small training budget. Character tokenization also pushes work into the model: it must compose letters like `s`, `k`, `y` before it can relate `sky` to `blue`.

The architecture is a stack of repeated blocks, each combining multi-head attention, a feedforward network, residual connections, and layer normalization. He demystifies it by counting parameters: most of the roughly 10 million weights live in the repeated blocks, not the small embedding table. Training uses next-token prediction, a warm-up-then-cosine learning-rate schedule, and validation loss to catch overfitting. The loss starts near the random baseline of about 4.17 and falls through recognizable milestones as the output turns from noise into plausible verse.

The closing Q&A widens the view: reasoning models share the same base architecture but are post-trained on curated, expert-written data; audio and multimodal models reuse the transformer but change the representation, the encoder, and the loss.

## Key Takeaways

- **Four building blocks:** tokenizer, architecture, training loop, inference. The training loop is where most capability gains come from.
- **Match the vocabulary to the data.** A character tokenizer is a good fit for a 65-character corpus; a 50,000-token vocabulary would add more parameters than the whole small model.
- **Character tokens move work into the transformer.** The model must learn letter combinations before it can learn word relationships, and it processes more tokens per sentence.
- **Attention, MLP, residuals, layer norm.** Each block has its own weights; residual connections (`x = x + update`) let a layer refine rather than replace its input.
- **Context length and parameter count are different controls.** You cannot just raise the context length — memory, attention cost, and stability all have to change.
- **Watch validation loss, not just training loss.** Falling training loss with rising validation loss means overfitting.
- **Generation is logits, then temperature, optional top-k, softmax, then sampling.** Greedy decoding is deterministic but boring; ~0.7 temperature is a reasonable starting point.
- **Reasoning is post-training.** The same base architecture learns to reason from high-quality reasoning data, not a new design.

## Detailed Notes

### Goals and setup: a tiny GPT-2 trained from nothing · 00:29
- The workshop trains a language model purely on Torch and basic libraries, with no pretrained weights or transformers-library shortcuts.
- Perivolaropoulos leads the speech-to-text team and mentions **Scribe v2**, ElevenLabs' batch transcription model, and its public-benchmark error-rate claim. The exercise is text-only.
- The repository offers two routes: train locally (roughly 16 GB of memory suggested) or use Colab's free GPU runtime.
- Karpathy's **nanoGPT** is the motivating reference: a compact PyTorch project that makes language-model training approachable.
- The workflow is four pieces — tokenizer, architecture, training loop, generation. The model is a small causal decoder: causal self-attention, feedforward layers, and normalization.

### Pick a tokenizer the data can support · 09:10
- A language model processes vectors, so text must first become integer IDs. The Shakespeare corpus has 65 distinct characters, making a character-level tokenizer a small, transparent choice.
- Build a sorted character vocabulary, map each character to an integer (`stoi` is a string-to-integer dictionary), and invert the mapping to decode. The same mapping must be used when generating from a saved model.
- With 65 characters there are 65² = 4,225 possible ordered **bigrams** (adjacent pairs). That small space is why the model can meet repeated patterns within a small corpus.
- The vocabulary-squared heuristic is intuition about sparsity, not a minimum-data formula: a 200,000-token vocabulary would need far more data, and training with a full tokenizer would never converge here.
- **Byte-pair encoding (BPE)** is the common alternative: it learns reusable pieces by merging frequent adjacent patterns. It needs more data and compute than this exercise allows.
- Vocabulary size drives the embedding table directly: 65 × 384 is about 25K parameters, while ~50,000 tokens × 384 is about 19.2 million — more than the whole small model.

### The transformer building blocks · 15:04
- **Multi-head self-attention** lets each position combine information from earlier positions; different heads can learn different relationships (one might track punctuation, another grammar). Those roles are examples of learned behavior, not labels assigned in code.
- The **MLP** (feedforward network) transforms each position's contextual representation into features the next layers and the output head can use.
- **Residual connections** add a learned update to the incoming representation (`x = x + update`) so a layer refines what is present rather than replacing it.
- **Layer normalization** regulates activation scale — without it, repeated amplification could turn a small value into an enormous one across layers.
- The configuration separates vocabulary size (65), context length (256 tokens), depth (6 layers), heads (6), and representation width (384). The workshop calls 384 the exercise's embedding width, not a general GPT-2 spec.
- **Context length is not parameter count.** A small model can process long sequences and vice versa. This model trains on at most 256 tokens, so it cannot be asked to handle arbitrary longer input without architectural changes.

### From token IDs to next-token loss · 30:10
- Two embedding tables — one for token identity, one for position — are added, passed through the transformer blocks, normalized, and projected through the **language-model head**.
- The head produces **logits**: unnormalized scores for the vocabulary (one score per character). During generation those scores become a probability distribution for the next token.
- Shapes clarify the flow: input `(batch, time)` → hidden `(batch, time, width)` → logits `(batch, time, vocab_size)`. With targets, cross-entropy scores the prediction at every position; without targets, the model returns scores for inference.
- Each transformer block has its own attention, normalization, and MLP parameters — blocks share an architecture, not weights.
- The model uses pre-normalization: normalize before attention, add the attention update, normalize again, add the MLP update. The residual is visible as the two additions.
- Parameter count comes from the instantiated model; the default is roughly 10 million parameters, most of them in the repeated blocks.
  - Token embeddings: 65 × 384 ≈ 25K
  - Position embeddings: 256 × 384 ≈ 98K
  - Attention per block: 4 × 384² ≈ 590K
  - MLP per block: 2 × 384 × 1,536 ≈ 1.18M
- Attention and MLP together are about 1.8 million weights per block; six blocks explain the ~10M scale. Biases, normalization parameters, and the output head affect the exact total.

### The training loop: batches and a learning-rate schedule · 40:54
- The task is **next-token prediction**: for each position the target is the following token. The objective is chosen to be learnable and easy to recognize — Shakespeare-like verse, not competitive programming.
- To build examples, shift the sequence: for `KING` with IDs `[2,1,3,0]`, inputs are `[2,1,3]` and targets `[1,3,0]`, so the model learns to predict each next character. Causal attention prevents a position from seeing the token it must predict.
- The corpus is about one million characters, split into training and validation. Each batch holds 64 sequences of 256 tokens sampled as contiguous windows.
- The code selects `cuda` (NVIDIA GPU), `mps` (Apple Silicon), or `cpu` automatically. CUDA and MPS accelerate; CPU is slower but supported.
- The **learning rate** controls update size. Too large overshoots and destabilizes training; the workshop uses a 100-step warm-up then cosine decay across 5,000 steps.
- Learning-rate decay and **weight decay** are separate: the former changes update size over time, while AdamW's weight decay regularizes the weights. A scheduler or explicit function supplies the cosine schedule.
- **Training loss** measures fit to sampled training data; **validation loss** measures predictions on held-out text. Checkpoints are saved every 1,000 steps, and samples are generated along the way.

### What the loss tells you · 49:54
- A uniform predictor over 65 characters has cross-entropy ln(65) ≈ 4.17, the useful baseline for a randomly initialized model.
- Approximate milestones in this exercise:
  - ~3.3 — character frequencies emerge
  - ~2.5 — fragments such as `TH` and `in` improve
  - ~1.5–2.0 — recognizable words appear
  - ~1.0–1.2 — names and more plausible Shakespeare-like text
  - below ~1.0 — overfitting becomes a concern in this dataset
- These are task-specific observations for reading samples, not thresholds to transfer elsewhere. The held-out trajectory matters more than crossing a particular number.
- In his test, 200 steps produced nonsense (validation loss around 3.5); output improved around 800 and 1,000 steps; the best point was roughly 2,400 steps, after which validation loss rose and output became less creative.
- Curve reading: a flat training loss suggests a bug or too-low learning rate; training falling while validation rises means overfitting; sharp spikes suggest data or numerical problems; a plateau points to data limits or capacity.

### Generation: temperature, top-k, seeds · 53:06
- Generation repeatedly runs the model and appends one chosen token.
- **Greedy decoding** always picks the highest-scoring token. It is a valid choice when determinism is wanted — he notes it suits transcription, where creativity is undesirable — but for creative verse it is "boring".
- **Temperature** reshapes the distribution before sampling: lower values concentrate on high-scoring tokens, higher values spread probability. He suggests 0.7 as a starting point.
- **Top-k** restricts sampling to the highest-scoring candidates, preventing an unlikely token from slipping through even at higher temperature.
- The order is logits → temperature scaling → optional candidate filtering → softmax → sampling. Only the final position's logits choose the next token, and the growing input is kept within the trained context window.
- A fixed **seed** fixes the pseudorandom generator's starting state so the same sampling sequence can be reproduced. It does not make every GPU operation deterministic.
- The project resolves into three files: `model.py` (architecture), `train.py` (data loading and optimization), and `generate.py` (inference) — a few hundred lines total.

### Beyond the toy: reasoning, audio, and multimodal · 1:03:34
- **Reasoning models** share the base architecture and are post-trained on high-quality data. Intermediate reasoning tokens become extra context, so later answer tokens can attend to a problem restatement or a calculation.
- **Qwen3** is his example of post-training on a common foundation; curated reasoning data is part of the recipe, not the whole of it. He associates rapid capability gains with task-focused data that resembles benchmark tasks.
- High-quality post-training data comes from specialist contractors — for instance physicists recruited through labeling companies such as Scale AI, or internal labeling teams. Defects in this data can make or break a model, so experienced labelers move into QA roles to check others' work.
- **Audio** keeps many transformer ideas but changes the representation. Models must decide what to preserve: speech, speaker traits, music, or a mix, and generated audio tokens need decoding back to waveforms.
- Different objectives fit different representations: **cross-entropy** for discrete targets, **L2 loss** comparing mel-spectrograms (a common text-to-speech approach), and **KL divergence** to train a smaller student to match a larger teacher during distillation.
- **Multimodal** models need not build one giant vocabulary. A separate encoder produces vectors — a video encoder, for example — and those vectors replace ordinary token embeddings at a placeholder position. The receiving transformer only cares about dimensions matching. Whether a horse video's representation resembles the word "horse" depends on training and stays an open question.
- **Music** can be modeled autoregressively or with diffusion; he says diffusion components are often easier to get working for audio, music, and images, without presenting a controlled comparison.
