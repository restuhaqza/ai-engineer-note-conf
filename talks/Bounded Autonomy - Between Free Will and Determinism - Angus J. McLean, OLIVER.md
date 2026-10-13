# Bounded Autonomy: Between Free Will and Determinism

| Field | Details |
| --- | --- |
| **Speaker** | Angus J. McLean |
| **Affiliation** | OLIVER |
| **Type** | Talk |
| **Date** | 2026-05-25 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=t4359sKBu4w) |
| **Tags** | `agent-engineering` `creative-and-generative-media` `rag-context-and-search` |

## Summary

Angus J. McLean, AI director at the advertising company OLIVER, argues that useful agents need bounded autonomy rather than unconstrained complexity. His evidence comes from advertising that actually runs: OLIVER produces around 4,000 creative assets a day for more than 200 brands, and puts real media spend behind them, so performance comes back from the market.

The talk is deliberately not prescriptive. McLean offers "conventional wisdom for unconventional times," aimed at people experimenting with agents. His first and most counterintuitive recommendation is to slow down: tools come and go, but the underlying limitations of language models are durable and deserve more attention.

From there he builds practical guidance — treat a model as a closed box that never learns during use, curate the evidence it sees instead of giving it open internet access, and use constraints to force creativity. He ends with a demo that clusters 50,000 posts into a strategy report an advertising team could act on.

## Key Takeaways

- **Generation only pays off in a loop.** Real media spend turns assets into measurable outcomes, so teams can iterate on what works.
- **Slow down.** Tool churn is noise; the core limitations of LLMs are long-standing, so design for them rather than chasing every new tool.
- **Treat the model as a closed box.** It does not continuously learn, so genuinely new trends may be missing from its knowledge.
- **Context is a soft constraint.** What you feed in and how you arrange it steers behavior without being a hard limit.
- **Curate the evidence.** Replacing open internet access with high-quality documentation beats broad access; models are weak at spotting promotional content and are susceptible to SEO.
- **Constraints create creativity.** Ask how little context completes the task, not how much the model supports.
- **Build the simple version.** A model's verbose, complex suggestions are often worse than a plain, direct approach.
- **Pick the right representation.** The same information can live as Markdown, graphs, clusters, folders, or timelines — choose what the task needs.

## Detailed Notes

### Advertising that has to work in the wild · 00:16
- McLean introduces himself as AI director at OLIVER, an advertising business that has moved heavily into generative AI, with 3,000 staff across 46 countries.
- He shows AI-generated work for Johnnie Walker: an audience can encounter it without knowing how it was made.
- The business problem is bigger than a good image — the image has to function as advertising.
- He reports around 4,000 assets a day for more than 200 brands, with media spend from tens of thousands to a few million (currency not specified).
- Because these assets run as paid media, performance data returns from real exposure, giving a strong feedback loop for faster iteration.
- Generation becomes useful mainly through that loop.

### Speed first, scale second · 02:14
- He sketches an agency through its three parts: accounts (client relationships and keeping projects on track), creative (turning ideas into ads), and strategy (the insight, audience, and direction).
- His rough split is about 50% accounts, 20% creative, 20% strategy — a sketch with mixed time references, not a precise accounting.
- Creative and strategy are increasingly agentic knowledge work: ideation, copywriting, and content production on one side; audience insight, trend analysis, competitor analysis, and performance optimization on the other.
- McLean works on the strategy side. Because these applications face customers, speed carries risk: scaling content can amplify a bad reception as easily as a good one.
- The main reason to use agents is speed; scale comes second.
- Campaign personalization connects the two: define personas or territories, research each one, and use that research to shape the work. Localizing for New York and Miami needs more than extra copies of one asset.
- The goal is more effective advertising with fewer resources — which leads to his first recommendation: slow down.

### Slow down enough to see what persists · 04:29
- McLean asks whether a tool you "blinked and missed" was really important, and argues durable limitations deserve more attention than short-lived tools.
- He places the foundations of LLMs at least as far back as the 1990s, and says apparent sophistication should not be mistaken for understanding.
- Two persistent limitations matter for design. **Data efficiency:** humans learn from few examples, while models need massive datasets. **Continual learning:** models do not keep learning without forgetting the way people do.
- He attributes some recent gains to brute-force compute, invokes The Bitter Lesson, and references a previous talk's image-generation figure of "400 marathons" — an analogy, not a defined cost.
- Against those constraints, many tools are band-aids: temporary, quick fixes that can mask symptoms or only partly solve a problem.
- The useful engineering question is what a surrounding system actually fixes — and what it merely makes less visible.

### A closed model needs working context · 06:17
- McLean's simplifying model is a closed box of knowledge, or a flexible database capable of "semantic math." He does not expect it to spontaneously learn new capabilities.
- In advertising this bites on trend identification: if a trend is genuinely new, the model will not recognize it, just when a strategist most needs it.
- Larger context windows (how much information a model can work with at once) explain much of the progress in long-running agents.
- That context holds the history of actions, tool outputs, task structure, goals, and plans — short-term storage and retrieval the agent uses to decide what to do next.
- Without enough context, the agent can forget mid-task and lose continuity; he cites an unspecified email-deletion incident as a warning.
- He compares GPT-2 with a model named aloud as Gemini 3.5 Pro and recalls 512-token windows, though the details do not establish a reliable capacity comparison.
- His broader point is that working space is finite and demand will always outgrow it, so you must decide what belongs inside the window. (He illustrates this with a claim that world knowledge doubles every 12 hours, which is unsupported.)

