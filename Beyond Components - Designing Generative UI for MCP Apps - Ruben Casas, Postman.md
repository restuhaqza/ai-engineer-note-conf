# Beyond Components: Designing Generative UI for MCP Apps

| Field | Details |
| --- | --- |
| **Speaker** | Ruben Casas |
| **Affiliation** | Postman |
| **Type** | Talk |
| **Date** | 2026-06-03 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=hCMrEfPG2Yg) |
| **Tags** | `apis-mcp-and-protocols` `enterprise` `agent-engineering` `safety-and-governance` `developer-workflows-and-testing` |

## Summary

Models have become good enough to write high-fidelity frontend code quickly — sometimes making design choices the human never asked for. So why, Ruben Casas asks, are agent products still mostly text conversations and static interfaces? His answer is that we are early: we have a powerful new kind of computer but no mature interface language for it yet.

Casas, a staff engineer at Postman, lays out a progression of how agent interfaces get built. Static components let an agent feed data into developer-made UI. Declarative UI has the model emit a descriptor — JSON, YAML, or Python — that a rendering engine turns into a layout from approved components. Generative components go further and let the model write HTML, CSS, and JavaScript at runtime.

That last step creates a trust boundary: if we do not blindly trust third-party code, we should not blindly trust model-generated code either. **MCP apps** (applications delivered through the Model Context Protocol) provide the containment — sandboxing, authentication, tool calling, and messaging between interface and agent — needed to ship generated UI safely. His forward-looking idea is less about widgets and more about shared artifacts that a person and an agent edit together.

## Key Takeaways

- **The interface lags the model.** Models can produce strong UI code, but agent products still lean on chat and static screens because no settled interface language exists yet.
- **Separate two questions.** *Where the UI runs* (chat everywhere vs. a super app) is different from *what the model generates* (props, a layout descriptor, or executable code).
- **Static components** let an agent orchestrate a tool call and pass props into developer-built UI — e.g. the AG UI protocol or Goose's auto visualizer.
- **Declarative UI** has the model emit a descriptor that a rendering engine maps to approved components, preserving the design system while adding personalization.
- **Declarative is a strong balance today** — flexible and predictable, and potentially faster/cheaper than generating a whole interface from scratch.
- **Generative components** let the model write HTML/CSS/JS at runtime; Casas's weather-agent experiment generated both a joke and the whole interface in a single tool call.
- **Containment is not optional.** Distributing model-generated UI requires a boundary and a sandbox; MCP apps provide this by default through a double iframe.
- **The future may be collaboration, not visualization.** The Excalidraw MCP App shows a shared canvas a human and agent both edit — a hint of where interfaces could go.

## Detailed Notes

### The interface has not caught up with the model · 00:15
- Casas recalls the 2022 workflow: ask ChatGPT for a component, copy and paste, fix it, repeat — "the poor man's vibe coding."
- Late 2025 brought an inflection point, partly from GPT-5.2 and Opus 4.5, which were strong at long-horizon tasks and high-fidelity UI generation — fast, and sometimes with real design judgment.
- His own example: a single prompt to rewrite his blog produced a search box, a blur animation, and accessibility features he had not requested. The significance is not just more code but decisions beyond what the operator specified.
- That raises the puzzle: if models write UI this well, why are agent products mostly static and text-based? Casas frames current interaction as direct access to a powerful new computer whose mature graphical interface has yet to be invented.

### Separate where the interface runs from what the model creates · 03:57
- Casas contrasts two deployment patterns: putting chat inside many existing apps, versus concentrating interaction in an agent-centered super app (ChatGPT, Claude, Gemini) where MCP apps render third-party interfaces inside the shared environment.
- He treats both as plausible transitional arrangements and says he does not know which consumers will prefer.
- The more consequential question is separate: **what is the model actually generating?** Delivery location does not determine whether the model supplies props, assembles a layout, or writes executable code.
- Keeping these apart lets you compare interface-generation strategies without confusing location with capability.
- In the **static component** pattern, developers build components in advance, the agent orchestrates a tool call, and the client renders a predefined component — like familiar client-rendered UI, except the agent produces the inputs.

