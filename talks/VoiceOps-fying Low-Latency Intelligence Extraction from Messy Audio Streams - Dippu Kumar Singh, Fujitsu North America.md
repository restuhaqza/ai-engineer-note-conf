# VoiceOps-fying Low-Latency Intelligence Extraction from Messy Audio Streams

| Field | Details |
| --- | --- |
| **Speaker** | Dippu Kumar Singh |
| **Affiliation** | Fujitsu North America |
| **Type** | Talk |
| **Date** | 2026-04-08 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=IEF842ZEU5A) |
| **Tags** | `evals` `speech-and-audio` `rag-context-and-search` `architecture` |

## Summary

Dippu Kumar Singh, who leads emerging data technologies and AI architecture at
Fujitsu North America, describes a low-latency pipeline that turns messy
contact-center audio into structured business records. Real customer conversations
don't start as clean text: they are overlapping, sometimes emotionally charged, and
often spread across multiple audio channels.

The operational pain is **after-call work (ACW)** — the typing, summarizing, and
disposition coding that happens once the call ends. Singh reports a baseline of
about 6.3 minutes of post-processing on top of roughly 6.5-minute calls, nearly a
one-to-one ratio. Because summaries depend on each operator's memory and writing,
records are also inconsistent.

His architecture moves from raw audio to a database-ready asset in four stages:
voice capture, speech-to-text, a generative AI core, and a customer-data sync layer
that writes into CRM (customer relationship management) systems. Operators stay in
the loop, confirming AI-generated drafts before they become records.

The reported result is ACW cut from 6.3 to 3.1 minutes, roughly a 50% reduction,
plus more consistent data. Singh is candid about what remains hard: transcription
quality, token cost, and the latency added by privacy masking.

## Key Takeaways

- **After-call work is the target.** Documentation nearly matches call length, and
  it is administrative work, not customer interaction.
- **Garbage in, garbage out.** Flawed audio intake leads the model to hallucinate
  later, so capture quality constrains everything downstream.
- **Preserve speaker channels.** Keeping agent and customer on separate channels
  preserves attribution even when they overlap; mixing to mono loses who said what.
- **Mask PII early.** Sensitive data such as card numbers and passwords should not
  reach the model, so the boundary is set during capture.
- **Transcription must be good enough first.** Singh reports STT accuracy above 90%
  was needed for effective summarization in their setup.
- **Define the output structure.** Separate lists for customer inquiries and
  operator actions beat a messy narrative summary.
- **Keep a human confirm step.** The operator validates and edits a prefilled draft
  before it becomes a record.
- **The gain is operational.** ACW fell about 50%, and record consistency improved.

## Detailed Notes

### The work does not end when the call ends · 00:00
- Contact centers are the front line of customer experience but are structurally
  under pressure: Singh cites an industry finding that over 50% name hiring,
  training, and productivity as critical barriers.
- High stress is the leading reason operators leave; they must manage customer
  emotions, navigate multiple CRM systems, and document everything accurately.
- This creates a negative spiral: understaffing raises stress, which drives
  turnover, which deepens understaffing. The fix is to engineer the stress out of
  the workflow, not just hire more people.
- The ACW target: Singh reports a baseline average call duration of about 6.5
  minutes and post-processing of about 6.3 minutes. (A slide shows 6.6 minutes for
  call duration.)
- The initial goal was to cut post-processing by roughly 50% or more through
  automated summarization and extraction, which also makes the voice of the
  customer available for business analysis.

### Capture usable audio without losing who said what · 06:43
- The pipeline has four stages, each feeding the next: **voice capture**,
  **speech-to-text**, a **generative AI core**, and a **customer-data sync** layer.
- The aim is minimal human intervention across the whole pipeline, not a standalone
  summary someone must copy into another system.
- Capture applies noise filtering (to strip back-office chatter) and level
  normalization.
- **Keep existing speaker channels separate.** In the example, the telephony feed
  puts the agent on the left channel and the customer on the right. Preserving this
  keeps attribution even when speakers overlap; mixing to mono makes it harder to
  tell who said what. (This preserves a feed that is already separated — it does
  not split voices out of an already-mixed recording.)
- Capture also sets the privacy boundary: audio can include credit-card numbers,
  passwords, and other personally identifiable information (PII). Buffer management
  and early PII masking are required so sensitive data does not reach the model.

### Make the transcript suitable for extraction · 09:53
- Singh reports STT accuracy needed to exceed 90% for effective summarization in
  their setup; he does not define the metric or evaluation conditions, so treat it
  as a reported requirement.
- Transcription combines acoustic modeling (mapping phonemes, handling regional
  dialects) with domain-specific language knowledge.
- Domain knowledge matters: in insurance, the engine must tell **term life** from
  **turn right**. A fluent summary cannot repair a wrong product by rewriting it,
  so domain-specific dictionaries help the recognizer pick the right terms.
- Post-processing makes recognized content easier to extract: **inverse text
  normalization** converts spoken quantities to numbers, and **automatic
  punctuation** adds structure.
