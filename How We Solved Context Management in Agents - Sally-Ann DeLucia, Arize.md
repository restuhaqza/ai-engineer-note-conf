# How We Solved Context Management in Agents

| Field | Details |
| --- | --- |
| **Speaker** | Sally-Ann DeLucia |
| **Affiliation** | Arize |
| **Type** | Talk |
| **Date** | 2026-05-10 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=esY99nYXxR4) |
| **Tags** | `observability-and-reliability` `coding-and-developer-tools` `rag-context-and-search` |

## Summary

Sally-Ann DeLucia is head of product at Arize and a hands-on contributor to
Alyx, the company's agent for building AI applications. Her talk is about a
problem the team kept hitting: the context window, meaning how much text a model
can look at once. She argues that choosing what the model sees matters more than
writing the perfect prompt, and that agents fail because of context more than
prompts.

Alyx works over observability data (records captured from running AI apps), so
its inputs are large and keep growing. The team got stuck in a loop: the agent
analysed traces of its own work, those records grew, the context overflowed, and
every failure added more data for the next attempt.

DeLucia walks through how they escaped that loop: smart truncation, retrievable
memory, long-session evaluations, and sub-agents that hold heavy search work.
She is candid about what is still unsolved, including long-term memory across
chats and a principled way to decide what to keep.

## Key Takeaways

- **Context beats prompts.** Her lesson from nearly a year of failures: agents
  break when the model lacks the right information, not only when instructions
  are weak.
- **Keep a path back.** Remove material from the active window but store it, so
  the agent can retrieve it later. "Context decides what the model sees, memory
  decides what survives."
- **Blunt truncation and raw summarization both failed.** Keeping only the start
  broke follow-ups; letting an LLM summarize was too inconsistent.
- **Smart truncation worked.** Keep the first and last 100 characters, drop the
  middle, and store that middle for retrieval.
- **Test late-session forgetting.** Load ten conversation turns, then test the
  eleventh, so memory loss becomes something you can trigger on purpose.
- **Move heavy data into sub-agents.** The main conversation stays light while a
  search sub-agent holds the bulky queries and returns just the result.
- **Several problems remain open:** provider input limits, no long-term memory
  across chats, and a heuristic rather than principled selection rule.
- **Treat it as iterative.** The three practices she stresses are context
  engineering, memory, and evaluation.

## Detailed Notes

### What should an agent remember? · 00:15
- **Sally-Ann DeLucia** is head of product at **Arize** and a core contributor
  to **Alyx**, which she calls an "AI harness" (a tool for building AI apps).
- Alyx had close to a year of work, advanced planning, and 40-plus skills:
  prompt optimization, data generation, augmentation, and annotation.
- "Context engineering" became a popular term in mid-2025; she credits an Andrej
  Karpathy post for pushing it ahead of prompt engineering.
- Her rule of thumb: a good strategy helps the agent remember what it needs and
  forget what it doesn't.
- Context management is not just filling the token window. It means choosing
  strategically what the model sees.
- Alyx runs on Arize's observability platform, so its inputs are **traces** (a
  record of a request) and **spans** (one operation inside a trace). A single
  trace already holds user input, prompts, and metadata.
- Asking about patterns across traces multiplies that load. Missing the right
  evidence produces bad answers, so she frames context as a product and UX
  concern, not just an engineering one.

### When debugging adds to the failure · 04:06
- The team "built Alyx using Alyx," as evidence it could help real users.
- That created a feedback loop: Alyx analysed a trace of its own activity,
  growing spans pushed past the context limit, and Alyx failed.
- The failure was then recorded as more data, so the retry hit an even larger
  record and failed again.
- The system meant to explain the data was constrained by the size of that same
  data.
- Retrying did not remove the cause; it added to it.
- Three related changes broke the loop: control the active context, separate
  context from memory (designing both together), and move heavy work out of the
  main agent.

### Truncation and summarization lose different things · 05:16
- **First try: blunt truncation.** Keep the first 100 characters and drop the
  rest.
- It looked adequate for simple interactions, but follow-ups failed. After a
  question about the most common inputs, a request for more detail about one
  item was not understood.
