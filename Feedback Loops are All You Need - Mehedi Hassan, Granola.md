# Feedback Loops are All You Need

| Field | Details |
| --- | --- |
| **Speaker** | Mehedi Hassan |
| **Affiliation** | Granola |
| **Type** | Talk |
| **Date** | 2026-05-10 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=ON5LIT0M4do) |
| **Tags** | `architecture` `speech-and-audio` `observability-and-reliability` `rag-context-and-search` `enterprise` |

## Summary

Mehedi Hassan, a product engineer at Granola, argues that shipping AI features well is not about one-shotting a better prompt. It is about building **feedback loops** that let you see what the model did and test the interface around it. Granola is a meeting-notes app that sits in the dock and captures both system and microphone audio to produce a real-time transcript, which users can annotate with their own notes.

He walks through what breaks when a polished-looking AI chat feature meets real users: it cannot produce a to-do list, web search is slow, follow-up emails do not sound like the writer, and "coach me about my meetings" returns football-coach advice. A general chatbot can respond fluently while failing the specific job the product promises. So Granola built its own **tracing tools** to inspect every tool call, the reasoning behind it, search trails, and cost — turning a vague "this output feels off" into a specific failure to fix.

The second half is about the interface. Because Granola is a desktop app, only one instance could run at a time, which made testing variants awkward. The team turned the Electron front end into an online **web shell**, so every pull request produces a preview link. Cursor can then test the preview and drop a screenshot into the PR, giving reviewers and the agent a shared, accessible environment.

His closing metaphor is a tennis rally: the result develops through repeated exchanges. Traces make failures inspectable; previews let people experience proposed changes. Together they replace hoping a feature will work with a stronger basis for believing it will connect with users.

## Key Takeaways

- **One-shotting is easy; fitting a model to a product is hard.** A fluent chatbot can still fail the exact job the product promises.
- **Web search hides real costs.** A "one line" tool can blow up token usage and spend, and provider updates can silently degrade quality with no visibility.
- **Different roles need different outputs.** Sales wants deals, engineering wants action items and blockers, HR wants something else — one prompt rarely serves all.
- **Build tracing your team will actually use.** Expose tool calls, reasoning, search trails, and cost, then tailor the UI so non-engineers can investigate failures without complex queries.
- **One-shotting internal tooling is a good use of LLMs.** The tracing tool was small: wrap the AI SDK calls, save records to a database, build the front end around how people investigate.
- **Make experiments easy to open.** Turning the Electron front end into a web shell gave every pull request a preview link and sped development up.
- **Coding agents can exercise the preview.** After a PR opens, Cursor tests the preview and uploads a screenshot — useful evidence, though not full verification.
- **The answer is a better loop, not a better single generation.** Traces and previews build conviction step by step.

## Detailed Notes

### A summary that notices what you wrote down · 00:15
- Granola captures system and microphone audio for real-time transcription. Hassan appears on a recording of the preceding talk as a demo.
- The differentiator is that users can write their own notes on top of the transcript, so the transcript supplies detail while the summary follows what the user cared about.
- He points to his note about a "twenty percent overlap" and shows how it shifted the summary's emphasis. That figure belongs to the earlier talk, not to a measurement of Granola's accuracy.
- The product goal is best-in-class meeting notes across roles, delivered without getting in the user's way — a philosophy Granola has held since day one.

### A chat box meets specific expectations · 01:52
- Granola's chat can answer questions about one meeting, several meetings, or shared context. He imagines building a version in one shot and shipping it in a fake Granola app.
- Getting the interface working is easy; users immediately expose what it does not understand. Their complaints span different problems:
  - Task completion: it cannot produce a to-do list.
  - Latency: web search is too slow.
  - Personal style: follow-up emails do not sound like the sender.
  - Intent: asking for meeting coaching returns an answer about a football coach.
- The gap is concrete: a general chatbot can be fluent and still fail the particular job. Molding the model to that job is the hard part.

### The operational cost of one-line web search · 02:56
- Adding a search tool may look like a single line, but complex queries can raise token usage, consume context, and increase cost.
- He uses roughly ten pence per web-search-assisted chat as an illustrative cost that becomes hard to sustain at millions of users. He names no model or measured average.
- Quality control is separate: one overnight provider update degraded web search, and the team could see the drop but not what changed inside the provider. Switching providers was the only available response.
- He points to the existence of billion-dollar search companies to stress how much complexity sits behind an apparently simple tool integration.
- The point: making search callable does not make its cost or its quality predictable.

### One meeting, different useful outputs · 04:01
- Even correct information can miss the user's needs. The opening summary suited Hassan, but another role expects a different selection and organization of the same meeting.
- Sales expects a deal focus; engineering expects action items, blockers, and tickets; HR wants something else again.
- One prompt generally cannot serve all of these expectations. The deeper problem is understanding why the model produced a given summary for a given use case — which requires inspecting behavior that otherwise feels opaque.

### Follow the agent from beginning to end · 04:39
- Granola's response was to build its own **tracing tools**, and one-shotting internal software was a good fit here because it cut the build effort.
- The trace exposes individual tool calls end to end, the recorded reasons for them, search trails, reasoning trails, and cost — a sequence, not just a final answer.
- The team tailored both the data structure and the interface to its own investigations. Engineering, product, data, and customer-experience staff all use it; diagnosing a failure should not require writing complex CloudWatch queries.
- The implementation is deliberately small: wrap the AI SDK calls, save records to a database, and build the front end around how people investigate. OpenTelemetry and other providers are options.
- The founder follows an entire agent loop front to back to find what went wrong. Tracing is valuable when it turns dissatisfaction into a specific thing to change.

### Make desktop experiments easy to open · 06:19
- LLMs make it cheaper to build multiple versions of a feature, but variants are useful only if people can try them.
- Granola's desktop setup allowed only one app instance at a time, so testing variants in parallel was awkward. Previously, testing meant installing dependencies and running the Electron app locally, and sharing a change meant asking a colleague to repeat the setup.
- The team turned the Electron front end into an online **web shell**. CI now produces a preview link whenever a pull request opens, so colleagues can open the proposed interface directly; Hassan reports this substantially sped up development.
- The preview also gives an agent somewhere to work: after a PR opens, Cursor tests the preview and uploads a screenshot into the PR. That adds visible testing evidence, though it does not establish that every behavior is verified.

### Separate the renderer from its desktop environment · 07:44
- The web shell relies on a boundary Electron already provides: the **main process** handles native capabilities, and the **renderer process** supplies the web-based front end.
- The two communicate through **IPC** (inter-process communication), which lets the renderer request functionality across that boundary. Granola abstracted these calls so the web environment could use web-standard implementations.
- The architectural point: the consuming code does not need to know whether an implementation crosses Electron IPC or calls a browser API. The same separation was applied to React-facing infrastructure — routing, sessions, and the query layer moved to web standards.
- With those dependencies no longer tied to Electron, the renderer could run as a web app. The payoff is the ability to use several versions of a feature, feel how each behaves, and compare alternatives before deciding what ships — instead of judging only a Figma design.

### Build conviction through iteration · 08:45
- Hassan compares working with an LLM to a tennis rally: the result develops through repeated exchanges.
- Traces make the model's failures inspectable, and accessible previews let people experience proposed changes. Together they build a stronger basis for believing a feature will connect with customers.
- The desired experience can feel like magic to the user without remaining an unexplained black box to the team building it.
- Asked about replacing Electron with Tauri, he says Granola has considered the move several times and tried Tauri, but saw no major performance gains — its main criterion. Electron currently serves the team well, and the practical improvement came from making the renderer portable and testable.