- Example: "five thousand dollars" becomes `$5,000`, making the amount explicit for
  later entity extraction. Recognition and formatting serve different purposes —
  first recover the words, then represent meaning consistently.

### Specify the summary's structure and classification · 11:21
- A generic "summarize this call" request produced messy narrative paragraphs in
  Singh's setup.
- Instead, prompt templates and few-shot examples establish structure: one bullet
  list for **customer inquiries** and another for **operator actions**, preserving
  the difference between what was requested and what the operator did.
- For intent classification, the model receives a predefined set of call reasons
  (for example cancellation, new application, claim status) and must pick a
  category and explain its choice.
- A compact JSON Schema expresses the contract: `customer_inquiries`,
  `operator_actions`, `call_reason` (from the allowed enum), and
  `classification_reason`, with `additionalProperties` false.
- The category vocabulary belongs to the business; the model classifies within it
  rather than inventing a new disposition per call.
- A trust layer adds token optimization for latency and automated hallucination
  checks to keep summaries grounded in the transcript. Structural validation and
  grounding are separate: an output can satisfy a schema yet claim an action that
  never happened.

### Put operator confirmation before the record · 12:58
- The **customer-data sync** layer makes output usable in existing systems: an API
  gateway acts as a **schema mapper**, translating JSON fields like customer intent
  and resolution status into CRM fields through REST APIs.
- This keeps the model's extraction task separate from the destination system's
  field names and record structure.
- A **verification step** stays in the workflow: the generated summary appears
  prefilled on screen, the operator validates fields, makes minor edits, and clicks
  **Confirm**.
- The important distinction is between a generated proposal and an
  operator-confirmed record — automation removes typing without treating every
  generated value as approved.
- The structured data also feeds business intelligence: a voice-of-customer
  aggregator for management dashboards, and recurring questions flagged as
  candidates for new FAQ entries (candidates, not auto-published answers).

### Follow the intermediate representations · 14:17
- Tracing the flow through its intermediate data makes the pipeline clear:
- **Prepare the transcript** — carry time indexes and confidence scores, with
  denoising applied upstream.
- **Reconstruct the dialogue** — use the preserved channel mapping to arrange
  customer and agent turns logically, retaining who said what.
- **Deduce context** — identify entities, analyze sentiment, and recognize intent.
- **Emit structured output** — match a predefined customer-data or CRM template,
  with summary content in separate lists.
- The result is a predictable representation that downstream software can map into
  records, not just readable prose.
- Singh notes that extracted entities include account numbers and customer names,
  alongside product names. How those sensitive values remain available for
  authorized use while satisfying earlier masking is left unspecified — an
  implementation needs an explicit policy for what is masked, retained, or
  referenced.

### The reported gain is less after-call work · 15:40
- Singh reports average ACW falling from 6.3 minutes manually to 3.1 minutes with
  the AI workflow — about a 50% reduction. This is a reported deployment outcome,
  distinct from the earlier target.
- The talk does not give sample size, measurement period, call mix, or a separate
  accounting of review effort.
- He illustrates potential scale with 500 seats handling thousands of calls a day,
  describing recovered capacity as equivalent to dozens of full-time staff. This is
  a capacity extrapolation, not an established deployment size or a hiring result.
- The other reported change is consistency: data entry becomes more standardized,
  and call-reason tagging depends less on an operator's memory or mood.
- Singh also attributes lower cognitive load, more stable operations, and reduced
  turnover to removing repetitive documentation, completing the link to the opening
  stress cycle — though no turnover measurement accompanies the claim.

### Remaining constraints and roadmap · 17:35
- **Transcription quality:** heavy accents and poor audio still cause recognition
  failures, and the summarizer depends on the transcript, so STT work continues.
- **Token cost:** complex reasoning over long calls (he uses a 20-minute transcript
  as an example) can consume substantial API tokens during initial scaling. Token
  optimization is a cost concern as well as a latency one.
- **Security overhead:** robust masking before data reaches a cloud endpoint is a
  strict requirement; the extra layers add latency and architectural complexity,
  and simplifying them must preserve the privacy boundary.
- The architecture targets low latency, but no measured end-to-end latency or
  component-level timing budget is provided.
- Roadmap phase 1 — **private operator coaching:** analyze post-call audio and give
  immediate private feedback on soft skills, empathy, and information accuracy.
- Roadmap phase 2 — **predictive staffing:** feed categorized intent data into
  time-series analytics to forecast topic-specific call spikes and improve shift
  scheduling.
- Roadmap phase 3 — **harassment response:** low-latency sentiment and acoustic
  analysis, under development, to detect abusive behavior and alert a supervisor or
  transfer the call to an AI voice agent to protect the operator.
- These are planned extensions, not reported capabilities of the deployed workflow.
  The destination is a contact center that gathers intelligence while protecting
  the people doing the work.

## Resources

- Evaluate speech-recognition accuracy — practical guide to testing recognition
  quality.
- Transcribe separate audio channels — channel-labeled transcription, timestamps,
  and speaker attribution.
- PII redaction in streaming transcription — identifying and masking sensitive data
  in real time.