- The earlier reference was gone, so the follow-up looked like a brand-new
  conversation. The issue was not shorter answers; over-truncation removed
  information needed to reason across turns.
- **Second try: LLM summarization.** Compress the context into fewer tokens and
  pass the summary forward.
- It sounded like the obvious solution, but retention of important details was
  inconsistent: the summarizer decided what mattered with too little control
  from the surrounding system.
- She is careful that this shows their experiment failed, not that
  summarization never works.

### Remove information from view without destroying it · 06:46
- The deployed approach combines truncation with retrievable memory. Keep the
  first 100 and last 100 characters, remove the middle from the active context,
  and store that middle.
- A small TypeScript function makes the boundary concrete: it returns the
  visible excerpt (`visible`) separately from the part to store (`omitted`), and
  short inputs stay whole.
- Duplicate messages are handled, and for long tool output the latest result is
  kept.
- The system prompt is preserved, not reset.
- The agent can retrieve earlier tool results or conversation messages when they
  become relevant.
- Her distinction: "context decides what the model sees, memory decides what
  survives."
- This combination worked well enough to stay unchanged for several months,
  though the team was starting to revisit it.

### Test the turn where forgetting appears · 07:57
- Users often keep one chat open while moving between pages, so conversations
  grow and failures appear late.
- At first, the team found these failures through user reports or by manually
  inspecting conversation data.
- Their **long-session evaluation** loads ten conversation turns and then tests
  the eleventh.
- This turns late-session forgetting into something they can exercise on
  purpose instead of waiting for a complaint.
- It is a test procedure, not a claim that eleven turns is the system's limit or
  that every such test passes.

### Keep search's working data outside the main conversation · 09:12
- A search over Arize data can involve hundreds of spans, multiple queries,
  large results, and intermediate reasoning.
- That work helps perform the search, but it does not all need to stay in the
  user's main conversation.
- **Before:** one agent held both the chat history and the heavy search data.
- **After:** the main agent keeps the chat and light context, delegates the
  search, and receives just the result.
- The result handoff is the boundary: the main conversation continues without
  inheriting the entire search process.
- Memory retrieval stays available for follow-ups, so a concise result does not
  cut off the supporting detail.
- She calls the split a major improvement and says the team later rolled out
  many sub-agents for data-intensive operations.

### The remaining limits: size, memory, and selection · 11:01
- Very large prompts and inputs can still hit provider limits.
- Alyx has an unusual source of pressure: other agents' system prompts, user
  messages, and conversation histories are the very data its users want
  analysed, so its context problem grows along with theirs.
- Conversations have grown from under ten turns to twenty-plus. That is a usage
  observation, not a measured success rate.
- Long-term memory is missing: the store holds retrievable conversation context,
  but a new chat cannot recall previously discussed issues. That work was
  underway.
- Context selection is still a heuristic. Keeping the first and last hundred
  characters is not proof those are the most useful ones.
- The team has no principled context budget and no direct metric for context
  quality; it uses task evaluations as an indirect signal.
- She noticed that publicly exposed Claude Code appears to use a similar
  truncation and compression strategy. That is her observation, not proof the
  systems are equivalent.

### Database retrieval does not settle cache preservation · 14:59
- An audience question asks whether context management can preserve the prompt
  cache (reusing unchanged prompt prefixes to save cost and latency).
- DeLucia says the team had done relatively little cache work and was
  prioritising long-term memory.
- Stored material lives in a database with IDs.
- Alyx has a tool that exposes those IDs, where the material sits in the
  conversation, message counts, and a short preview.
- That is enough for the agent to locate and request earlier context without
  carrying all of it in the active window.
- More sophisticated handling is on the roadmap, but the current retrieval
  works. Long-term memory got priority because that is where she heard the most
  complaints.

## Resources

- *Managing Memory in AI Agents: Beyond the Context Window* — Arize article.
- *Alyx 2.0: The AI Agent That Actually Plans* — Arize article.
- *How we use Alyx to build Alyx* — Arize article.
- *Alyx getting started* — product documentation.
- *Effective context engineering for AI agents* — Anthropic article.
- *How Claude Code uses prompt caching* — Anthropic documentation.
