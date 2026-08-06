# Metis — ARM AI Security Code Review

ARM's open-source AI security code review framework. Uses tree-sitter deterministic reachability analysis for C/C++ to build call graphs, then enlists an LLM to confirm which reachable paths are actually exploitable. For all other languages, it runs a conventional LLM review pipeline with language-specific prompts. Findings flow through a deterministic adjudication layer that gates LLM decisions against evidence coverage, then output as SARIF. Think of it as a SAST tool where the rules engine is an LLM, but the control flow analysis is real static analysis — not one more "feed the whole file to GPT and pray."

#tool #project #security #static-analysis #agents

---

## Architecture

Metis is a Python CLI app with three major subsystems:

**1. Review pipeline** — `ReviewService` (`engine/review_service.py`) dispatches each file to one of two backends:
- **Tree-sitter reachability** (C/C++ only): `TreeSitterReachabilityService` (`engine/reachability/service.py`) builds an in-memory call graph from tree-sitter ASTs, resolves sources (public entrypoints) and sinks (security-sensitive calls), traces paths between them, then batches paths to an LLM for vulnerability confirmation.
- **LLM review graph** (everything else): `ReviewGraph` (`engine/graphs/review.py`), a 3-node LangGraph StateGraph (build_prompt → review → parse) that chunk-splits files by token count, wraps findings in a source map for line-number anchoring, and returns structured review output.

**2. Triage pipeline** — `TriageService` processes SARIF findings through a 2-node LangGraph StateGraph (collect_evidence → triage). Evidence collection uses tree-sitter scope extraction + line-local `sed`/`grep` probes + C/C++ macro/include resolution. The LLM returns a structured decision (valid/invalid/inconclusive + evidence chain), then `adjudication.py` applies deterministic rules: contradiction signals force `invalid`, critical unresolved hops force `inconclusive`, insufficient obligation coverage forces `inconclusive`. The model never gets the final word.

**3. Plugin system** — Three extension points, all discovered via Setuptools entry points:
- **Providers** (`providers/`): `ChatProvider` and `EmbeddingProvider` abstract classes, 10 concrete backends (OpenAI, Anthropic, Gemini, Bedrock, Ollama, llama.cpp, vLLM, etc.)
- **Language plugins** (`plugins/`): 19 concrete plugins each defining supported extensions, tree-sitter splitting params, and prompt templates (security_review, validation_review, security_review_checks)
- **Tools** (`engine/tools/`): grep, sed, cat, find_name primitives exposed to the LLM during triage, plus an opt-in vector index search tool

The vector store (ChromaDB default, pgvector optional) is used by the `index`/`ask`/`update` commands but is not required for review or triage.

## Key Techniques

### Tree-sitter call graph construction for C/C++

The most innovative piece. `CFamilyTreeSitterExtractor` (`engine/reachability/c_family.py`) parses every `.c`/`.cpp`/`.h` file with tree-sitter and extracts:
- **Function definitions**: name, location, body span, call targets, static vs. external linkage
- **Global constructs**: function tables and initializers that reference functions by address
- **Public declarations**: non-static function declarations in header files

These are assembled into a `ReachabilityGraph` (`engine/reachability/graph.py`) — an in-memory directed graph where nodes are `FunctionNode` dataclasses keyed by `file_path::function_name`. The graph resolves call targets (handling overloads by preferring same-file matches), annotates public entrypoints from header declarations, classifies sinks by matching unresolved calls against a security-relevant call classification function, and marks sources (nodes with no internal callers).

This is a surprisingly clean 167-line graph implementation — no external graph library, just dicts and dataclasses with hand-rolled call resolution.

### Path-based vulnerability confirmation

The `VulnerabilityConfirmer` (`engine/reachability/confirmer.py`) takes reachable paths (source → ... → sink) and batches them for the LLM with the actual function body text. It uses two distinct prompt strategies:
- **Codebase-wide**: "For EACH path determine if it contains a real exploitable vulnerability" — full path context
- **Per-file**: "Only report a vulnerability when the primary bug mechanism is actually present in the TARGET FILE code shown" — separates target file code from supporting path code, enforcing file-scoped findings

Both modes request structured output with `primary_file`, `primary_function`, `primary_line`, `root_cause_id`, and `canonical_key` — canonical ownership fields that prevent duplicate reporting of the same root cause across different paths. The schema includes explicit rules for choosing the primary location ("the actual defective code, not merely the source, caller, helper, or path endpoint") by vulnerability category (memory bugs, auth bugs, state/order bugs, cleanup bugs, lifecycle bugs, refcount bugs).

### Deterministic adjudication as guardrail

Unlike most LLM-based security tools that treat model output as final, Metis's `adjudication.py` applies post-hoc deterministic rules:
- Empty evidence + unresolved hops → `inconclusive`
- Contradiction signals → `invalid`
- Critical unresolved hops → `inconclusive`
- Insufficient obligation coverage → downgrade to `inconclusive`
- Invalid-to-valid direct upgrades are blocked by invariant checks

This is the same insight behind [[Guardrails and Feedback Loops]] — linters beat prompts, deterministic enforcement beats instructions. Metis applies it to the output side: the LLM proposes, the adjudicator disposes.

### Source map anchoring

`SourceMap` (`engine/source/source_map.py`) tracks line-number mappings so findings are anchored to actual file locations even after chunk-splitting. Each issue's `code_snippet` is matched against the source map to resolve `start_line`/`end_line`, producing a stable anchor. This matters because review chunks may start at arbitrary offsets, and the LLM often reports approximate line numbers — the source map corrects drift.

### Supplementary lens analysis

