# Agents on the Canvas in tldraw

| Field | Details |
| --- | --- |
| **Speaker** | Steve Ruiz |
| **Affiliation** | tldraw |
| **Type** | Talk |
| **Date** | 2026-05-01 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=sPUjIBH5Cwg) |
| **Tags** | `apis-mcp-and-protocols` `enterprise` `developer-workflows-and-testing` `agent-engineering` |

## Summary

Steve Ruiz works at tldraw, a London whiteboard company with a React-based infinite-canvas SDK (a software development kit others build products on). His talk traces a line of AI experiments on that canvas, from **Make Real** (2023), which turned a sketch into a working prototype, to **Fairies**, small animated agents that live and work directly on the canvas.

The through-line is a gap between generating an artifact and collaborating on one. Make Real could produce HTML from a drawing, and you could annotate the result to request changes. Later experiments went further: the model emits structured shapes rather than a flat image, and an agent loop iterates until a task looks done. But Ruiz kept feeling like he was "handing his keyboard to an AI" rather than working alongside one.

Fairies answers that by putting agents into the shared space. You can see where each one is acting, they can observe one another's work, and a leader can coordinate a group. He also pushes into riskier ground: an Electron wrapper that lets agents run code against the live editor, which grants arbitrary code execution. He is explicit that this is a toy for a local, file-based desktop app — not something to ship — and that more access expands what an agent *can* do without ensuring it does the *right* thing.

## Key Takeaways

- **tldraw is three things:** a free whiteboard, a startup, and an SDK. Its canvas and its contents are React components, so it is a place to build new behavior.
- **Make Real was early:** draw an interface, send it to a vision model, get a working prototype. It predates Lovable and the term "vibe coding".
- **Annotate the result to revise it.** The rendered output plus your marks become the next prompt, replacing prose descriptions of visual details.
- **Structured output beats images.** The model emits editable shapes (circles, etc.) from the same vocabulary the human uses, not a raster image.
- **Geometry is harder than semantics.** The model can recognize a candle but often applies the wrong coordinate convention or reference frame.
- **Agents on the canvas surface state.** You see each agent's position and action relative to others, instead of watching terminal windows.
- **Shared state is the hard problem.** Agents are effectively blind while working, so their efforts can overlap; the demo did not show a locking protocol.
- **More access is not more alignment.** Given runtime access, the agent eagerly modified the software but put the change in the wrong place.

## Detailed Notes

### What can a programmable whiteboard become? · 00:22
- Ruiz introduces **tldraw** three ways: a free online whiteboard (he uses it for his slides), a London startup, and an **SDK** for building other products.
- Integrations he names: Replit's Agent canvas and Luba AI's canvas are built with tldraw; Google Stitch's annotation mode uses it, though not the main canvas.
- Building experiments is part of his job improving the SDK. The canvas runtime is built with React, and the canvas contents are React components too.
- That makes the whiteboard a surface for new behavior — including interfaces where AI reads drawings, produces artifacts, and eventually works *alongside* the person drawing.

### A sketch becomes a working prototype · 01:58
- **Make Real** starts with a direct move: draw an interface on the canvas, send it to a vision model, ask for a working prototype.
- He places it in 2023, when vision models were newly available, before Lovable and before "vibe coding" was a term.
- Its appeal: someone who does not code could make a technical artifact without inspecting its implementation.
- The live demo is simple ("make this interactive") and outputs working HTML, which he immediately notes is "not what I asked for" — a recurring theme of honest demo failure.

### The result becomes part of the next prompt · 03:46
- The revision interaction matters more than the first generation. He annotates the rendered result, asks for green, then adds red and black color samples with an instruction to use them.
- The existing website plus the marks around it jointly become the **revision prompt**. You point at what to change instead of translating every visual detail into words.
- He connects this pattern to Google Stitch. But the live revision does not satisfy him: a green button appears, yet he judges that the changes were not followed, and moves on.
- The takeaway is about the interface for expressing revisions — not a claim that every instruction executes reliably.

### Generate editable shapes, then resolve spatial ambiguity · 04:43
- After briefly noting **tldraw Computer** (chains of prompts), he turns to AI as a canvas collaborator. Requests range from "draw a cat" to producing a diagram, finishing slides, or completing a graph.
- Instead of a raster image, the model emits **structured text outputs** that create the same circles and shapes the human can make. The result stays editable in the canvas vocabulary.
- The cat becomes a test of interpretation. He asks it to blow out a candle, then sends a separate request to draw a mouse elsewhere. The app has no concept of a candle, but the model interprets it and folds the action into the drawing.
- Predictable geometry is harder than plausible meaning. He blames less vision training data and conflicting spatial conventions:

  | Convention | Meaning |
  |---|---|
  | Cartesian coordinates | Y increases upward |
  | Browser coordinates | Y increases downward from the top-left origin |
  | Viewer-relative left | Left from the viewer's perspective |
  | Stage left | Left from the performer's perspective |

- A model can recognize an object yet apply the wrong convention when placing it. He first calls this training, then corrects himself: the intervention here was **prompt engineering**.

