# Ship Real Agents: Hands-On Evals for Agentic Applications

| Field | Details |
| --- | --- |
| **Speaker** | Laurie Voss |
| **Affiliation** | Arize |
| **Type** | Workshop |
| **Date** | 2026-05-14 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=Xfl50508LZM) |
| **Tags** | `evals` `apis-mcp-and-protocols` `coding-and-developer-tools` `safety-and-governance` `rag-context-and-search` |

## Summary

Laurie Voss (head of developer experience at Arize, formerly a co-founder of npm)
runs a hands-on workshop about evaluating an agent, not building one. The running
example is a Claude-powered financial-analysis agent that researches stock tickers
and writes short investment reports. The sequence is: trace what happened, read and
categorize failures, write evals, check the evals themselves, and finally iterate.

His central idea is plain: evals are tests, and traces are the logs that supply
evidence. Because the same prompt can produce many valid answers, exact string
matching fails. So Voss layers three instruments — deterministic code checks, LLM
judges (a model grades against a rubric), and human review — each catching failures
the others miss.

In the demo, an Apple trace shows the agent researching correctly but writing a
Markdown file instead of returning the report. A simple ticker check passes 11 of
13 runs. A built-in correctness judge scores 0/13 because it lacks current
financial data, while a faithfulness check (does the report follow the research)
passes 13/13. A custom "actionability" rubric then flags reports that summarize
facts without a buy, sell or hold recommendation.

Voss stresses meta-evaluation — checking the judge against human labels — and
warns that a small, vaguely labeled sample proves little. He closes with practical
guidance: start small, fix the data before reaching for a bigger model, and treat
release decisions as a statistics question rather than a single pass rate.

## Key Takeaways

- **Evals are tests; traces are logs.** A trace is nested spans (LLM calls, tool calls, agent turns) with inputs, outputs, timing and token counts.
- **The "vibes problem" is real.** Running a few queries and eyeballing them misses edge cases, adversarial input and unexpected user wording.
- **Layer your tools.** Code checks handle explicit rules, LLM judges handle meaning, humans handle new failure modes — the "Swiss cheese" model of safety.
- **Don't prescribe the path.** A request about Tesla once turned into research on Nikola Tesla; evaluate the outcome and constraints, not a fixed tool order.
- **Correctness and faithfulness answer different questions.** The built-in correctness judge failed 0/13 here; faithfulness passed 13/13.
- **Write product-specific rubrics.** Ask for a constrained label like `actionable` / `not_actionable`, not a vague 1–10 score.
- **Meta-evaluate the judge.** Compare its labels with human labels on a meaningful sample, and version evaluator prompts like code.
- **Improve data first.** The impact hierarchy is data quality → prompting → model choice → hyperparameters.
- **Release needs uncertainty.** 6/200 and 12/400 both look like 3%, but their confidence intervals and the interval method differ; shipping decisions need a larger sample.

## Detailed Notes

### Your agent runs — how do you know it works? · 00:40
- The workshop evaluates a prebuilt agent rather than building one. Assume you already have an agent going off the rails.
- The example agent uses the Claude Agent SDK; Claude both powers the agent and judges it.
- Setup: an Anthropic API key and a free Arize Phoenix Cloud account (Phoenix is also open source and runs locally).
- Watch out: Arize has two products — Phoenix (used here) and AX (enterprise). The homepage Sign Up button leads to AX.
- OpenAI and Gemini work too, since the stack is open source and uses open standards, but the workshop demonstrates only Claude.
- Workshop arc: fundamentals → tracing → run the agent → read traces → write evals → meta-evaluate → datasets and experiments.

### Tests need evidence: traces and the "vibes problem" · 05:50
- Reframe the ML jargon: evals are tests, and traces are runtime logs for AI — every LLM call, tool call and agent turn.
- Traces are built from spans: one span per step, with nested spans inside an agent turn. Each records inputs, outputs and metadata like timing and token counts.
- Why evals: informal testing covers only the questions you expect. Real users bring unfamiliar vocabulary, edge cases and adversarial input.
- Fixed unit tests do not work: the same prompt yields different (often all-valid) outputs, so string matching cannot capture the space of correct answers.
- Human review does not scale and does not run in CI, so it cannot catch regressions.
- Concrete risk: editing a prompt to fix tone can make the agent invent product features; a faithfulness eval catches that.

### Use code, models, and humans for different jobs · 10:27
- Deterministic code checks are the cheapest start: validate JSON, required fields, forbidden phrases, length and required mentions.
- LLM judges grade meaning against a rubric — useful for factual accuracy, faithfulness and tone, at the cost of another model call and another prompt that can be wrong.
- Human review supplies domain judgment and finds new failure modes, but has limited throughput and suffers fatigue.
- The three methods complement each other: code for mechanical rules, models for meaning, humans to define and audit success.
- Humans label a golden dataset and arbitrate disagreements; Voss warns that repetitive annotation causes fatigue.
- A capability eval targets something the agent cannot yet do reliably. Once the behavior works, keep it as a regression eval and move development to the next capability.

### Trace a real failure: right research, wrong destination · 42:52
- The Apple trace is the clearest failure: the agent researched correctly but read "write a report" as "create a Markdown file."
- It called `Write` in a notebook where the operation failed. The prompt named the deliverable but not the destination — the report should have been returned in the response.
- Without the trace, that would look like a research or writing failure.
- A second, semantic defect: the Apple and Nvidia reports summarize finances without saying whether to buy. Nvidia avoids the file problem but still lacks the requested decision.
- The fixes differ — one is about output destination, the other about content that makes a report useful.
- A Rivian claim raised a source-verification question, but suspicion is not proof of hallucination; the real question was whether the claim was supported by retrieved sources.
- Categorize root causes (hallucination, reasoning gap, unverifiable data, missing recommendation, delivery failure) and prioritize by **severity × frequency**, not frequency alone.