Beyond path confirmation, Metis runs independent "supplementary" semantic analysis passes (`engine/reachability/supplementary.py`) on the entire call graph. These are lens-based: different prompt perspectives (allocation patterns, error handling, state machines) look at the same graph from different angles. Findings are cached by `(scope_id, graph_fingerprint)` and shared across concurrent workers via a threading.Condition.

## Design Decisions

### Optimized for precision over recall (but tried hard for both)

The tree-sitter reachability path is explicitly designed to reduce false positives: only functions the call graph says are reachable get LLM attention. The supplementary lens passes then sweep for things the path-based approach might miss (e.g., state machine bugs where the dangerous code is the source, not the sink). This is a two-pass design: first prune with static analysis, then sweep with semantic lenses.

The trade-off: this only works for C/C++. Every other language gets the conventional "feed the file to an LLM" treatment, where precision depends entirely on prompt quality.

### LangGraph over custom orchestration

Metis uses LangGraph StateGraphs rather than building its own workflow engine. The graphs are trivially simple (2-3 nodes each), and LangGraph provides checkpointing, caching (InMemoryCache), and structured state management. The alternative would be reinventing these, but the dependency adds weight (langgraph + langchain + langchain-core are ~2MB of packages).

### LLM as judgment machine, not fact-finder

The architecture embodies the [[Smart Models Dumb Pipes]] principle. Tree-sitter does the fact-finding (what functions exist, what calls what, what's reachable). The LLM does the judgment (is this reachable path exploitable?). Deterministic code owns the evidence; the model owns the interpretation. This is the same pattern as [[Tone LLM]] — LLM fills a structured schema, deterministic code translates and validates.

### Plugin config via YAML, not code

Language-specific behavior (extensions, chunk sizes, prompts) lives in YAML files under `plugins/config/`, `plugins/languages/`, and `plugins/profiles/`. New languages can be added without writing Python — just YAML — though tree-sitter reachability support still requires code.

### SARIF as universal interchange

Metis speaks SARIF natively for both input and output. The `sarif/` module (`sarif/triage.py`, `sarif/writer.py`) handles the full SARIF v2.1.0 schema. This means Metis can triage findings from other SAST tools (Semgrep, CodeQL, Coverity) and its own output can be consumed by any SARIF-compatible dashboard or CI plugin. It's the anti-silo design choice — Metis is a component in a pipeline, not a walled garden.

### Separate review and triage models

The default configuration uses `llama_query_model` for everything, but the reachability path supports a `confirmation_model` override — a potentially more capable/expensive model for vulnerability confirmation, while triage uses the cheaper default. This tiered-model approach echoes [[Thrifty (Tiered Delegation for Claude Code)]] and [[Orchestrating AI Code Review at Scale]].

## Comparison Notes

**vs. [[OpenCodeReview]]** (Alibaba): Both use hybrid deterministic+agent architectures for code review. OpenCodeReview uses per-file concurrent subagents; Metis uses tree-sitter call graphs + path batching. OpenCodeReview has dual-threshold context compression and a comment filter pass; Metis has deterministic adjudication and SARIF-native output. Both represent the same insight — pure LLM review produces too many false positives, so wrap it in deterministic scaffolding.

**vs. [[Orchestrating AI Code Review at Scale]]** (Cloudflare): Cloudflare's system uses 7 specialized agents + coordinator judge; Metis uses supplementary lens passes that serve a similar function (multiple perspectives converging). Both tier models by task difficulty. Cloudflare reports detailed cost metrics ($1.19/avg review); Metis tracks token usage but doesn't summarize cost by default.

**vs. [[Trailmark]]** (Trail of Bits): Both treat source code as a queryable graph for security analysis. Trailmark is a general-purpose graph framework; Metis builds a purpose-specific call graph and discards it after analysis. The difference is scope: Trailmark wants to be infrastructure; Metis wants to be a tool.

**vs. [[brooks-lint]]**: brooks-lint is pure prompt engineering — 12 named decay risks diagnosed via Iron Law chain. Metis is architecture engineering — static analysis + LLM judgment + deterministic gating. Both are trying to make LLMs reliable at code review, but from opposite ends: brooks-lint through better prompts, Metis through better scaffolding.

**vs. Semgrep / CodeQL**: Traditional SAST tools use hand-written rules against ASTs/CFGs. Metis replaces the rules with an LLM but keeps the AST analysis. The LLM isn't doing the whole job — it's the judgment step in a pipeline that starts with deterministic parsing. This is the key difference from "just ask ChatGPT to review my code."

**vs. CI Forge (ciforge)**: Metis represents the deep end of AI code review — tree-sitter call graphs, 19 languages, 10 LLM backends. [[CI Forge (ciforge)]] takes the opposite design: ~30 shallow scanners averaging 50 lines each, Python-only AST, simple regex for other languages. ciforge's value is one-command breadth across secret detection, IaC, dead code, CVEs, and cloud cost; Metis's value is surgical depth in security review.

**vs. [[Ship Safe]]**: Ship Safe takes yet another approach: 29 pattern-matching agents running in parallel across a broader security scope (code + supply chain + AI/LLM + CI/CD), offline-first with optional LLM deep analysis. Where Metis uses the LLM as a judgment step inside a deterministic AST pipeline, Ship Safe uses it as an optional escalation tier for taint analysis on pre-filtered findings. Ship Safe's distinctive contribution is treating AI-coding-agent surface area (MCP configs, agent instruction files, slopsquatting, GhostApproval symlinks) as first-class security domains alongside traditional appsec. Neither tool replaces the other — Metis for deep local reasoning on C/C++, Ship Safe for broad offline gating across polyglot repos with agent-specific threat surfaces.

---
*Sources: [[raw/metis]]*
*Last updated: 2026-07-08*