### Iteration improves the artifact, but still hands over control · 07:34
- The next experiment adds an **agent loop**: asked to draw the life cycle of a butterfly, the agent produces output, reviews it, and iterates until it judges the task complete.
- Its sidebar follows coding-agent conventions (like Cursor): you can inspect reported thinking and work, and reject results. The finished diagram links Egg, Larva (caterpillar), Pupa (chrysalis), and Adult (butterfly).
- It works, but it still feels like handing his keyboard to an AI rather than collaborating.
- The pattern is not useless: he cites Lovart and MagicPath as design apps using tldraw, and tutoring as a strong fit (homework help, filling in worksheets).
- To show contextual completion, he starts a D&D character sheet with a name and fields, asks the agent to finish it, and it assigns him the fighter class.

### Bring the agents onto the canvas · 09:51
- **Fairies** gives agents a visible presence in the workspace: small animated characters you can pick up and throw. Hold one too long and it reacts.
- They accept familiar requests (draw a cat), but their positions and animations expose agent state where the work happens — instead of several terminal windows or a composer interface, you see where each agent acts relative to the others.
- Agents can observe one another. He gives separate Fairies requests to add a hat and a neck to the same cat; both work on the shared drawing concurrently.
- A group request adds orchestration. He selects three Fairies and asks for more animals:

  1. A leader is elected for the group.
  2. The leader scouts the existing canvas.
  3. It creates a to-do list and delegates tasks to the other agents.
  4. It observes the work and judges whether it is correct and complete.

- Shared state is the hard part. Ruiz describes agents as effectively blind while working, risking overlap. The leader supervises rather than draws, but the demo shows no locking or conflict-resolution protocol.
- Larger example: he places a long description of an e-book app on the canvas, summons the Fairies, and asks for wireframes. They keep working as he moves on.

### Runtime access widens the capability—and the risk · 13:35
- Giving agents more access raises a safety question: what is safe to expose in a hackable app?
- tldraw has a runtime API (an interface for controlling the editor in code), and coding models are good at writing against it. But a generic code sandbox is not enough — the agent also needs the DOM (the browser's page structure), rendering, and screenshots to see the effects of its changes.
- His solution: an **Electron** wrapper (Electron packages a web app as a desktop app) around tldraw, plus a small HTTP server. Anything posted to the endpoint is treated as JavaScript and executed.
- That bridges the coding agent to the running editor — and grants arbitrary code execution. He explicitly warns against doing this in a real app and frames the offline, file-based desktop experiment as a risk the individual user accepts. Local ownership is not technical isolation or a safety guarantee.
- One idea he describes but skips demonstrating: a round trip between code and diagram — visualize code, edit the diagram, update the code to match.
- He draws an interface with controls for leg length and T-shirt color and asks the agent to make it interactive. Even though tldraw lacks built-in hover/click primitives here, the agent can write code against the editor to add the behavior.

### Risky power, local files, and the closing demo · 15:58
- While the interactivity request runs, he describes how eagerly agents propose script injection and modifications to local software.
- Anecdote: a teammate, Max, asked Claude to remove podcasts from Spotify; per Ruiz's account, the agent was willing to work through the app's minified code or bundle. This is an anecdote about the agent's willingness, not a demonstrated Spotify mod.
- The live task returns something unexpected: a separate HTML site with a character, color choices, and a long pointer that is "not even a slider". He wanted the behavior *inside* tldraw and redirects the agent back to the canvas. The character also blinks, but that extra behavior does not prove the requested controls work.
- His narrower result: an agent can enthusiastically change software while misunderstanding where the change should live and what it should do. More access widens the space of possible actions; it does not by itself align them with the request.
- The interactivity attempt never fully succeeded. He first suggests releasing the experiment, then corrects himself — it is already released — and invites people to catch him later for a working demo.
- Preference and motivation: he loves local-first apps and "file over app". A local, file-based app can expose itself to Claude and other agents to script and modify it. Maximizing that agency means letting users accept the risk themselves; he points to **OpenClaw** as an example of that attitude toward sharp tools.
- Work continues. He invites people to try Fairies and find surprising behaviors, and notes they have IRC too, without demonstrating its protocol or architecture.

## Resources

- **Make real, the story so far** (article) — the original sketch-to-website experiment and its history.
- **Make Real** (demo) — turn selected canvas sketches into website prototypes.
- **tldraw Computer** (demo) — build visual workflows from connected components that generate and transform data.
- **tldraw Fairies** (demo) — the public canvas-agent experiment demonstrated in the talk.
- **tldraw SDK** (repo) — React infinite-canvas SDK with custom shapes and runtime editor control.
- **Agent starter kit** (docs) — build an agent that observes canvas content and modifies editable shapes.
- **Introducing tldraw offline** (article) — a later desktop release adding local files and persistent agent-authored content.
- **Electron security** (docs) — security guidance for desktop apps combining browser rendering and Node.
- **OpenClaw security model** (docs) — operator trust, tool permissions, and isolation for a powerful assistant.
