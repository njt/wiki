---
url: https://github.com/arm/metis
title: Metis — AI-Powered Security Code Review
author: Arm Product Security Team (open-source-office@arm.com)
date_fetched: 2026-07-08
date_published: 2025
---

# Metis — AI-Powered Security Code Review

Metis is an open-source, agentic AI security framework for deep security code review, created by Arm's Product Security Team. It helps engineers detect subtle vulnerabilities in large, complex, or legacy codebases.

## Architecture Analysis

### System Overview

Metis is a Python 3.12+ application structured as a CLI-driven pipeline with a plugin architecture. The topology:

```
CLI (entry.py) → MetisEngine (core.py) → ReviewService + TriageService
                     ↓
    Provider Layer (LLM backends)  +  Plugin Layer (language support)
                     ↓
    LangGraph Graphs (review, triage, ask)  +  Tree-sitter Reachability (C/C++)
```

The key files and their roles:

**Entry Point & CLI** (`src/metis/cli/`):
- `entry.py` (456 lines): argparse-based CLI, interactive REPL and non-interactive modes, builds the engine from runtime config
- `command_registry.py` (174 lines): CommandSpec dataclass with invocation modes (path, question, index, args, meta), tool requirements, validation
- `commands.py` (357 lines): handlers for index, review_code, review_file, review_dir, review_patch, triage, ask, update

**Engine Core** (`src/metis/engine/`):
- `core.py` (298 lines): MetisEngine class — the god object that wires together provider, reachability service, review service, triage service, tools, and usage tracking
- `review_service.py` (369 lines): ReviewService — orchestrates file-by-file review, delegates C/C++ files to the reachability backend and everything else to the LangGraph LLM review graph; uses ThreadPoolExecutor for parallelism
- `triage_service.py` (54 lines): TriageService — composition class inheriting from RuntimeMixin and ExecutionMixin; delegates to worker-pool-based parallel triage

**LangGraph Graphs** (`src/metis/engine/graphs/`):
- `review.py` (298 lines): ReviewGraph — 3-node StateGraph (build_prompt → review → parse), chunk-splits files by token count, builds source maps to anchor findings to line numbers
- `triage/graph.py` (246 lines): TriageGraph — 2-node StateGraph (collect_evidence → triage), deterministic adjudication post-model
- `triage/adjudication.py`: Deterministic rules that gate LLM decisions (contradiction signals → invalid, unresolved hops → inconclusive, etc.)

**Tree-sitter Reachability** (`src/metis/engine/reachability/`):
- `service.py` (419 lines): TreeSitterReachabilityService — the orchestrator for C/C++ analysis; builds call graphs, finds paths from sources to sinks, runs supplementary semantic analysis, confirms vulnerable paths with LLM, deduplicates findings
- `graph.py` (167 lines): ReachabilityGraph — in-memory call graph: nodes indexed by unique_name (file::function), resolved calls, source/sink/entrypoint annotations
- `c_family.py` (299 lines): CFamilyTreeSitterExtractor — parses C/C++ files with tree-sitter, extracts function definitions, call edges, globals, public declarations
- `confirmer.py` (391 lines): VulnerabilityConfirmer — batches reachable paths and sends them to the LLM to verify exploitability; has codebase-wide and per-file modes
- `supplementary.py` / `supplementary_lenses.py`: Runs independent semantic analysis passes (lens-based) on the full graph to find issues the path-based approach might miss

**Provider Layer** (`src/metis/providers/`):
- `base.py`: ChatProvider and EmbeddingProvider abstract classes; ChatProvider returns LangChain BaseChatModel, EmbeddingProvider returns LlamaIndex BaseEmbedding
- Concrete providers: openai, azure_openai, anthropic, gemini, bedrock, bedrock_mantle, ollama, llamacpp, vllm, openai_compatible
- `registry.py`: Plugin-based provider discovery via Setuptools entry points (`metis.providers` group)

**Plugin Layer** (`src/metis/plugins/`):
- `base.py`: BaseLanguagePlugin ABC (get_name, can_handle, get_splitter, get_prompts, supports_reachability_review) + ConfigBackedLanguagePlugin for YAML-driven plugins
- 19 concrete plugins: c, cpp, python, java, javascript, typescript, go, rust, csharp, kotlin, ruby, php, solidity, terraform, systemverilog, verilog, tb (TableGen), aarch64_assembly
- Each plugin defines supported extensions, tree-sitter code splitting params, and prompt templates (security_review, validation_review, security_review_checks)