### Static components in practice · 06:17
- The AG UI protocol offers an SDK that maps a registered client tool to a React component, passing tool-call properties into it.
- Goose, an MCP client, includes Goose auto visualizer: pass it data and it organizes that data and matches it to components the Goose team created beforehand.
- This is the most common way of generating UI today and looks like the client-server rendering model of the past twenty years.
- The difference is that an agent generates or selects the data and props rather than a traditional server.

### Declarative UI expands flexibility without abandoning constraints · 07:29
- Declarative UI keeps developer-built components and the design system but changes what the agent produces.
- Instead of only supplying props, the agent emits a descriptor — JSON, YAML, or a Python representation associated with FastMCP2 — that a translation/rendering engine turns into the final interface from approved components.
- The gain is more dynamic composition and personalization within established visual and functional boundaries.
- Casas compares it to Netflix-style server-driven personalization: different people get different homepage arrangements, but always from the same Netflix UI elements.
- He points to JSON Render (built by Vercel) as an example of mapping components through JSON and YAML. The key limitation is deliberate: the model generates the descriptor, not the components.
- He calls declarative generation a strong balance for now — it keeps design-system consistency and predictable output while being faster and potentially cheaper, since less is spent generating an entire interface.

### Runtime-generated components introduce a new trust boundary · 09:35
- The next step is **generative components**: letting a model write frontend code on demand at runtime rather than constraining output to a descriptor.
- An agent can call the same model via reverse sampling or call another model to produce HTML, CSS, and JavaScript for delivery to the client.
- Casas's Postman experiment built a weather agent that, in one tool call, hits a weather API, creates a joke, and generates the HTML, CSS, and JavaScript for an imaginative interface — no component catalog, no translation layer.
- The freedom brings a serious limitation: if application developers do not automatically trust third-party code, they should not automatically trust model-generated code shown directly to users.
- So runtime-generated interfaces need a delivery architecture with an explicit boundary, containment, and sandboxing.

### MCP apps deliver containment · 11:41
- Casas identifies **MCP apps** as the best delivery mechanism for generated UI, because the protocol supplies authentication, tool calling, and message passing between interface and agent.
- MCP apps are sandboxed by default with a double iframe; Casas treats containment as a prerequisite for distributing dynamic UI, not an optional add-on.
- He notes MCP apps are already the default for third-party UI delivery, so the standard is largely settled.
- Although MCP apps commonly deliver third-party UI into an agent environment, the same architecture works for first-party UI. He cites Anthropic's visualizer feature as choosing MCP apps over building a separate rendering and delivery mechanism — a signal that teams can reuse the same protocol features internally.
- If a first-party provider uses MCP apps for its own UI, Casas asks why others cannot do the same; he calls it a very strong protocol, especially when UI is generated on the fly by coding models.

### Beyond components: toward shared human-agent work · 14:28
- His longer-term proposal moves beyond generated widgets toward **human-agent collaboration** around shared artifacts.
- The Excalidraw MCP App illustrates this: a shared canvas where a person can request changes conversationally while also clicking and editing directly — a two-way workspace, not a one-way visualization.
- Casas says the future is explicitly uncertain and speculative. Chat, MCP apps, floating windows, and familiar component metaphors may all be intermediate forms rather than the finished interaction model.
- The "Jarvis" vision — floating windows that appear and disappear — is probably too obvious; he suspects we simply do not have enough imagination yet.
- He closes with a TV analogy: early television just put cameras in front of radio shows because nobody could yet imagine what the new medium could do. We are likely in that same early phase with agents.
- His strongest prediction is narrower: generative interfaces will become more personalized and collaborative as builders discover patterns that use agent capabilities more fully.
- His parting point is that we are still early and do not have the answer, but we can shape that future and help build this new computer.
