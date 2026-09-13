# TX Text Control AI — Private AI for Document Workflows

Text Control's preview release (0.1.0-beta.1, September 2026) wires local generative AI into .NET document workflows through a strict division of labor: the LLM handles language, an integration layer manages the workflow, and the TX Text Control engine performs the actual document operations. The model never touches document binaries — it proposes structured MCP tool calls that a separately hosted document server executes. Underneath the product marketing is a genuinely useful architectural reference for private, document-heavy AI systems: three independently deployable hosts (web UI, AI/inference service, MCP document server), a private RAG layer with versioned sources, and an unusually honest account of what "local" does and does not buy you.

---

## The Architecture

Six NuGet packages, each with one job:

| Package | Role |
|---|---|
| `TXTextControl.AI` | Developer-facing chat/streaming/structured-response APIs over `Microsoft.Extensions.AI.IChatClient` |
| `.AI.LlamaServer` | Native llama.cpp server process management, hardware reporting, embedding generator |
| `.AI.Mcp` | MCP client: document-tool discovery, invocation, artifact download |
| `.AI.AspNetCore` | Reusable HTTP endpoints and browser libraries; "your application retains ownership of its pages, navigation, identity, and business rules" |
| `.AI.Knowledge` | Private collections, versioned sources, indexing jobs, SQLite full-text search, optional semantic/hybrid retrieval |
| `.AI.McpServer` | Hosts the document engine separately, exposing document operations through MCP |

The deployment topology is the interesting part: web host owns UI and editor integration, AI host owns model files, inference runtime, Knowledge database, and embeddings, MCP host owns document processing, working sessions, rendering, and conversion. Inference can move to a central GPU server while documents and the knowledge base stay where they are — the same building blocks from a workstation to an organization-controlled server.

## Key Quotes

> "The model handles language. The integration layer manages workflow. The document engine performs document operations. Separating these responsibilities makes it possible to choose a model, replace a user interface, or move inference to a different host without turning the entire application into a tightly coupled system."

The thesis, stated in one paragraph. The LLM is deliberately demoted to a proposal engine; a request like "Change Heading 1 to red and underline it" becomes a structured document-tool call applied by the engine, not prose instructions a human must transcribe. This is the same pattern as deterministic renderers elsewhere in the wiki — [[Morningprint]]'s ESC/POS renderer, [[Building Shippy — Agent Architecture for High-Stakes Domains]]'s deterministic CLI wrappers — applied to rich documents.

> "It is not another language model. It is the component that knows how to work with document formats, native paragraph styles, tables, fields, and exports."

The MCP server's self-description is the sharpest line in the piece. MCP here is not a chatbot accessory; it is the seam that lets the nondeterministic component (the model) delegate to the deterministic one (the engine) without either knowing the other's internals.

> "Just because something is local does not mean it is secure, compliant, or completely offline. [...] The benefit is architectural control, not the claim that installing a package establishes regulatory compliance."

Remarkable candor for a launch post. The sample loads Google Fonts by default and needs explicitly provisioned runtimes; the admin interface has no built-in password (you generate one into user secrets, which the post itself calls "a development convenience, not an encrypted production secret store"). The FAQ's "No" answers about not sending documents to public providers are the marketing; this sentence is the engineering.

> "Text inside a reference document is still data and does not grant permission to execute tools or override the user's request."

A prompt-injection rule stated as a design principle — retrieved passages ground answers but never carry authority to act. The honest caveat: the mechanism enforcing this is a policy default ("References are read-only by default"), not a described enforcement layer. Compare the token-level rigor of [[CSharp MCP Cross App Access]], where the delegation chain is cryptographically verifiable; here the trust boundary between "reference text" and "edit instruction" is asserted rather than shown.

> "Uploading documents does not train the model. The model's weights do not change."

Necessary in 2026, and still worth saying. Knowledge is retrieval, not fine-tuning: updating a source updates retrievable knowledge without touching weights.

> "This distinction prevents an answer about a document from being confused with the document itself."

Small UX decision with large architectural consequence: generated summaries and the working document export through different paths. The system keeps the map (chat answer) separate from the territory (the DOCX/PDF).

> "A successful load or a filename is not a benchmark, nor is it proof that every document-tool operation behaves correctly."