**Vector Store** (`src/metis/vector_store/`):
- `base.py`: BaseVectorStore ABC (init, get_retrievers, index_nodes, get_index_handles)
- `chroma_store.py`: ChromaDB backend (default, local)
- `pgvector_store.py`: PostgreSQL + pgvector backend (multi-project, scalable)

**Memory System** (`src/metis/memory/`):
- Uses LangGraph's BaseStore protocol for repository-level memory
- SQLite-backed with fingerprint-based deduplication for findings

### Key Architecture Decisions

1. **Hybrid deterministic+LLM for C/C++**: Tree-sitter builds the call graph deterministically; LLM only looks at paths the graph says are reachable. This is the core insight — use static analysis to filter what the LLM sees, rather than dumping full files into context.

2. **Two-tier language support**: C/C++ gets deep reachability analysis; all other languages get a generic LLM review with language-specific prompts. The split is explicit in the code — `ReviewService.review_file()` checks `supports_file()` and routes accordingly.

3. **LangGraph as the workflow engine**: Both review and triage use LangGraph StateGraphs (not agent loops). The graphs are deterministic pipelines: collect evidence, then ask the LLM, then parse. The triage graph has exactly 2 nodes; the review graph has 3.

4. **Plugin discovery via Setuptools entry points**: Both providers and language plugins use the `metis.providers` and `metis.plugins` entry point groups, allowing third-party extensions without modifying core code.

5. **SARIF as the interchange format**: Findings flow in and out as SARIF, making Metis composable with the broader SAST ecosystem. External tools' SARIF can be fed to Metis for triage; Metis output can be consumed by other SARIF tools.

6. **Deterministic adjudication over model trust**: The triage system applies deterministic rules after the LLM decision, downgrading `valid`/`invalid` to `inconclusive` when evidence coverage is insufficient. This prevents overconfident status flips.

## File Line Counts

| File | Lines | Role |
|------|-------|------|
| `engine/core.py` | 298 | MetisEngine god object |
| `engine/review_service.py` | 369 | Review orchestration |
| `engine/reachability/service.py` | 419 | Tree-sitter reachability orchestrator |
| `engine/reachability/confirmer.py` | 391 | LLM path vulnerability confirmation |
| `engine/reachability/c_family.py` | 299 | C/C++ tree-sitter extraction |
| `engine/reachability/graph.py` | 167 | In-memory call graph |
| `engine/graphs/review.py` | 298 | LangGraph review pipeline |
| `engine/graphs/triage/graph.py` | 246 | LangGraph triage pipeline |
| `engine/triage_service_exec.py` | 283 | Multi-threaded triage execution |
| `cli/entry.py` | 456 | CLI entry point and REPL loop |
| `cli/commands.py` | 357 | Command implementations |
| `providers/base.py` | 69 | Provider abstraction |
| `plugins/base.py` | 79 | Language plugin abstraction |

## Dependencies

Core dependencies from pyproject.toml:
- langchain 1.3.4, langgraph 1.2.4 — LLM workflow orchestration
- tree-sitter 0.25.2, tree-sitter-language-pack 1.8.1 — AST parsing
- chromadb 1.5.9 — default vector store
- llama-index 0.14.22 — vector indexing and retrieval
- pydantic 2.13.4 — schema validation
- prompt-toolkit 3.0.52 — interactive CLI
- pathspec 1.1.1 — .metisignore glob matching
- unidiff 0.7.5 — diff parsing for review_patch

Optional providers: langchain-anthropic, langchain-google-genai, boto3+langchain-aws, llama-index-llms-langchain
Optional backend: llama-index-vector-stores-postgres

## Test Coverage

47 test files covering: core engine, reachability (tree-sitter + triage), graphs, provider integration (anthropic, gemini, bedrock, bedrock_mantle, azure, openai_compatible, llamacpp), memory (registry + backend), CLI, tools, plugins, SARIF output, diff parsing, models.
