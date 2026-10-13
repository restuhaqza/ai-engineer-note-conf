# Let LLMs Wander: Engineering RL Environments

| Field | Details |
| --- | --- |
| **Speaker** | Stefano Fiorucci |
| **Affiliation** | deepset |
| **Type** | Talk |
| **Date** | 2026-04-08 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=71V3fTaUp2Q) |
| **Tags** | `enterprise` `apis-mcp-and-protocols` `data-and-model-adaptation` `evals` `reasoning-and-models` |

## Summary

Stefano Fiorucci, who builds the Haystack LLM framework at deepset, argues that
language models can learn by doing, not only by imitating examples. An
"environment" gives a model somewhere to act, run tools, and attempt multi-step
tasks, and it returns feedback the model can learn from. He frames this as the
engineering side of modern reasoning models such as OpenAI o1 and DeepSeek-R1.

To make the ideas concrete, Fiorucci turns tic-tac-toe into a training gym. He
starts with a small open model that barely plays, wraps the game in a reusable
environment using Prime Intellect's Verifiers library, then improves the model with
a short supervised fine-tuning warm-up followed by reinforcement learning (RL,
where a model is rewarded for good outcomes). The result is a small model that
draws most games against perfect play.

The talk is less about a scoreboard than about the plumbing. Fiorucci shows how
parsers, reward rules, opponent difficulty, and controlled randomness all shape
what the model actually learns. Several failed runs show that a healthy-looking
reward number can still hide a narrow or biased training setup.

The practical message: if you can define a clear, checkable reward, you can build
an environment and train a small specialist model that beats a larger closed model
on one focused task.

## Key Takeaways

- **An environment turns outcomes into signal.** Anything the computer can check
  automatically — a right answer, a won game, a successful tool call — can train a
  model.
- **RL explores beyond the examples.** Supervised fine-tuning imitates given
  answers; reinforcement learning samples many attempts and reinforces better ones.
- **Verifiers packages environments as Python artifacts.** It handles model
  serving, async rollouts, parsing, and rewards, so you focus on the task.
- **Design the reward and the opponent, not just the prompt.** Opponent skill and
  output format each get their own reward.
- **Control randomness for fair comparison.** Group-based RL needs reward
  differences that reflect the model's choices, not luck.
- **Batch size is a stability knob.** Below roughly 256 games per update, Fiorucci
  saw unstable training and collapse in this setup.
- **Inspect real rollouts and play the game yourself.** Aggregate plots can look
  healthy while the model quietly memorizes a biased opponent.
- **Warm up an instruct model with SFT, then train with RL.** This beats starting
  from a reasoning model that emits long traces under tight GPU limits.

## Detailed Notes

### Why practice beats examples · 00:07
- Fiorucci works on AI orchestration at deepset by day and tinkers with small
  models and RL by night.
- Environments are "gyms" where models use tools, run code, and solve multi-step
  tasks.
- RL-environment startups are drawing major funding and working with big labs;
  DeepSeek and MiniMax reports describe training across thousands of environments.
- Tic-tac-toe is chosen because its small state space makes every engineering
  decision inspectable.
- **The RL loop:** an **agent** (the model) sees a **state**, takes an **action**,
  and gets a numerical **reward**. The goal is to maximize total reward.
- It must balance **exploration** (try new actions) with **exploitation** (reuse
  what works). A **trajectory** (or **rollout**) is one whole game, not one move.

### From imitating answers to rewarding outcomes · 03:32
- Standard training: pretraining on text, supervised fine-tuning (SFT) on curated
  examples, then RL to align behavior. **PPO** (proximal policy optimization) is
  one RL technique from that third stage.
- SFT works by statistical imitation: the model mimics the prompt–response pairs it
  is shown.
- Ilya Sutskever's 2024 NeurIPS talk flagged diminishing pretraining returns;
  OpenAI's o1 improved with more RL in training and more thinking at inference.
- **DeepSeek-R1** used **RL with verifiable rewards** — a checker compares the
  answer to ground truth — plus **GRPO** (group relative policy optimization), a
  lighter setup than PPO.
- Key difference: SFT stays close to its examples, while RL can discover strategies
  beyond them. Fiorucci quotes Karpathy on environments letting models act and see
  outcomes rather than only imitating experts.

### Building environments with Verifiers · 10:19
- **Verifiers** (by Prime Intellect) packages environments as installable Python
  packages usable for both evaluation and training.
- It provides base classes for single-turn, multi-turn, and tool environments, plus
  abstractions for parsing responses and defining rewards.
- Model serving sits behind an OpenAI-compatible endpoint, so OpenAI, OpenRouter,
  or a local model via vLLM all work.
- It handles async interaction and parallel rollouts, and integrates with Prime RL,
  Tinker, and SkyRL for training.
- **Reverse Text** example: `load_environment` assembles a dataset, an
  `XMLParser`, a reward comparison, and a `SingleTurnEnv`.
- Parsing is kept separate from scoring; a **rubric** collects weighted reward
  functions.
- Evaluation run: pick examples, sample several rollouts each, send to the model,
  parse, score, and summarize. Training reuses this and adds weight updates.