The hardware section refuses to give certified minimums and instead tells you to measure time-to-first-token, prompt processing, generation throughput, end-to-end document latency, and tool-call correctness separately — "A short chat benchmark is not a substitute for testing a long document review or an indexing job." Evaluation advice embedded in a vendor doc, pointing at the same truth [[Goodhart's Law and AI Benchmarks]] argues from the benchmarking side.

## Key Themes

#tool #pattern #mcp #local-inference #rag #documents #dotnet #architecture

## The Private RAG Layer

The Knowledge package is deliberately unfancy: bounded passages, retained source/version info, SQLite full-text search as the always-available baseline, Qwen3-Embedding-0.6B (1,024 dimensions, exact query/document prefixes per Qwen's docs) as optional semantic search, hybrid retrieval combining both. Extraction splits by format: text and Markdown are extracted locally on the AI host, while DOCX/RTF/PDF/TXT/HTML extraction is delegated to the MCP document server — "a dedicated extraction request, not an LLM-driven tool loop for every paragraph."

Two operational details stand out. Source replacement is versioned — a failed replacement "does not silently discard the previously active version." And context budgeting is explicit: attachments are transferred through the document workflow, their Base64 representation never enters the conversation, and long inputs get hierarchical chunked analysis with configurable limits. Most chat-app AI integrations bloat context with file payloads; this one routes documents around the model.

The honest limit: "retrieval is not proof of completeness." Comparing a working document against an approved corpus "requires a deliberate, exhaustive workflow rather than assuming that the top few search results represent all documents," and the application should flag gaps when evidence is missing or conflicting.

## Critical Analysis

**The real contribution is boundary discipline, not novelty.** Nothing here invents a technique — llama.cpp serving, GGUF models, SQLite FTS, MCP are all standard. What the preview demonstrates is how far you can get by refusing to let the model do anything but language: structured tool calls for edits, dedicated extraction requests instead of LLM loops, documents routed around the context window, references read-only by default. Each rule is an admission about what LLMs are bad at (format fidelity, determinism, completeness) turned into an interface decision.

**The vendor undercuts its own marketing, which is the credible part.** "Architectural control, not regulatory compliance"; "a starting point, not a complete enterprise, multi-tenant deployment"; "neither an automatic multi-GPU cluster nor an unlimited-concurrency inference service." A launch post that enumerates its own failure modes is rarer than it should be, and it makes the product claims easier to evaluate rather than harder. The cost model gets the same treatment: local inference trades per-request fees for "hardware, administration, energy, and maintenance" — no pretense that self-hosting is free.

**The prompt-injection story is asserted, not engineered.** The rule that reference text "does not grant permission to execute tools" is exactly right, but nothing in the post describes what enforces it beyond a read-only default. The MCP connection gets "endpoint-scoped credentials and trust settings," which is more than most samples bother with, yet the gap between a policy sentence and an enforced boundary is where these systems actually fail.

**Operational friction is the tax on this architecture.** Changing embedding models requires reindexing and a process restart; switching integration modes requires a website restart and does not migrate models, collections, or conversations; the runtime tab requires explicit backend installation, allowlisted URLs, and per-endpoint credentials. None of this is wrong — it is the honest price of the control being sold — but "start on a workstation and move to a GPU server" glosses over how much reconfiguration the move entails.

**Where it sits in the wiki.** [[Document Generation in .NET]] asked "when the document needs to change, who can change it?" and argued for template-based generation with a real editor. This source strengthens that argument from the AI side: a model-plus-tool-call layer makes the previously developer-only edits ("change the style, then regenerate") operable in natural language while keeping the deterministic engine as the thing that actually touches the file. [[docmason]] is the open-source counterpart — locally-managed, provenance-traced document knowledge, with the same "retrieval is not proof" humility and the same local-first tradeoff of convenience for control. [[LocalAI]] is the platform-level version of the same bet; TX's contribution is showing what a vertical slice (documents) looks like when each responsibility gets its own deployable host. And for production RAG in a regulated industry, [[Building Reliable Agentic AI Systems]] makes the deeper argument this preview only gestures at: retrieval needs explicit data-sufficiency reflection, not just a source list attached to an answer.

---

*Sources: [[raw/introducing-tx-text-control-ai-private-ai-for-real-document-workflows-in-dot-net-c-sharp]], [[summary/introducing-tx-text-control-ai-private-ai-for-real-document-workflows-in-dot-net-c-sharp]]*
*Last updated: 2026-09-13*
