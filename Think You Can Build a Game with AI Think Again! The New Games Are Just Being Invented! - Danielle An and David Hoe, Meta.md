# Think You Can Build a Game with AI? Think Again! The New Games Are Just Being Invented!

| Field | Details |
| --- | --- |
| **Speaker** | Danielle An, David Hoe |
| **Affiliation** | Meta |
| **Type** | Talk |
| **Date** | 2026-07-08 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=grdoOC1BT1s) |
| **Tags** | `creative-and-generative-media` `reasoning-and-models` |

## Summary

Danielle An (a principal engineer) and David Hoe (a prototyper) at Meta skip conventional slides. Instead, the audience scans a QR code and steers a shared character; each NPC (non-player character) they reach reveals the next part of the talk. A voting system decides which input wins when many people move at once. The presentation itself is a live example of their work leading AI-driven game creation at Meta.

Their argument is that AI has made it easy to produce a *recognizable* game, but that is only the start. Once prompting a platformer or a Tetris-like game is routine, the results start to look alike. Raising the barrier is not enough — what separates a good game from a bad one is still aesthetics, cohesion, playtesting, and above all **taste**: understanding who will enjoy the game and why.

They also cover a newer role for AI. A **runtime LLM** makes decisions while someone plays, rather than only helping build the game. Their multiplayer demo gives four LLM-driven NPCs personalities drawn from player input, and the characters independently scheme — stealing, blocking, and fighting. They close on the hard open problems: nondeterminism across an entire platform, the token economics of running models at scale, and content safety.

## Key Takeaways

- Generative tools remove skill barriers — a coder without art skills or an artist without coding skills can now get unblocked — but they do not remove the need to make a *good* game.
- Prompting a familiar game style makes results converge; the initial delight wears off once everything looks similar.
- **Key art** can anchor an LLM: one strong image guides the style of assets, suggests gameplay, and keeps sessions coherent.
- **Taste** — knowing which players will enjoy the game and why — is the differentiator once polish becomes common.
- **Runtime LLMs** drive NPC decisions during play, producing unscripted interactions (stealing cubes, blocking, kicking) that make matches unique.
- Runtime direction could power a **game master** that adjusts difficulty so friends of different skill can keep playing together.
- AI-assisted parallel work shortens iteration from months to hours or days, buying more time for playtests and reworking the game idea.
- At platform scale, **nondeterminism runs through the whole stack**: a model or prompt change can alter behavior even when the surrounding code does not, raising new debugging and stability questions.

## Detailed Notes

### A game replaces the slide deck · 00:15
- The audience joins via QR code and controls a shared character in a small quest game; each NPC reveals a slide.
- A **voting system** resolves conflicting inputs when many people steer at once, though crowds still make the NPC text hard to read.
- Hoe is a prototyper and An a principal engineer; both lead AI-driven game creation work at Meta.
- They frame the talk for people who play games, want to make them, or wonder how AI will change their craft.

### Recognizable games are only the beginning · 03:01
- By 2026, prompting for a platformer or a Tetris-like game is a familiar experiment; a couple of prompts can produce something recognizable, though it may not work at first.
- The first reaction is delight at seeing a game appear, but the novelty fades: if everyone asks for a Mario-like platformer, the results look alike.
- Producing a game does not yet give it a reason to stand out.

### Lower barriers leave the craft intact · 04:21
- The bigger change is who can build: a programmer who cannot draw or model in 3D can reach for tools like **Nano Banana** (image generation/editing) or **Meshy** (text to 3D), and an artist blocked on code can use AI help.
- These tools remove barriers between disciplines, not the gap between a good and a bad game. Classic demands return:
  - **Aesthetics:** a purple box next to a green platform communicates mechanics but not a visual identity.
  - **Cohesion:** UI, story, and art should feel like one universe.
  - **Playtesting:** real player reactions reveal what works.
- An art director's video shows **key art** as an anchor: iterate until an image sets a direction, use it to guide asset styles and even gameplay, and keep it as context across sessions.
- Once polish is common, An says the next requirement is **taste** — knowing which subset of players will enjoy the game and what they are seeking from play.

### Runtime LLMs turn personalities into competition · 08:23
- A **runtime LLM** acts during play, modifying and directing the experience as it unfolds.
- In a multiplayer demo built in a couple of days, four players assign their NPCs personalities — a thief, an honorable character, a fast one — and the NPCs then pursue one goal: collect the most cubes.
- Personality shapes both how they chase the goal and how they treat rivals; An points to characters stealing other NPCs' cubes, blocking, and kicking, all as unscripted runtime decisions.
- The difference is between scripting each encounter and giving characters direction from which interactions emerge, making matches unique and non-repeatable.
- She attributes the feasibility to faster inference and cheaper models, and says the team is experimenting with more competitive, friendlier, or cooperative variants.

### More iteration, and more personal play · 10:57
- Traditional game production is a linear handoff — design, then art, then modeling, animation, and coding — where upstream changes are costly to revisit.
- AI assistance lets teams work in parallel, with updates and iterations in hours or days instead of months, per An's report.
- The benefit she emphasizes is more time for playtests and for changing the game idea itself, not just faster execution of the original plan.
- Runtime direction could personalize play: An describes being "coordination challenged" in co-op games and sometimes quitting rather than holding friends back.
- A runtime LLM acting as a **game master** could adjust the experience so she keeps contributing — a proposed use, not a demonstrated system.
- A Star Trek-style spoken request that materializes a forest and a bear hunt is placed explicitly in the future; today the goal is discovering which forms of runtime direction help.

### Nondeterminism extends across the platform · 13:53
- At Meta's scale, the ambition is millions of pieces of gaming content, with agentic systems involved from the creator's prompt through runtime decisions to serving, ranking, and delivery.
- That spreads **nondeterminism across the whole stack**. Conventional stability assumes a known codebase you can test and debug.
- In an agentic system, changing a model or a prompt can heavily change behavior even when the application code does not, so a model upgrade becomes a system-level change.
- An leaves debugging and maintaining these systems as open engineering questions and does not prescribe an architecture.

### From a Labubu universe to the next experiment · 15:20
- The long-term vision is a world like the OASIS in *Ready Player One*; An says the industry is at day zero of that possibility.
- The shared controls finally reach a character, and the abstract vision turns personal: An imagines a universe where she can be any Labubu she wants and collect them.
- Advice depends on where a creator is:
  - **First-time creators:** prompt a basic game with Gemini, Manus, or another tool this weekend and expect both failures and pleasant surprises.
  - **Prototype builders:** improve the art, UI, and surprises until it feels deliberately made rather than like default prompt output.
  - **Platform engineers:** work out the token economics so creators, players, and the platform are all viable.
- Runtime content raises **content safety** as part of the playing experience, since a model generating images or content mid-session must keep output appropriate for its audience.
- They end with open challenges across tooling, codebases, quality expectations, economics, and safety.

## Resources

- [Nano Banana image editing announcement](https://blog.google/products-and-platforms/products/gemini/updated-image-editing-model/)
- [Meshy Text to 3D guide](https://help.meshy.ai/en/articles/9996858-how-to-use-meshy-text-to-3d)
- [Create and iterate on prototypes with Gemini Canvas](https://blog.google/products-and-platforms/products/gemini/gemini-collaboration-features/)
