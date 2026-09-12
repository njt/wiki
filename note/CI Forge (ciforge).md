# CI Forge (ciforge)

A zero-dependency Python CI tool that bundles ~25 scanning capabilities into a single package, replacing a constellation of paid services (Snyk, SonarQube, etc.) for solo developers. It combines static analysis, AI code review across 3 providers, supply chain vulnerability checking, and an MCP server interface into ~3,040 lines of pure Python. AGPLv3 with commercial dual licensing.

#tool #project #ci #code-quality #security #agents #mcp

---

## Architecture

ciforge follows a **flat module pipeline** pattern: ~30 independent scanner modules, each producing `List[Finding]`, orchestrated by a single `cli.py` (472 lines). There's no plugin system — new scanners are added as new Python modules that conform to an implicit `analyze() -> List[Finding]` contract.

The orchestrator's flow in `cli.py:126-191`:
1. Collects files to scan via `scanner.get_all_files()` (recursive walk) or `scanner.git_changed_files()` (incremental mode)
2. Filters files through `.ciforge-ignore` rules
3. For each file, runs per-file scanners (code_quality, secrets, config_validator, multi_ai) against the git diff
4. Runs repo-level scanners once (coverage, ai_reviewer, l10n, metrics, mobile_lint, etc.)
5. Optionally runs explicit scanners gated by CLI flags (dead_code, vuln_scan, duplication, cloud_cost, etc.)
6. Formats output as markdown or self-contained HTML with inline CSS
7. Optionally auto-fixes, creates PRs, sends Discord notifications, publishes to dashboard

Every scanner failure is caught and silently converted to an empty result list — ciforge will never block a CI pipeline.

### Key files

| File | Lines | Role |
|------|-------|------|
| `cli.py` | 472 | Orchestrator, arg parsing, output formatting |
| `scanner.py` | 82 | Finding dataclass, git diff/file helpers |
| `multi_ai.py` | 154 | Multi-provider AI review (OpenAI/Anthropic/Ollama) |
| `auto_fixer.py` | 163 | AI-driven PR generation with hallucination guard |
| `dead_code.py` | 133 | Cross-file dead code detection |
| `duplication.py` | 74 | AST-based structural code duplication |
| `vuln_scan.py` | 60 | Supply chain CVE via OSV.dev API |
| `mcp_server.py` | 86 | MCP stdio JSON-RPC server |
| `code_quality.py` | 64 | Regex lints + AST cyclomatic complexity |

## Key Techniques

### Multi-provider AI with zero dependencies

`multi_ai.py` calls OpenAI, Anthropic, and Ollama APIs using only `urllib.request` — no SDK packages. Each provider gets a dedicated function with the correct request body shape. The system prompt forces JSON output, and the parser extracts JSON by finding the first `[` and last `]` in the response — tolerating models that add text around the JSON. This is a pragmatic trade-off: simple to implement, but fragile to models that don't output clean JSON arrays.

### AST-based structural duplication

`duplication.py` parses Python files with `ast`, extracts function bodies, normalizes by stripping docstrings, serializes the AST to a string via `ast.dump()`, and flags structurally identical bodies as duplicates. This catches identical logic even when variable names differ. It filters out trivial bodies (single `pass`/`return`), `NotImplementedError` stubs, and functions under 2 lines.

### Hallucination guard in auto-fixer

`auto_fixer.py:112-119`: after the AI rewrites a Python file, `ast.parse()` validates the output. If parsing fails, the original content is restored. This is a simple but effective deterministic guard against the most common LLM code-generation failure mode — producing syntactically invalid code.

### Regenerative feedback loop

`telemetry.py` implements two feedback channels: `CIFORGE_CRASH_WEBHOOK` captures sanitized stack traces when scanners fail on edge-case code, and `CIFORGE_AI_WEBHOOK` aggregates AI reviewer findings to build a dataset for converting AI-caught patterns into fast deterministic static rules.

### Diff-first scanning

Most scanners operate on the git diff via `_extract_diff_sections()` which parses unified diff `@@` headers to track correct line numbers through hunks. This means findings are anchored to the actual new file line numbers, not approximations, and only new/changed code is scanned.

## Design Decisions

**Single-file modules with no shared infrastructure.** Each scanner is self-contained, importing only `Finding` from `scanner.py` and stdlib. This makes adding new scanners trivial (create a new file, add an `analyze()` function, wire it into `cli.py`) but means there's duplication — the AST parsing pattern appears in code_quality, dead_code, duplication, blast_radius, and prompt_scan, each with slightly different error handling.

**Breadth over depth.** With ~30 scanners averaging 50-80 lines, no single scanner is deeply sophisticated. The secret scanner handles 3 regex patterns. The IaC scanner handles 3 specific anti-patterns. The cloud cost estimator uses hardcoded monthly estimates. The developer value is catching 80% of what 15 specialized tools would catch, from a single command.

**Graceful degradation over correctness.** Every scanner is wrapped in a try/except that returns `[]` on failure. The top-level `main()` catches all exceptions and exits 0. False negatives are explicitly preferred over false CI failures. This makes ciforge safe to drop into any pipeline, but means it can silently miss real issues.

**Immediate non-blocking scanning over thoroughness.** The main loop iterates files sequentially with no concurrency. AI API calls per file are serial, not parallelized. The `--incremental` flag partially mitigates this by only scanning changed files, but there's no batching, no concurrency, no caching of AI results across similar files.

### What's distinctive

- **No dependencies at all** — not even `requests`. Everything is stdlib Python 3.8+.
- **MCP server as a distribution channel** — the CI tool becomes a tool that coding agents can invoke directly.
- **Four UX surfaces** — CLI flags, interactive wizard menu, drag-and-drop GUI (tkinter), and natural language chat.
- **AGPLv3 with commercial dual licensing** — the solo dev gets it free, enterprises pay via GitHub Sponsors.

## Comparison Notes

Unlike **Metis (ARM's AI security code reviewer)**, which does deep tree-sitter call-graph analysis across 19 languages, ciforge's code quality module is Python-only for AST and regex-based for everything else. But ciforge's breadth (secret detection + IaC + dead code + cloud cost + l10n + etc.) far exceeds Metis's focused scope.

Unlike **OpenCodeReview (Alibaba)**, which has a hybrid deterministic+agent architecture with concurrent subagents, ciforge uses a simple flat pipeline with optional single-model AI review. ciforge sends the raw diff with no context window management.

Unlike **semantic version control (sem)**, which uses tree-sitter for true entity-level diffs, ciforge operates on raw unified diff output — it doesn't understand language semantics beyond Python AST for specific modules.

Unlike commercial tools (Snyk, SonarQube, etc.), ciforge has no proprietary vulnerability database — it queries the public OSV.dev API for CVEs. This is both a weakness (misses non-OSV CVEs) and a strength (zero vendor lock-in, zero license cost).

The MCP server is notable: ciforge is one of very few CI tools that exposes itself via MCP for direct agent invocation. See [[Building Agents for Production Systems with MCP]] for the broader MCP-as-integration-layer pattern.

Related pages: [[Guardrails and Feedback Loops]] (the linters-beat-prompts philosophy), [[Security and Sandboxing]] (prompt injection detection via `prompt_scan.py`), [[Agentic Code Review]] (for the AI code review approach ciforge takes), [[sem]] (for the tree-sitter-based alternative to ciforge's diff scanning).

---
*Sources: [[raw/ciforge]]*
*Last updated: 2026-07-11*
