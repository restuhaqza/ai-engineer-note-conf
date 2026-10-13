# Building a Chess Coach

| Field | Details |
| --- | --- |
| **Speaker** | Anant Dole, Asbjørn Steinskog |
| **Affiliation** | Take Take Take |
| **Type** | Talk |
| **Date** | 2026-05-13 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=FlzpEGHNVKQ) |
| **Tags** | `agent-engineering` `creative-and-generative-media` `observability-and-reliability` `evals` `reasoning-and-models` |

## Summary

Anant Dole and Asbjørn Steinskog work at Take Take Take, the chess company founded by Magnus Carlsen, and describe the AI chess coach built into its iOS and Android app. After a game, the app marks moves and generates commentary. The key design choice is a strict split: chess analysis comes from engines and detectors, and the language model's only job is to turn those facts into plain English.

They explain why that split exists. Language models play chess poorly and lose track of the game, while a transformer trained on positions paired with engine evaluations can reach grandmaster strength — but it cannot explain anything. The coach needs both strong analysis and clear explanation, so it keeps a deliberate boundary between them.

The middle of the talk shows how the team closes the loop with an agent. When a player downvotes commentary, the report goes to Slack and to a running Claude Code session, which investigates, can revise prompts or detectors, regenerates the explanation, asks for guidance, and proposes a pull request for human review. They demo this live.

They close on trade-offs and lessons: post-game review wants feedback in under three seconds, so they use a fast model, and they evaluate explanations with a small scenario suite and their own chess expertise.

## Key Takeaways

- **Separate analysis from language:** engines and detectors decide the chess facts; the model only translates them into English.
- **Three signal sources:** Stockfish says which move is best, detectors name threats and structures, and Maia predicts what humans play at a given rating.
- **Playing strength ≠ explaining ability:** a position-trained transformer can play at grandmaster level but cannot describe why.
- **Move quality vs human difficulty:** a move can be the engine's best pick while being unlikely for a player at that rating.
- **Ground everything:** the model should not invent tactics or calculate new lines on its own, though hallucinations still happen.
- **Close the loop with agents:** downvotes trigger an investigation through Slack and a Claude Code channel, with humans reviewing before any merge.
- **Latency matters for consumers:** they target under three seconds; Gemini 3 Flash gives about a one-second first token and ~3s end-to-end.
- **Context is the hard part:** a large JSON file is pruned step by step, and human domain experts still own the final quality call.

## Detailed Notes

### The product: explanations, not just scores · 00:27
- The app is available on iOS and Android; users play friends and post about their games.
- After a game, the review combines automatic move notation with generated commentary.
- In the opening example, White's knight moves from `f3` to `e5`, capturing a pawn and leading to checkmate; the review labels it brilliant.
- The commentary explains the nuance — which positional and tactical detectors fired, what the threats are, and why the move works.
- A later layer adds insights like accuracy by game phase, current rating, and depth in an opening, which the coach turns into learning opportunities.

### Chess and AI: from brute force to hallucination · 03:00
- Steinskog, calling Shannon "the OG Claude," points to Shannon's 1949/1950 paper on programming a computer to play chess.
- Shannon's two strategies frame the history: **Type A** searches moves broadly to a limited depth; **Type B** selects promising moves and lines for deeper analysis.
- Early computers were too weak to search the whole tree, so selective search looked necessary.
- Faster hardware let brute-force engines scale, culminating in **Deep Blue** beating Kasparov in 1997 (its first individual game win came in 1996).
- **AlphaGo** used neural networks to select lines in Go's larger space; **AlphaZero** extended the approach to Go, chess, and shogi.
- Language models arrived with a different profile: plausible openings, but they lose the thread and hallucinate as the game continues.

### Playing strength and explaining ability are different · 05:03
- A clip of Magnus Carlsen commentating from the Oslo office shows **Grok** choosing an early queen move to `b6` in a poisoned-pawn line and losing badly; the commentary attributes this to a broader inability to play reliably.
- The game came from **Kaggle Game Arena**, which evaluates models through gameplay and has since added Werewolf, a social-deduction game.
- Strong reasoning models can work through some moves, but their calculations still break down in his account.
- The architecture is not the problem: he describes a DeepMind transformer trained on millions of positions paired with Stockfish evaluations, learning to evaluate chess rather than predict text, and reaching grandmaster-level strength.
- The distinction is the training objective: a model that evaluates positions cannot automatically explain them, and a model that talks about chess is not a reliable engine.