### Constrain the evidence, cut the noise · 07:49
- McLean treats context as a soft constraint: what the agent receives, and how it is arranged, steers behavior without enforcing a hard limit.
- His everyday example is replacing broad internet access with high-quality documentation. Restricting access is a capability boundary; choosing documentation shapes the evidence inside it.
- Competitor research shows the failure mode: an agent seeking consumer insight can absorb the competitor's own promotional language instead.
- Search visibility does not make a source representative of consumers, and he finds models susceptible to SEO and poor at recognizing promotional material.
- Once knowledge is retrieved through a tool, the result depends on how the agent uses the tool as well as what the model knows.
- With small windows he used TF-IDF (a method that weights words by how distinctive they are across documents) for cluster labeling, plus Top-K selection to limit what enters context.
- With larger windows, getting information in is easier, but keeping irrelevant information out becomes more important. The target is useful context, not a full context window.

### Practice with less · 09:28
- His question turns scarcity into a design exercise: if model progress stopped tomorrow, what could you still build?
- Instead of asking how much context a model supports, ask how little context completes the task.
- He draws parallels with early computing — Spacewar! built with only 4,000 words (the talk does not say whether that is program size or memory), and Crash Bandicoot's memory optimization for the original PlayStation.
- The relevant lesson is the engineering response to limited memory: decide what must be kept resident and what can be brought in when needed.
- He offers experiments, explicitly separate from production advice: try an older or smaller model, or a simpler harness, to make limitations easier to observe.
- Build a harness and implement memory and compaction, so decisions about retaining and discarding information become explicit.
- Work on preprocessing and archiving; study how file systems and knowledge graphs organize information for later use.
- The payoff is better prompting and control, backed by a closer understanding of the data — and, he quips, he is not yet a "token billionaire."

### Keep it simple; pick the right representation · 11:24
- McLean describes building a complex CV application that worked, but less well than simply asking for the result in HTML — an improvement he estimates at more than 10×, perhaps 100×, without a measured metric.
- The comparison is about implementation choice: a direct document format solved his problem better than the surrounding machinery.
- Models make unnecessary complexity easy: he calls them verbose and inclined toward the most complicated solution, which wastes tokens and creates maintenance work.
- His rule is to build a simple version that works, and to shorten the feedback loop with reality, not just with agents.
- He frames AI at its core as translation. The Transformer paper (Attention Is All You Need) started with machine translation, and the idea extends to text-to-image, image-to-audio, and audio-to-video.
- Data moves between representations: unstructured input to structured output, or structured input to prose. He connects this to MCP and handoffs in long-running agents, while noting MCP's scope is connecting AI apps to external data and tools, not general model-to-model handoff.
- Useful structure depends on the observer: slides make this tangible, since some material suits a diagram, some needs prose, and some is better voiced over.
- Representations complement each other — Markdown or outlines for hierarchy, graphs for relationships, clusters for large text collections, folders for fast retrieval, and timelines when sequence matters. His example is a research note that separates a finding from its evidence and implication.

### From play to a usable intelligence report · 14:49
- McLean says much of this understanding comes from thoughtful play, and he attends more hackathons now to experiment in ways his job does not allow.
- The closing question is how much structure agents need at work. He reaches back to Adam Smith's pin factory: work can be broken into small, repeatable tasks, though he has often found workflows more effective.
- His rule: do not automate a job unless you can do it yourself. Domain competence is a prerequisite for knowing how to divide the work and whether the output is useful.
- The final demo returns to his former job in social media intelligence: an LLM-produced report about OpenClaw using roughly the preceding year's data.
- He reports it clusters 50,000 tweets and organizes them into strategies, including standout tweets and a breakdown of insights, created almost instantly (no measured runtime given).
- Its destination is a creative or strategy team: a large body of social conversation organized into something people can use to decide what to make next.

## Resources

- **Attention Is All You Need** — The Transformer paper, reporting machine-translation experiments. *(mentioned)* — https://arxiv.org/abs/1706.03762
- **GPT-2 model implementation** — Includes GPT-2's default token context setting. *(mentioned)* — https://github.com/openai/gpt-2/blob/master/src/model.py
- **Andy Gavin on building Crash Bandicoot** — A developer's account of fitting PlayStation game levels into limited memory. *(related)* — https://all-things-andy-gavin.com/tag/poweranimator/
- **Dropout: A Simple Way to Prevent Neural Networks from Overfitting** — The 2014 paper explaining dropout. *(related)*
- **Model Context Protocol introduction** — How MCP connects AI applications to external data, tools, and workflows. *(update)* — https://modelcontextprotocol.io/docs/2026-07-28/getting-started/intro
- **OpenClaw** — The personal AI assistant project used as the subject of the closing social-intelligence report. *(mentioned)* — https://github.com/openclaw/openclaw
