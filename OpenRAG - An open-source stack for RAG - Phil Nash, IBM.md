# OpenRAG: An open-source stack for RAG

| Field | Details |
| --- | --- |
| **Speaker** | Phil Nash |
| **Affiliation** | IBM |
| **Type** | Talk |
| **Date** | 2026-04-08 |
| **Video** | [Watch on YouTube](https://www.youtube.com/watch?v=4TxOBhDRRCM) |
| **Tags** | `rag-context-and-search` `enterprise` `architecture` |

## Summary

Phil Nash, a developer relations engineer at IBM, pushes back on the idea that
huge context windows make retrieval-augmented generation (RAG — fetching relevant
documents to feed a model) obsolete. Even if all a business's data fit in one
prompt, sending it with every question is expensive, and most data won't fit
anyway. In practice, RAG is not dead as much as it is assumed to be solved when it
is actually hard.

His answer is **OpenRAG**, an open-source baseline that combines three existing
projects: Docling for document processing, OpenSearch for indexing and search, and
Langflow for visual orchestration and agents. The point is to make good default
decisions while leaving application-specific choices open and inspectable.

Nash walks through the whole pipeline on his laptop. Documents are parsed into a
structured model, chunked with awareness of that structure, embedded, and indexed
for hybrid (vector plus keyword) search. On the generation side, an agent decides
its own searches instead of trusting a single nearest-neighbor lookup.

He finishes by opening the agent's actual flow in Langflow, adding a guardrail
component, and showing how OpenRAG can serve other agents through an API and an
MCP server. His verdict: whether RAG is "solved" depends on your data and users;
OpenRAG offers an opinionated starting point, not proof that every case is handled.

## Key Takeaways

- **Large context windows do not remove retrieval.** Capacity and recurring input
  cost both argue against dumping all information into every prompt.
- **RAG is hard for different reasons per project.** PDFs, chunking, improving
  embeddings, and reranking each add choices.
- **A reusable pipeline is not a universal configuration.** Different documents,
  users, and questions need different decisions.
- **Keep structure after extraction.** Docling's chunkers work on the structured
  `DoclingDocument`, not a flat stream of text.
- **Hybrid search beats one method.** OpenRAG combines vector and keyword search
  with filters in OpenSearch.
- **Let the agent drive retrieval.** Agentic retrieval lets the model run several
  searches and interpret results, instead of one top-K lookup.
- **Inspect and edit the flow.** The ingestion and generation pipelines are things
  a developer can open and change in Langflow.
- **OpenRAG can be used offline**, including Docling in air-gapped settings, with
  local models served through Ollama.

## Detailed Notes

### A large context window does not solve retrieval · 00:14
- The "RAG is dead" claim assumes a business has less than a million tokens of
  data and is happy paying for a million input tokens on every question.
- A better reading is "RAG is solved" — we think we know the recipe:
  extract text, chunk, embed, store in a vector database, embed the question,
  retrieve top K, and pass those to the model.
- The difficulty hides inside each step: PDFs are painful to parse, chunking
  strategies are hard to test, and embeddings keep improving.
- Extra techniques — chunk summaries, chunk expansion, cross-encoder reranking,
  query rewriting — add further choices.
- Nash's core point: a reusable pipeline is not a universal configuration, because
  documents, users, questions, and expectations differ.

### Start with a baseline, keep document structure · 01:57
- **OpenRAG** brings together **Docling** (document processing), **OpenSearch**
  (indexing and search), and **Langflow** (visual orchestration and agents).
- Docling came out of IBM Research Zurich and handles HTML, Markdown, Word, slides,
  spreadsheets, audio, video, and PDFs.
- Different inputs take different paths: a simple pipeline extracts text from
  Markdown/HTML/Word; audio and video go through automatic speech recognition
  (ASR); PDFs use focused models for layout, text, tables, and images.
- For scanned PDFs, a selectable **OCR** (optical character recognition) backend
  reads text that isn't embedded.
- An alternative PDF path uses **Granite-Docling-258M**, a vision-language model
  that does the extraction in one task-specific model; `258M` is model size, not an
  accuracy claim.
- Extraction retains structure: the **`DoclingDocument`** is the structured object
  (a Pydantic datatype), and `DocTags` is one serialization of it, not the object
  itself.
- Docling's native chunkers work directly on `DoclingDocument`, using the parsing
  hierarchy to make meaningful chunks rather than treating text as a flat stream.

### Embeddings and hybrid search in OpenSearch · 05:14
- OpenRAG is not prescriptive about embeddings; providers include OpenAI,
  watsonx.ai, and Ollama for locally hosted embeddings.
- The whole stack can run offline with local models, including Docling in
  air-gapped environments; hosted providers and cloud connectors still need network
  access.
- Chunks are indexed in **OpenSearch**, the open-source fork of Elasticsearch,
  which supports vector search, keyword search, filtering, and aggregation.
- OpenRAG combines vector and keyword retrieval into **hybrid search** and exposes
  filters to narrow the documents searched.
- Vector search across multiple embedding models is supported — useful during an
  embedding-model migration, but slower in practice.
- A "secret fourth" component: the **JVector k-NN plugin**, used by default.
  JVector supports live indexing, and its DiskANN-based design means the whole
  index need not fit in memory.
- Langflow's drag-and-drop editor ties Docling, embedding providers, OpenSearch,
  and data-enrichment stages into an inspectable ingestion flow.

### Agentic retrieval: the agent picks its searches · 07:15
- On the generation side, OpenSearch already holds the indexed documents, and
  Langflow orchestrates an agent with access to OpenAI, Anthropic, Ollama, and
  watsonx.ai.
- **Traditional RAG:** embed the query, retrieve top-K chunks, and hand them to the
  answering model.
- **Agentic retrieval:** give the agent instructions and search tools, and let it
  choose its searches and interpret the results.
- In the traditional path, success depends on the first nearest-neighbor search
  finding the answer; the agentic path can search repeatedly as needed.
- In the laptop demo, the first question "What is OpenRAG?" is answered from the
  agent's default prompt — though the agent still calls a tool for the current
  date. Tool use and document search are distinct.
- Suggested follow-up questions, called **nudges**, are generated through Langflow.
  A follow-up about Langflow's role triggers a real documentation search.

### Inspecting knowledge and connecting sources · 09:28
- The knowledge section accepts a single file or a whole folder, and exposes a sync
  control.
- Users can inspect stored objects, documents, and their chunks, so the result of
  ingestion is visible.
- **Knowledge filters** use OpenSearch filtering; users build filters from
  attributes in their data and select them in chat to restrict retrieval.
- Uploading information and choosing which information a conversation uses are
  separate controls.
- Cloud connectors extend ingestion to external stores — demonstrated with Google
  OAuth — letting users connect Google Drive, SharePoint, or OneDrive directories.
- OpenRAG can synchronize those sources instead of repeated manual uploads, though
  the walkthrough does not state a refresh interval or freshness guarantee.

### Configure local models and ingestion · 10:55
- Model settings support API providers or local models through Ollama; the demo
  uses an **IBM Granite 3B** language model.
- The same settings expose the agent's instructions, so the system prompt can be
  edited without opening the flow.
- Embeddings use **Qwen3-Embedding-0.6B** through Ollama.
- Ingestion settings expose chunk size and chunk overlap, plus Docling options:
  capture table structure (on), run OCR (off), extract picture descriptions (off).
- Picture descriptions can surface information inside images but add ingestion
  time — which is why OCR and picture descriptions are off in the demo.
- API-key configuration at the bottom lets another application use OpenRAG's search
  or agent without adopting the bundled chat interface.

### Open the agent's implementation in Langflow · 12:11
- The **Edit in Langflow** button opens the real agent implementation.
- In the generation flow, chat input passes through a prompt template that adds
  knowledge-filter information when filters are selected.
- The agent has several tools: **URL ingestion** (an MCP server exposing another
  Langflow flow), a **calculator** (so the model doesn't do arithmetic), and
  **OpenSearch retrieval** across embedding models.
- The canvas shows both the agent's incoming context and the capabilities it can
  invoke while responding.

### Guardrails, and using OpenRAG elsewhere · 13:05
- Nash customizes the generation flow: unlock and save the flow, add a guardrails
  component after chat input, add a parser on the pass path, and extract the
  original text to send to the prompt template.
- The pass path preserves the user's text while inserting a check before prompt
  construction; the fail path is wired to chat output with an error message.
- This demonstrates a flow edit, not a tested level of protection.
- OpenRAG also exposes an **MCP server** and an API, so other agents can use it as
  a capability. This is separate from the URL-ingestion MCP tool inside the flow.
- The version presented is **OpenRAG 0.4.0**; the frontend is Next.js and the rest
  is Python.
- Closing point: OpenRAG supplies an opinionated, agentic baseline while leaving
  room to customize how documents become context; its defaults are a starting
  configuration, not a guarantee for every dataset.

## Resources

- OpenRAG — source code and setup for the integrated Docling, OpenSearch, and
  Langflow platform.
- OpenRAG 0.4.0 release notes.
- OpenRAG documentation — architecture, installation, flows, and knowledge
  management.
- DoclingDocument and Docling chunking documentation.
- Granite-Docling-258M model card.
- OpenSearch JVector plugin.
- Langflow repository.
- Qwen3-Embedding-0.6B model card.