### Layer the checks: a deterministic ticker eval first · 48:45
- The Swiss cheese model: layering evals improves coverage because each layer has blind spots.
- First eval: does the report mention the requested stock ticker? Run it on root spans so it judges whole outputs, not individual tool calls.
- The check is a small Python function wrapped with `create_evaluator`, kind `code`, returning a score, a label and an explanation; a regex extracts ticker candidates from the request.
- Run it over parent spans with `evaluate_dataframe`; suppress tracing so judge calls do not mix with application traces.
- Result: 11 of 13 runs pass. The two failures: Tesla (the attempted disk write) and Amazon (the report narrows to AWS and omits the ticker).
- Code evals go beyond strings: parse JSON, enforce limits, query a pricing database, or check a claimed price — and fix the reference conditions when external data changes.
- Accept equivalent answers (2 hours = 120 minutes) and don't require an incidental tool order.

### Correctness and faithfulness answer different questions · 55:24
- An LLM eval has three parts: the judge model, the prompt or rubric, and the graded examples.
- Phoenix's built-in correctness evaluator checks factual accuracy, completeness and logical consistency. Other built-ins cover tool selection, tool arguments, document relevance and refusal detection.
- Setting: Claude Sonnet judges the Haiku agent's outputs — use a more capable judge when one is available.
- Every correctness result was zero. Reading the explanations, the judge objected to the 2026 financial figures, treating them as unavailable future facts.
- The two-turn architecture helps: extract the first turn's research from the child spans, add it as a `context` column, and let the faithfulness evaluator check whether the report follows that research.
- These are different signals, not a before/after: correctness 0/13, faithfulness 13/13.
- Faithfulness proves source adherence, not that the research itself is true.

### Define the rubric, then calibrate the judge · 1:04:35
- A built-in evaluator cannot know this product's definition of an "actionable" investment report, so Voss writes a custom rubric.
- Five parts: role and purpose; observable criteria; data boundaries; labeled examples; and a constrained output label.
- Prefer `actionable` / `not_actionable` over an unanchored 1–10 score. A third label is fine only if its meaning is explicit.
- Ask for an explanation before the final classification, while keeping the machine-readable label constrained.
- Initial results have both passes and failures; explanations point to missing buy/sell/hold recommendations even when the financial data is strong.
- You can also filter by latency, token count and cost — an agent that answers correctly after excessive searching may still be unfit for production.
- **Treat evaluator prompts like code:** version them, compare behavior on known examples, and keep a path back. Keep dimensions separate — one judge grading accuracy, tone, completeness and formatting at once is hard to diagnose or calibrate.
- **Meta-evaluation:** add human annotations and compare them with the model's labels. Humans need the same criteria and examples; arbitrary approval is not ground truth. Use a 75/25 development/held-out split to avoid overfitting the rubric.
- For disagreement analysis, define the positive class first (failure detected). Precision asks how often a flag is justified; recall asks how many real failures are caught.
- Watch for judge biases: position, length, confident wording, or similarity to the judge's own output. Trying a different model is a mitigation, not a substitute for calibration.

### From failure subset to a shipping decision · 1:32:03
- Iterate in Phoenix: select failing traces, "Add to Dataset", and rerun only that focused set — faster than the whole corpus after every edit.
- The prompt changes follow the explanations: the research prompt asks for financial ratios, current price and recent news; the writing prompt demands a buy/sell/hold recommendation.
- On the six selected failures, all pass after the changes. That is a hand-picked subset, not a full-suite improvement rate; real gains are often incremental, and some edits get rolled back.
- Repeat trials to account for nondeterminism, but live web search can return different evidence unless retrieval is frozen or replayed.
- **Impact hierarchy:** fix data quality, then prompting, then model selection, then hyperparameters. Prompt engineering cannot recover facts the system never receives.
- **Release uncertainty:** 6/200 and 12/400 both observe 3%, but the rounded two-sided 95% intervals differ (~0.6–5.4% vs ~1.3–4.7%), and a Wilson interval for the larger case still crosses 5%. Agree on a method and use representative sampling.
- Planning heuristics: 12–20 examples for directional feedback; 200–400 as a starting target for shipping decisions.
- Repeated-trial metrics: `pass@K` asks whether at least one of K attempts succeeds; `pass^K` asks whether all K succeed.
- Start small: read real outputs, write one format or required-string check, and grow the suite. Evals become infrastructure when informal checking slows improvement or a change silently breaks something else.
- After release, monitor a sample of live traffic for quality drops and adversarial use. Cost-normalized accuracy (accuracy ÷ cost) makes routing economics visible — for example 92% at $0.02 versus 95% at $0.15 per query.
- Stabilize layers one at a time; introduce several uncertain judges at once and you cannot tell which measurement changed. Rerun the full dataset before shipping to catch overfitting.
- On autonomous self-improvement (DSPy, autoresearch): Voss agrees reliable self-improving software is still frontier work, and the demonstrated loop still needs human trust.

## Resources

- [Ship Real Agents: companion slides](https://slides.com/seldo/ship-real-agents)
- [Instrument the Claude Agent SDK with OpenInference](https://arize-ai.github.io/openinference/python/instrumentation/openinference-instrumentation-claude-agent-sdk/)
- [Demystifying evals for AI agents (Anthropic)](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents)
- [NIST: confidence intervals for defect rates](https://itl.nist.gov/div898/handbook/prc/section2/prc241.htm)
- [DSPy](https://dspy.ai/)
- [Karpathy's autoresearch](https://github.com/karpathy/autoresearch)
