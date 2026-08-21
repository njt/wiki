# Local Deep Research

An open-source (MIT) "deep research" assistant you run on your own machine: it turns a complex question into a cited report by searching the web, academic databases, and your private documents — with no telemetry, per-user encrypted storage, and any LLM backend from a local Ollama to a cloud model. Its claim to fame is the first ~95% SimpleQA score on a single consumer RTX 3090 (Qwen3.6-27B, fully local), which makes it the reference point for what "deep research" can now cost-free on local hardware.

---

## Architecture

LDR is a **Flask monolith with a plugin spine** — one web app (`web/`), but every dimension of the research pipeline (strategy, search engine, LLM provider, export format) is an abstract-base-class plugin. ~189K lines of Python.

The request path: Flask route (`web/routes/research_routes.py`) → research worker (`web/services/research_service.py`) → `AdvancedSearchSystem.analyze_topic()` (`search_system.py`) → a **strategy** chosen by `search_system_factory.py::create_strategy()`. The strategy owns the research loop; the search engine owns retrieval; the report generator (`report_generator.py`, 58KB) owns synthesis.

**Five user-facing strategies** (registered in `constants.py::AVAILABLE_STRATEGIES`): `source-based` (extract-and-cite, best for small context windows), `focused-iteration` / `focused-iteration-standard` (quick Q&A vs. long-form), `topic-organization` (cluster sources by theme), and `langgraph-agent` (autonomous tool-calling). The factory also handles internal strategies (news, contextual-followup).

**Search engines** are the deepest plugin surface. `BaseSearchEngine` (`web_search_engines/search_engine_base.py`) imposes a **two-phase retrieval** contract: fetch *previews* (title/link/snippet) → run preview filters → optionally LLM relevance-filter → fetch *full content* only for survivors. ~25 engines implement this in `web_search_engines/engines/`, and new ones are added by subclassing `BaseSearchEngine`, registering in `ENGINE_REGISTRY`, and allowlisting in `security/module_whitelist.py`.

**LLM providers** mirror the pattern: `BaseLLMProvider` (`llm/providers/base.py`) returns a *bare* LangChain `BaseChatModel`; `config/llm_config.py::get_llm()` wraps it with rate limiting, token counting, and `<think>`-tag stripping. The wrapper lives outside the provider so providers stay trivial.

**Storage** is the boldest architectural choice: **one SQLCipher database per user**, each with a key derived from the user's password (never stored — login works by *attempting to decrypt*). `database/encrypted_db.py::DatabaseManager` keeps a per-user SQLAlchemy `QueuePool` (`pool_size=20`, `max_overflow=40`) because a shared pool can't span different encryption keys. `docs/architecture.md` documents the resulting FD budget (each WAL-mode connection costs 2 FDs + 1 SHM) and the cleanup layers (thread-cleanup decorator, dead-thread credential sweep, 30-min pool dispose) added after a per-`(user,thread)` NullPool system leaked file handles to exhaustion.

## Key Techniques

**The LangGraph agent is a tool-calling loop where the tools are search engines.** `advanced_search_system/strategies/langgraph_agent_strategy.py` (101KB) builds the agent with LangChain's `create_agent(model, tools, system_prompt)`. Instead of a hand-rolled ReAct loop, each `web_search` / `search_arxiv` / `search_pubmed` tool *re-instantiates the engine per call from the settings snapshot*, so engine choice is dynamic and the LLM decides which specialized engine to query next. The `research_subtopic` tool spawns **nested subagents** — each gets its own `create_agent()` with a filtered tool list — run in parallel under a `ThreadPoolExecutor`. A lock-guarded `SearchResultsCollector` assigns stable `[n]` citation indices across subagents so concurrent fetches of the same URL dedupe instead of duplicating. If the agent hits the recursion limit, it falls back to `_synthesize_from_collector()` over already-collected sources rather than returning nothing.

