# Teaching Coding Agents to Work with Spreadsheets

| Field | Details |
| --- | --- |
| **Speaker** | Nuno Campos |
| **Affiliation** | Witan Labs |
| **Type** | Talk |
| **Date** | 2026-07-08 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=HEFSExa0xl0) |
| **Tags** | `agent-engineering` `finance` `developer-workflows-and-testing` `vision-and-video` `coding-and-developer-tools` |

## Summary

Nuno Campos describes four months spent making coding agents as capable with spreadsheets as they are with Python or JavaScript. Answers like "what's the revenue?" are deceptively hard: the agent must work out which revenue, which period, and whether a cell is an input or a formula. Witan Labs raised accuracy on an internal financial-analysis benchmark from about 50% to 92%.

Several ideas failed first. A three-agent design with a five-step edit process made errors land in the plan instead of the build, but it was too rigid. SQL, XML, CSV/TSV, and HTML representations each had appeal, yet none worked as the sole interface. The biggest win was replacing roughly fifteen separate tools with a single **Node.js REPL** (read-eval-print loop): the operations became JavaScript functions the agent could compose in one call.

Two follow-on pieces mattered. A **persistent REPL** keeps variables alive between calls, so the agent can run short scripts, reason about the result, then continue — unlike stateless Code Mode. And because producing a plausible formula is not the same as confirming the workbook works, the team built a **formula calculation engine** and a **range render engine** to give the agent a write–calculate–render feedback loop.

## Key Takeaways

- Spreadsheet understanding needs **structure and computation**, not just values; the same UI that makes a workbook obvious to a human hides it from a model.
- A multi-agent, plan-first architecture reduced some errors but failed because discovery happened once and context could not flow back and forth.
- **SQL, XML, CSV/TSV, and HTML** all fell short as standalone representations; CSV/TSV views and HTML-style layout survived as capabilities inside the final system.
- Replacing ~15 tools with one **Node.js REPL** let the agent compose operations as JavaScript in a single call — a gain in composition, not just concurrency.
- Unlike stateless **Code Mode**, a REPL persists variables between calls, so the agent writes shorter scripts and interleaves more reasoning.
- New capabilities became new JavaScript methods documented by a **TypeScript type definition file** placed in the prompt.
- Verification needs high-fidelity engines: an incomplete formula engine turns correct formulas into apparent failures and can make the agent **worse**.
- Prefer **deterministic evaluation** (comparing outputs against a golden spreadsheet) over an LLM judge where possible; inspect traces, because many "reasoning" failures are actually tool or prompt bugs.

## Detailed Notes

### What's the revenue? · 00:16
- The goal was to get coding agents as good at spreadsheets as at Python or JavaScript.
- Opening an unknown workbook is a visual task for a person: you instantly see a revenue table, an assumptions area, and a chart. A model sees none of that structure.
- Even after finding revenue, the agent must tell gross from net, pick the right quarter or year, and know whether a cell is an input or a formula.
- These are distinct uncertainties: what the user means, where the answer lives, and how the workbook derives it.

### Planning helped; rigid discovery did not · 01:53
- An early design split work across three agents, centered on an edit agent with a five-step process (define the end state, plan, execute, verify).
- This shifted mistakes from building a model into the planning stage, where they were easier to fix.
- But discovery ran only once up front, so the system could not return to explore when editing raised a new question, and context failed to flow between agents.
- The team then tried every representation they could think of. None worked as a standalone interface:
  - SQL: familiar for structured data, but insufficient alone.
  - XML: Excel's on-disk format, ineffective as the interface.
  - CSV/TSV: compact views, frequently useful inside a larger system.
  - HTML: surfaced layout and formatting, which led toward image rendering.

### Turn tool calls into composable JavaScript · 04:21
- The biggest change was replacing about fifteen tools with a single **Node.js REPL** tool.
- The operations did not vanish — they became JavaScript functions the agent could combine inside one call, expressing orchestration as code instead of many tool exchanges.
- JavaScript was chosen because it is easy to sandbox and models know it well; Campos thinks Python would work equally well.
- The spreadsheet work itself stayed in C#: the scripting language serves the agent, while a different language handles the files.
- Before, exploring a spreadsheet to an answer often took ten to fifteen sequential tool calls, which was slow and often timed out. Parallel calling did not fully help because results could not be combined within the calls.

### Keep state between execution steps · 06:08
- This composition pattern is often called **Code Mode** — generated code combining several tool operations in one call.
- A REPL goes further by keeping state: variables declared in one call are still available in the next, so the agent can pause to interpret results and keep building.
- Without persistence, agents commonly wrote ~50-line scripts, committing to a long sequence before seeing anything intermediate. With it, scripts got shorter and reasoning was interleaved more often, which often produced a better answer faster.
- Adding capability also simplified: instead of new tool schemas, it meant exposing more JavaScript methods, documented by a **TypeScript type definition file** in the prompt.

### The gains and their evaluation conditions · 08:21
- Accuracy moved from roughly 50% before the REPL, to 74% after, and eventually 92% after further changes.
- The later gains came from better fuzzy search, formula-dependency tracing, system-prompt improvements, and bug fixes, not the REPL alone.
- The progression reflects the team's engineering results rather than a controlled test of one change; the research log uses changing task sets.
- Under a usual five-minute whole-task limit, this approach produced essentially zero timeouts, because the agent spent less of its budget on separate exchanges.

### Calculate and render before trusting the result · 09:28
- Coding agents improve when they can run a compiler, linter, or test suite and react; spreadsheet agents need the same feedback.
- Two engines supply it: a **formula calculation engine** to compute formulas, and a **range render engine** to produce an image of a selected range with its formatting.
- Together they enable a write–calculate–render loop: make a change, observe the numerical and visual results, and return to fix the formula or formatting.
- Fidelity is critical. If the engine supports only half of Excel's formulas, a correct formula can come back as an error, and the agent may "repair" something that was never broken.

### Compare behavior, not just judged answers · 13:29
- Evaluation was a lot of work and was essential to tell whether a representation actually helped.
- The team first relied entirely on an LLM judge, which supplies a score but adds another source of variation: when a score changes, was it the agent or the evaluator?
- Where possible they used deterministic comparison against a **golden spreadsheet**: put values into the reference inputs, read its outputs, put the same values into the generated workbook's inputs, and compare the outputs.
- This treats each workbook as a black box, so construction need not match; LLM grading is still kept for some text answer tasks.

### Closing lessons: composition, feedback, and trace-checking · 15:08
- Many apparent reasoning failures are actually infrastructure bugs: a wrong example in a skill or prompt that the model follows faithfully, or a broken tool it keeps retrying around.
- Inspecting the trace — what the agent was told, what it called, and what the tool returned — separates a model mistake from a defect the team can fix.
- General lessons: if an agent makes many sequential or parallel tool calls, you have effectively built a bad scripting language, so give it a real one (Code Mode or a REPL).
- Feedback loops matter: use existing domain tools where they exist, and invest in a calculation or rendering engine where they do not.
- Interfaces deserve deliberate work but will need revisiting as models change; planning and domain guidance still help; prefer deterministic evaluation, and always check traces and plumbing.

## Resources

- [Witan research log — Teaching Machines to Read Spreadsheets](https://github.com/witanlabs/research-log)
- [The REPL Tool](https://github.com/witanlabs/research-log/blob/main/06-repl-tool.md)
- [Benchmark Framework Evolution](https://github.com/witanlabs/research-log/blob/main/05-benchmark-evolution.md)