### Ground the model in chess facts · 06:16
- The pipeline first runs **Stockfish** (a leading open-source chess engine) over the whole game to find the best move in each position.
- The best move alone is not a lesson; players also need threats, plans, tactics, and alternatives to what they played.
- **Detectors** extract those features: forks, pins, and skewers (tactical), plus structural themes like doubled pawns (positional).
- **Maia**, from University of Toronto research, predicts human moves: given a rating, it returns a probability distribution over moves in the position.
- Combining the sources separates move quality from human difficulty — a move can be the engine's best while being unlikely for a player at that level.
- All of it becomes context for the language model, whose assigned job is deliberately narrow: translate the supplied information into English.
- The team does not want the model inventing tactics or calculating extra lines, so the intended source of every explanation is explicit: analysis first, generation second.

### A bad move can still contain a threat · 09:19
- In a game Steinskog played, the opponent advanced a pawn to `f5`; Stockfish marks it bad, which is useful for grading but does not answer what the opponent is trying.
- The detectors find a plan to trap the queen, including a bishop move to `g5`, and also the response: capturing the central pawn defends a square the queen can use to escape.
- A compact JSON representation makes the contract concrete: the played move, the engine verdict, the threat (goal and continuation), and the response (action and effect).
- The valuable part is the causal link — the pawn capture protects the queen's escape square — because that turns a bad-move label into coaching.

### From a downvote to a reviewed change · 09:53
- When a player downvotes commentary, the report goes to Slack and to a running Claude Code session through a **Claude Code channel**.
- **MCP** (a standard way for agents to call tools) is the mechanism: an MCP server injects external events into an active session (described as a research-preview feature).
- A custom **triage skill** defines the agent's procedure:
  1. Inspect the report and the position.
  2. Change the cause — the prompt, an existing detector, or a new detector.
  3. Regenerate commentary with scripts and inspect the result.
  4. Ask questions back through Slack so the team can guide chess judgments.
  5. When it looks right, request a pull request for human review in GitHub.
- The autonomous part covers investigation and candidate changes; humans still guide and approve before merge.
- The live demo starts with a deliberately weak report — the presenter marks commentary bad without checking it — and the visible result is a successful handoff into investigation.

### Latency vs quality, and evaluating explanations · 12:39
- Post-game review has a short latency budget: players step through moves and should not stare at an indefinite "coach thinking" screen.
- The team targets under three seconds. **Gemini 3 Flash** typically reaches its first token in about a second, with average end-to-end latency of about three seconds (application measurements).
- Other reasoning models give useful analysis but less predictable completion times, which is awkward for immediate review.
- A planned "chat with your coach" experience can afford longer waits, since users asking questions expect to wait more than users stepping through a finished game.
- Quality check: Dole and Steinskog are strong players and compare the generated response with how they would calculate and play.
- Automated evaluation uses 16 scenarios drawn from real games, covering tactics, blunders, and hallucination limits, with an **LLM judge** to score responses.
- Reported results: Gemini Flash about 75% pass rate, Claude with more thinking just under 60% with much longer latency, and GPT-5 Mini lower accuracy; exact versions and settings are not given, so this is the team's own comparison.

### Lessons: context is the hard part · 15:43
- Separate the data pipeline from language generation — it helps when latency matters.
- Close the loop with autonomous agents; the Slack-and-Claude flow lets them iterate quickly.
- Build a clear context-extraction model. This starts as a large JSON file and is pruned step by step while checking how output quality changes; a good context model emerges from repeated decisions, not by including everything.
- Automated evals help, and subject-matter experts remain necessary. In this team the builders are also strong chess players; in other domains, the right judges may be partners rather than the engineers.
- Closing the demo, the agent asks what specifically feels wrong about the commentary; the presenter admits nothing is wrong. The useful result is that a downvote started an investigation without establishing a defect — the loop can question a report before changing the system.

## Resources

- [Programming a Computer for Playing Chess](https://archive.computerhistory.org/projects/chess/related_materials/text/2-0%20and%202-1.Programming_a_computer_for_playing_chess.shannon/2-0%20and%202-1.Programming_a_computer_for_playing_chess.shannon.062303002.pdf) — Shannon's 1950 paper.
- [AlphaZero: Chess, shogi, and Go](https://deepmind.google/blog/alphazero-shedding-new-light-on-chess-shogi-and-go/) — DeepMind's account of self-play training.
- **Amortized Planning with Large-Scale Transformers** (paper) — learning chess action values from Stockfish evaluations.
- [Stockfish](https://github.com/official-stockfish/Stockfish) — the open-source engine used for position analysis.
- **Maia-2 implementation** (repository) — predicting human chess moves conditioned on player skill.
- [Claude Code Channels reference](https://code.claude.com/docs/en/channels-reference) — delivering external events into a running session.
- [Game Arena adds Werewolf and poker](https://blog.google/innovation-and-ai/models-and-research/google-deepmind/kaggle-game-arena-updates/) — Google's benchmark for models playing games.