### A tic-tac-toe environment, step by step · 18:52
- `MultiTurnEnv` introduces a **state** dictionary for information kept across the
  episode, initialized by `setup_state`.
- `env_response` returns messages; a method marked `@vf.stop` ends the rollout when
  its condition is true.
- `ToolEnv` adds Python functions as tools: the model requests one, the environment
  runs it, and the result feeds the next response.
- Variants include **MCP environments** (connect to Model Context Protocol
  servers), **`StatefulToolEnv`** (per-rollout resources like a DB connection), and
  **recursive language model environments** (decompose long input via a REPL — not
  an infinite context window).
- First tic-tac-toe version: the model always plays X and goes first, returns a
  cell number 0–8 in `<move>` tags, and the opponent plays randomly.
- `env_response` parses the move, ends the game as a loss on an invalid move,
  otherwise applies it, plays a random opponent move, and checks win/draw.
- The rubric weights a win reward at 1 and an XML-format reward at 0.2.

### Adjustable difficulty and fair comparisons · 21:15
- A second version lets the model sometimes go first and adds an optimal opponent
  using **minimax**; against perfect play a draw is the best possible result.
- Too-perfect an opponent too early starves the model of wins and learning, so a
  parameter lets the opponent sometimes move randomly.
- `min_random_move_prob` and `max_random_move_prob` set the range: both 0 is always
  optimal, both 1 is always random, in-between is a mix.
- A thinking trace is added before the move, checked by a regex reward for `think`
  tags.
- Invalid moves no longer end the game; play continues with a flat −0.1 penalty and
  an eight-turn cap.
- To reduce noise, an **example seed** fixes the starting player, and each turn's
  seed derives from the seed and board state, so a repeated position gets the same
  opponent response.
- **Stratified sampling** forces each batch to mix opponent difficulties evenly,
  controlling noise across updates.

### Warm-up with SFT, then group-relative RL · 25:42
- Baseline: **GPT-5 Mini** follows format well and plays competently but not
  perfectly; Liquid AI's **LFM2-2.6B** struggles with format and legal moves.
- Plan: use SFT to teach the output contract and valid-move syntax, then RL for
  strategy — LFM2 is small, fast, and already instruction-tuned.
- Generate 200 synthetic examples with GPT-5 Mini, filtering out losing games;
  Prime RL runs the SFT in a few minutes on a 96 GB GPU.
- After SFT, formatting is near perfect and invalid moves drop, but real strategic
  weaknesses remain.
- Group-relative loop: sample many games from one board, score each with
  deterministic rewards, compare each to the group average to get its **advantage**,
  then favor the better trajectories.
- The experiment uses **CISPO** (from MiniMax-M1) for importance-weight clipping,
  presented as an improvement over GRPO here.
- First RL run samples opponent random-move probability from 20%–70%, excluding
  purely random and perfectly optimal players to allow both attack and defense.
- He saw unstable training and collapse below 256 games per batch: too few games
  expose too narrow a set of opponents.

### What failed, and what to inspect · 30:55
- Run one: win and total reward rose, format stayed near perfect, and the
  invalid-move penalty fell toward zero.
- Evaluation after run one: an 85% draw rate against an optimal opponent over 100
  games, dominance over random players, and near-zero invalid moves.
- Inspecting rollouts revealed recurring **fork traps** — the opponent builds two
  winning paths and the model can block only one; the decisive mistake happened
  before the final move.
- Run two narrows opponent randomness to 0%–25% (stronger opponents). Training only
  against perfect opponents made the model overly defensive, so some imperfect play
  is kept.
- Higher **sampling temperature** encourages exploration; too high produces
  gibberish. Rewards dipped initially, then recovered to new highs.
- Final comparison: similar to GPT-5 Mini against random opponents, but better than
  the teacher against an optimal opponent.
- Lessons: large batches learn slower but more stably. Watch for hidden environment
  biases — an earlier minimax always picked the first free square among
  equal-scoring moves, so the model memorized that narrow opponent.
- A reasoning starting model emits long traces that get truncated under tight
  limits, wasting budget; prefer an instruct model and add reasoning for the task.
- Check candidate base models with a few completions first; inspect rollouts; and
  after programmatic evaluation, play the task yourself.
- His closing advice on patience: "start training and go for a walk."

### To real tools · 38:40
- SFT and RL are complementary: demonstrations teach the initial contract,
  interaction and rewards refine behavior.
- His broader proposal: with a clear reward signal, train a small specialist that
  can outperform a larger closed model on one task, at lower cost (no matching
  total-cost measurement is given).
- Next project: train a small model on two or three tools you use often and test it
  against a larger model on that workflow; the `wiki-search` example in the Prime
  RL repository is a recommended starting point.

## Resources

- LLM RL Environments Lil Course — tic-tac-toe environment and training code.
- Verifiers — Prime Intellect's environment library.
- Single-turn and multi-turn environment examples (Reverse Text, DoubleCheck).
- Tic-tac-toe RL training and evaluation walkthroughs (first and second runs).
- MiniMax-M1 paper — introduces the CISPO algorithm used here.
