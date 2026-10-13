# How We Built Zeta2: Training an Edit Prediction Model in Production

| Field | Details |
| --- | --- |
| **Speaker** | Ben Kunkle |
| **Affiliation** | Zed |
| **Type** | Talk |
| **Date** | 2026-05-30 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=phchDt63qAA) |
| **Tags** | `coding-and-developer-tools` `infrastructure-and-deployment` `evals` `observability-and-reliability` `industry-applications` |

## Summary

Zeta 2 is Zed's edit-prediction model: it reads the code around your cursor and predicts your next change. Because it runs on **every keystroke**, it must answer almost instantly, which is why Zed chose a small, specialized, fine-tuned model rather than a large general one.

Ben Kunkle, the edit-predictions lead at Zed, walks through the training pipeline. It starts with opt-in production snapshots, uses a large "teacher" model to generate targets (distillation), and then repairs and filters those targets so the small "student" model learns from clean data.

Most of the talk is about making data work cheaply and honestly: caching teacher output, using the student itself as a filter, and evaluating candidates against multiple reasonable answers. The talk closes on how Zed tests models with live traffic and watches acceptance rate and latency.

## Key Takeaways

- **The task is narrow and constant:** predict the next edit from cursor context, run on every keystroke, so latency matters more than raw model size.
- **Rich inputs help:** recent edits, cursor position, nearby type and variable definitions, and diagnostics/errors.
- **The teacher is inconsistent.** A frontier model asked 100,000 times gives slightly different answers, so teacher output must be checked and repaired.
- **Distillation:** turn a repaired teacher prediction into the expected output the student learns to reproduce.
- **Cache the expensive part.** Teacher and repair work is reusable; experiments only change how examples are formatted.
- **"Settled data" is noisy.** Wait for editing to stop, snapshot the result, but filter out cases where the developer changed direction.
- **Near misses are the gold.** Examples the model almost gets right often teach the most; run offline evals against several acceptable answers, not one.
- **Offline scores don't equal user approval,** so Zed also runs live experiments and watches acceptance rate and latency.

## Detailed Notes

### An edit prediction on every keystroke · 00:15
- An **edit predictor** looks at the code around your cursor and predicts the next change you will make.
- Its task is to propose an edit within a supplied region of code.
- The editor supplies more than the current text: **recent edits**, the **cursor position**, nearby **type and variable definitions**, and **diagnostics/errors**.
- Prediction runs on **every keystroke**, so it must return fast enough to be useful while you type.
- That mix of rich context, a narrow task, and frequent use is ideal for a **small, fine-tuned, specialized model**.

### Turn production snapshots into checked teaching targets · 01:02
- Training starts with **opt-in production snapshots** that preserve the context available when a prediction was requested (including related definitions).
- Zed turns them into training examples through **distillation**: give a **frontier model** (the teacher) the captured input and ask for an edit prediction.
- Teachers are inconsistent: ask the same thing many times and answers vary, and some do not meet the task's rules.
- The team refines the teacher prompt, then **checks** output instead of trusting all of it.
- The repair sequence:
  1. Run **static heuristics** to catch predictions that undo what the developer just typed.
  2. Check whether a prediction crosses the **editable-region boundary**.
  3. Send a failing prediction to **another frontier model** with the specific failure described, asking for a fix.
- After repair, the teacher's edit becomes the **expected output** for the student.

### Cache the teaching work, vary the student input · 02:36
- Teacher and repair work is **reusable across experiments**, so Zed caches it before experiment-specific formatting.
- Experiments vary what the student sees: include diagnostics or not, change how much **edit history** is passed, change the desired output format.
- Crucially, these choices **do not require the teacher to generate its answer again**.
- Data is stored as **JSONL** (one large JSON object per line); each stage adds fields or rearranges them, which keeps the process fluid.
- Typical sizes: around **100,000 examples** at the top end, and **10,000–50,000** for smaller experiments.

### Use later code as evidence of intent · 03:53
- The developer eventually writes the code they want — that motivates **settled data**.
- Because Zed is the editor, it can wait until editing stops in the model's region, then snapshot that region.
- The later state is **not automatically a clean answer**: the developer may change direction, or an agent may rewrite the region entirely.
- So a prediction that was reasonable at request time can look wrong against code written after a change of intent.
- One filter: generate **ten teacher predictions** from the same context and compare them to the settled region using a distance measure such as **Levenshtein distance** (edit distance).
- Close matches suggest the outcome was reachable from the input — but ten predictions × 100,000 examples means **one million frontier requests**, which Kunkle calls prohibitively expensive.

### Find useful near misses with the student · 05:29
- The trained **student** provides a cheaper filter, since its quality is approaching the teacher's.
- He describes sampling a student checkpoint about **50 times per example** at much lower cost than calling the teacher repeatedly.
- The same idea applies: look for a generated prediction close to the settled region.
- Distances sort the training material:
  - **Far from settled code** → likely noise or a change of intent.
  - **Very close** → probably already obvious to the model (his example: a second operand in an `add` function).
  - **Almost right (the middle)** → often the most useful, e.g. a new function past the student's training cutoff.
- The best examples are **not** the ones the model already reproduces most easily; near misses expose gaps.
- Even after filtering, Zed generally does **not** train on the raw settled region — it stays noisy — but on the generated prediction closest to it.

### Evaluate more than one reasonable next edit · 07:03
- Offline evaluation uses a **held-out test set**, keeping test examples separate from training examples.
- **`deltaChrF`** compares **character n-grams** of different lengths (a character-level similarity score).
- The team also tracks **reversal ratio**: predictions that undo exactly what the developer just typed — a specific bad behavior a general score might miss.
- They also watch **production kept rate** as a signal.
- A context often allows **several reasonable next edits**, so they evaluate each held-out context against **three distinct frontier-model predictions**.
- A student answer close to any one of those references can still be a good answer.

### Deploy experiments and measure what survives · 07:54
- Offline scores do not necessarily match what developers want in the editor, so Zed evaluates through **deployed experiments**.
- The experiment page shows live variants; one receives **15% of production traffic**, the other the remainder.
- After deployment, the team can adjust allocation (e.g. 15% → 20%) or **promote** a variant to the live model.
- A dashboard tracks **acceptance rate** and **latency**; **Zeta 2** is among the experiments managed this way.
- They are exploring signals beyond immediate acceptance:
  - **Kept rate** compares post-prediction text to the later settled region to see how many characters were retained.
  - **Diagnostic-error counts** compare errors before and after a prediction.
- These capture things acceptance alone misses: a suggestion can be accepted, then edited, and change diagnostics.

### The stopping rule is ten seconds of inactivity · 09:37
- The closing question: how does Zed decide a region has "settled"? Does it use a Git commit as a sign the developer is satisfied?
- **No** — the implementation does not inspect Git commits.
- Zed snapshots a region after **ten seconds without edits** to that area.
- This is a rough collection heuristic, not a claim that the developer finished or approved the code.
- If someone keeps editing the same spot without a ten-second pause, that region is not captured until such a pause happens.

## Resources

- How We Developed Zeta2 — Kunkle and Syvokon's companion account of the pipeline.
- [Zeta 2 model card and weights](https://huggingface.co/zed-industries/zeta-2) — the model's public weights and prompt format.
- [Zed source repository](https://github.com/zed-industries/zed) — the editor behind production context collection.
- Configure edit prediction in Zed — current instructions for trying edit predictions locally.
- [Character n-gram evaluation with chrF](https://aclanthology.org/W15-3049/) — background on the character n-gram metric family (not Zed's custom metric).