**The egress guardrail is the standout engineering.** `security/egress/policy.py` is an in-process **PDP** (policy decision point) implementing ADR-0007's two-axis DLP model: every component is labeled **Sensitivity** (is the *data* sensitive?) × **Exposure** (does the *sink* send data off-machine?), and the invariant is *sensitive data must never reach an exposing sink*. It's enforced at multiple PEPs — the search-engine factory, a runtime backstop inside every `BaseSearchEngine.run()`, and a **PEP-578 audit hook** (`security/egress/audit_hook.py`) that intercepts raw `socket.connect` calls and classifies the target host. The scopes (`adaptive`/`public_only`/`private_only`/`strict`/`unprotected`) are SELinux-style; `adaptive` resolves by classifying the primary engine. Notable details: DNS lookups run with a bounded 2s timeout by abandoning a `ThreadPoolExecutor` future (because `getaddrinfo` has no timeout kwarg), cloud-metadata IPs are blocked even in NAT64-wrapped and octal/hex/integer-encoded forms, and denials **raise** rather than return an empty result so the LangGraph agent can't infer policy from response latency (a timing-leak mitigation). A per-run denied-fetch quota (50) stops prompt-injected documents from looping the agent through denial URLs.

**The "knowledge compounds" loop.** Research → download sources → extract text → chunk + embed (sentence-transformers + FAISS) → the collection becomes a search engine. `docs/architecture.md` calls it the knowledge loop; it's RAG where the retrieval index is *also* a first-class search tool the agent can call.

**Journal reputation scoring.** `journal_quality/` scores academic sources against OpenAlex + DOAJ + Stop Predatory Journals (212K+ sources) to flag predatory venues before they're cited — a filter registered as a preview filter on academic engines.

## Design Decisions

- **Privacy as the product.** No telemetry, no analytics SDKs; the only network calls are ones the user initiates. This is a real trade-off — they can't see what breaks, so they lean on community bug reports and a benchmark leaderboard instead of internal metrics.
- **Guardrail honesty.** The egress module's own docstring says it is "an in-process correctness guardrail, NOT a hard security boundary," and ADR-0007 lists *known residuals* (e.g. the run-start audit keys on the primary engine, not the full set the agent might expand into). That candor about what the guardrail does and doesn't stop is rarer than the guardrail itself.
- **Security theater budget.** 57 CI workflows, 22+ scanners, SHA-pinned actions, Cosign-signed images with SLSA attestations and SBOMs. For a self-hosted research tool this is an unusually heavy security surface — arguably over-invested relative to the threat, but it buys the "you can trust the container" story the local-privacy pitch depends on.
- **Correctness over convenience.** The AVX CPU floor, the Linux-only `--network host` Docker default, and removing the `llm.model` default (which used to silently download a multi-GB Gemma) all choose "fail loudly" over "just works" — the same philosophy as the egress fail-closed defaults.

## Comparison Notes

- **vs. [[DeerFlow]]** — the closest sibling: both are LangGraph deep-research agents built on `create_agent()`. DeerFlow is a multi-tenant *platform* (27-middleware chain, sandboxed execution, IM channels); LDR is a single-user *product* whose defining middleware is the egress policy and whose "sandbox" is the encrypted per-user DB. LDR optimizes for the private, local-first researcher; DeerFlow optimizes for composable enterprise deployment.
- **vs. [[Three Kinds of Agentic Search]]** — LDR is a concrete instance of Turnbull's taxonomy: the `langgraph-agent` strategy is harness-centric (the agent steers, engines lead it by the nose), while its library/RAG search is retrieval-centric. The egress pre-filter that strips forbidden engines from the tool list *before* the LLM sees them is harness engineering of the same kind Turnbull's judge-loop advocates.
- **vs. [[Building Reliable Agentic AI Systems]]** — Bayer's PRINCE separates context engineering from harness engineering; LDR's egress PDP + runtime backstop + audit hook is a harness-engineering layer in a research tool, though LDR lacks PRINCE's explicit *data-sufficiency* reflection loop (it trusts the agent's "when you have enough information" judgment).
- **vs. [[Sawtooth Memory]]** — both default to local Ollama, but their "memory" is different: Sawtooth is hierarchical *episodic* memory for a conversation; LDR has no agent-memory hierarchy at all — its long-term memory is the encrypted library/RAG store and research history, i.e. *semantic* memory over documents rather than over turns.

## Tags

#tool #project #agents #search #memory #security #rag #local-llm

---
*Sources: [[raw/local-deep-research]], [[summary/local-deep-research]]*
*Last updated: 2026-08-21*
