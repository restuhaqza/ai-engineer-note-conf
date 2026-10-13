# Running a Chess YouTube Channel Entirely by AI

| Field | Details |
| --- | --- |
| **Speaker** | Stephan Steinfurt |
| **Affiliation** | TNG Technology Consulting |
| **Type** | Talk |
| **Date** | 2026-07-08 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=BqZrTdgBaPw) |
| **Tags** | `vision-and-video` `agent-engineering` `reasoning-and-models` `creative-and-generative-media` |

## Summary

Stephan Steinfurt's team in Munich built a pipeline that downloads human games from Lichess every night, analyzes them, and publishes narrated chess puzzle videos to YouTube with no manual editing. The channel aims at something harder than showing the best move: explaining why a move works well enough to teach a person. A German newspaper called that goal the "Holy Grail of chess programming," and estimated it could still be five years away.

The core mismatch is familiar. Chess engines have played at a superhuman level for decades but explain their conclusions poorly. Language models explain fluently but cannot reliably play. The project joins the two: a model reasons in words while specialized tools supply chess facts. In the opening example, a rook sacrifice ends in a knight fork whichever way Black captures.

Steinfurt is honest about the state of things. The channel has no revenue yet, costs around 20–30 cents per video, and roughly one in twenty videos carries a strange description. He argues the output is not "slop," and sees a personal use case: generating a narrated video of any ordinary player's own game.

## Key Takeaways

- The pipeline runs nightly: download games from **Lichess** → background analysis → deeper agent investigation → a special intermediate format → a rendered, narrated video.
- The central idea is an **agent that pairs a model's verbal reasoning with chess-specific tools**, because engines and language models each have opposite weaknesses.
- **Gemini 3.1 Pro** was the best chess model Steinfurt had used; earlier the team's strongest was **Grok 4**, and OpenAI models had enough baseline chess knowledge to select tools.
- Tools cover legal moves, board state (play a move and take it back), engine evaluation, checks-captures-threats, and sometimes web search for a game's context.
- The **checks, captures, and threats** tool lets the agent explore tempting moves — a good lesson must sometimes explain a bad move a human would try.
- Reasoning models moved decisions from Python scripts into the agent, which now chooses which positions to investigate.
- Narration uses **ElevenLabs v3** text-to-speech with audio tags; the agent also decides which squares and arrows to highlight and what counts as a brilliant move.
- Human-move models like **Maia** (from the University of Toronto) enrich the candidate set, though exact imitation is not the point.

## Detailed Notes

### Can AI explain chess like a human trainer? · 00:16
- Steinfurt borrows a phrase from a German newspaper for his title, calling this the "Holy Grail of chess programming."
- The article, attributing the phrase to Wilhelm Weber, suggested AI might match a human trainer only after several more years.
- The ambition behind the videos is exactly that: explanations good enough to teach.
- He opens with one of the team's fully AI-generated lessons.

### One sacrifice, two ways to lose the queen · 01:29
- Black's queen is attacked by a white rook on the h-file and retreats to g4, which looks safe but is called a crushing blunder.
- Leaving the h-file lets White play `Rxh5`, offering a full exchange as a trap built on the knight's next jump.
- If Black recaptures with the g-pawn (`...gxh5`), White plays `Nf6+`: the knight checks the king on g8 and attacks the queen on g4, then takes it after the king steps to f8.
- If Black instead takes with the queen (`...Qxh5`), the same `Nf6+` fork wins the queen on h5.
- The shared mechanism is a **fork with check**; the second line adds attraction, pulling the queen into the knight's attack.
- This explanation — why both capture choices fail — is the work the video must do beyond merely naming `Rxh5` as strong.

### From nightly games to a finished lesson · 03:42
- The opening lesson was produced overnight; Steinfurt liked it better than the example he had prepared the day before.
- Four stages produce each video:
  1. Download human games from Lichess each night.
  2. Analyze them in the background, then let an agent investigate more deeply.
  3. Convert the analysis into a special intermediate format.
  4. Render a video that explains variations and flags brilliant moves and blunders.
- The intermediate format separates analysis from the final audiovisual presentation.

### Give the model a board it can investigate · 04:37
- The problem: engines do not explain well, and language models do not play well. The agent joins verbal reasoning to tools that supply chess facts.
- Steinfurt says Gemini 3.1 Pro shows stronger chess understanding in its reasoning traces than earlier models; his guess that this reflects chess-specific post-training is speculation, not a disclosed detail.
- Tools provide legal moves, board state, engine evaluation, checks-captures-threats, and historical context via web search.
- The board is a working environment, not a position pasted into a prompt: the model can test an idea, inspect the result, and return to an earlier branch.
- An architecture slide places this model-and-tools loop alongside LangGraph / Deepagents.

### Explain the moves a person might consider · 06:22
- The checks-captures-threats tool turns a classic beginner's scanning habit into something the agent can request.
- In the shown position, the agent gets more than the engine's best move; it can inspect forcing candidates and test why they succeed or fail.
- Among the checks, a queen taking a bishop on a4 is clearly bad, while other queen checks are reasonable; the actual best move is a rook sacrifice on e3.
- In other positions, captures may be the more productive candidates.
- Returning several possibilities from one tool broadens exploration because a lesson sometimes has to explain a bad move a human would be tempted to play.

### Move the investigation into the agent · 08:03
- The first version put most analysis in Python scripts that assembled checks and engine evaluations, then handed a bundle to a model for description.
- Reasoning models changed that: the agent could decide which positions to investigate and which tools to call.
- In the preceding autumn, Grok 4 had been the team's strongest model for the task.
- Engine analysis, human-move tools, and historical material do not always agree, and Steinfurt finds the conflict useful — it balances the best line against the moves a person might consider.

### Turn analysis into narration and annotations · 09:46
- Findings become the intermediate format, then speech via ElevenLabs v3 text-to-speech.
- Audio tags can steer delivery — for example making a passage sound excited.
- The agent also chooses squares to highlight, arrows to draw, and whether to label a move brilliant.
- A small illustrative record shows narration and annotations kept together for one move, pairing the spoken line with the highlighted squares and arrows.

### Scaling, costs, errors, and human moves · 12:09
- The channel has not reached YouTube's monetization stage, so it earns nothing and is net negative for now.
- Automation was enabled only a couple of weeks before the talk; Steinfurt initially still watched videos before publishing.
- He estimates roughly every twentieth video has a very strange description (such as a missed checkmate), with no formal sample or evaluation behind that number.
- Such errors often point to a missing final tool call, which makes them useful debugging evidence; he has become more willing to publish automatically and remove a bad video afterward.
- Cost is roughly 20–30 cents per video, with much longer videos sometimes reaching euros; the team is not optimizing aggressively because it prefers a strong explanation.
- On "human" moves, he points to **Maia** as a way to get plausible moves for a rating; exact imitation is unnecessary, and rating-aware information could also help target videos at different audiences.
- Asked about other games: not tried, though he thinks it could extend.

## Resources

- [Maia chess models](https://github.com/CSSLab/maia-chess)
- [Maia-2 implementation](https://github.com/CSSLab/maia2)
- [Gemini 3.1 Pro model card](https://deepmind.google/models/model-cards/gemini-3-1-pro/)
- [Eleven v3 audio tags](https://elevenlabs.io/docs/help-center/product/core-capabilities/text-to-speech/how-do-audio-tags-work-with-eleven-v3-alpha)
- [Lichess open database](https://database.lichess.org/)
