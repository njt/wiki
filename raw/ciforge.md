---
url: https://github.com/Tahiram32/ciforge
title: CI Forge (ciforge)
author: Tahiram32
date_fetched: 2026-07-11
date_published: 2026-07-09
version: 6.1.0
license: AGPLv3
---

# CI Forge (ciforge)

Zero-dependency Python CI tool (~3,040 lines) that bundles ~25 scanning/code-quality capabilities into a single package, with multi-provider AI review (OpenAI, Anthropic, Ollama), an MCP server interface, and four user interfaces (CLI, interactive wizard, desktop GUI, chat). Replaces a constellation of paid services (Snyk, SonarQube, etc.) for solo developers. AGPLv3 with commercial dual licensing via GitHub Sponsors.

## Architecture

The project follows a **flat module pipeline** pattern — not a plugin system with dynamic discovery, but a collection of independent scanner modules unified by a single `cli.py` orchestrator. Every scanner conforms to the same implicit contract: a function (usually `analyze()`) that returns `List[Finding]`, where `Finding` is a shared dataclass (`file`, `line`, `message`, `severity`).

### Entry Point

`cli.py` (472 lines) is the monolith orchestrator. It:
1. Parses ~30 CLI flags via argparse
2. Collects files to scan (all files recursively, or git-changed-only with `--incremental`)
3. Runs a set of per-file scanners against each file's diff
4. Runs a set of repo-level scanners once
5. Optionally runs explicit scanners gated by flags
6. Formats output as markdown or self-contained HTML with inline CSS
7. Handles severity thresholds, badge generation, auto-fix, and PR automation

The main loop iterates files and runs per-file scanners inline — no concurrency, no batching. Explicit scanners default to on when no explicit scanner flag is provided (dead_code, vuln_scan, dupe_scan), giving sensible defaults for `ciforge --repo .`.

### Scanner Modules

Each module in `src/ciforge/` is independent — they import only `Finding` from `scanner.py` and standard library modules. The file tree:

```
src/ciforge/
  __init__.py         — empty (just the name)
  cli.py              — orchestrator, 472 lines
  scanner.py          — Finding dataclass, git diff/file helpers, 82 lines
  code_quality.py     — regex patterns + AST cyclomatic complexity, 64 lines
  secrets.py          — regex-based secret detection (AWS keys, GitHub tokens), 26 lines
  config_validator.py — JSON/YAML/TOML/XML validation, 56 lines
  coverage.py         — reads .ciforge-coverage.json, 20 lines
  ai_reviewer.py      — OpenAI-only diff review, 50 lines
  multi_ai.py         — multi-provider AI review (OpenAI/Anthropic/Ollama), 154 lines
  auto_fixer.py       — AI-driven PR generation with hallucination guard, 163 lines
  fixer.py            — deterministic auto-fix (debug statements, YAML tabs), 55 lines
  ignore.py           — .ciforge-ignore file parsing (fnmatch + substring), 31 lines
  vuln_scan.py        — supply chain CVE check via OSV.dev API, 60 lines
  iac_scan.py         — Dockerfile/Terraform/docker-compose security scanning, 42 lines
  duplication.py      — AST-based structural code duplication detection, 74 lines
  dead_code.py        — dead code detection via cross-file name matching, 133 lines
  blast_radius.py     — Python import coupling analysis, 44 lines
  cloud_cost.py       — hardcoded resource cost estimation from Terraform, 40 lines
  load_test.py        — concurrent HTTP load testing with timing, 42 lines
  schema_guardian.py  — SQL breaking change detection (DROP TABLE/COLUMN), 15 lines
  prompt_scan.py      — LLM prompt injection detection via AST, 48 lines
  mcp_scan.py         — MCP config file validation, 25 lines
  mcp_server.py       — stdio JSON-RPC MCP server implementation, 86 lines
  changelog.py        — conventional-commit changelog generator, 79 lines
  config_drift.py     — env file key comparison, 87 lines
  semantic_bump.py    — conventional-commit semantic version bumper, 47 lines
  arch_diagram.py     — Mermaid diagram generator from Python imports, 96 lines
  mobile_lint.py      — Flutter/Android/iOS config checker, 165 lines
  discord_notify.py   — Discord webhook sender, 18 lines
  badges.py           — SVG badge generator, 45 lines
  community.py        — first-time contributor greeting, 19 lines
  l10n.py             — localization key sync checker, 39 lines
  metrics.py          — PR metrics from git log, 26 lines
  assets.py           — large file detection, 25 lines
  wizard.py           — interactive arrow-key menu, 50 lines
  gui.py              — tkinter-based desktop GUI, 66 lines
  chat.py             — interactive chat with codebase, 44 lines
  ci_cost.py          — CI cost waste estimation, 30 lines
  auto_update.py      — automated dependency version upgrades, 80 lines
  telemetry.py        — crash/AI finding telemetry to Discord webhooks, 58 lines
  pr_describe.py      — AI PR description from git diff, 173 lines
  github_app.py       — GitHub App webhook server, 42 lines
  universal_scanner.py — regex-based lints for C/C++/Java/Go/Rust, 25 lines
```

### Core Abstractions

1. **`Finding` dataclass** (`scanner.py:24-28`): The universal output type — every scanner produces `List[Finding]`. Four fields: `file`, `line`, `message`, `severity` (low/medium/high/critical). This is the schema that unifies all 30+ scanners.

2. **Diff-first scanning** (`scanner.py:43-58`): Most scanners operate on the git diff, not the full file. The `_extract_diff_sections()` function parses unified diff format to extract only added lines with their correct line numbers. This is the key design decision — new code is what matters for CI, not existing code.

3. **`IgnoreRules`** (`ignore.py`): A singleton loaded from `.ciforge-ignore` that supports two modes: file-level (fnmatch patterns) and secret-level (substring matching). Used by both `cli.py` for file filtering and `secrets.py` for whitelisting test credentials.

## Key Techniques

### Multi-provider AI with Zero Dependencies

`multi_ai.py` calls OpenAI, Anthropic, and Ollama APIs using only `urllib.request` — no `openai`, `anthropic`, or `requests` packages. Each provider has a dedicated function that formats the request body appropriately for that API's shape (OpenAI: `messages` array, Anthropic: `system` + `messages`, Ollama: `model` + `messages` + `stream: false`). The system prompt forces JSON output by specifying "Return ONLY a valid JSON array." The parser extracts JSON by finding the first `[` and last `]` in the response — a pragmatic approach that tolerates models that prepend/appended text around the JSON.

### AST-Based Structural Duplication

`duplication.py` goes beyond text similarity. It parses Python files with `ast`, extracts function bodies, normalizes them by stripping docstrings, and serializes the AST to a string via `ast.dump()`. Functions with identical AST serializations are flagged as duplicates. It filters out trivial bodies (single `pass`/`return`), `NotImplementedError` stubs, and functions under 2 lines. This catches structurally identical code even when variable names differ — something regex-based approaches miss.

### Hallucination Guard in Auto-Fixer

`auto_fixer.py:112-119` implements a critical safety check: after the AI rewrites a file, if it's a Python file, `ast.parse()` is called on the output. If parsing fails (`SyntaxError`), the original content is restored. This prevents AI-generated invalid code from corrupting the repo — a pragmatic deterministic guard against the most common category of LLM code-generation failures.

### Git Diff Line Number Tracking

`scanner.py`'s `_extract_diff_sections()` parses unified diff `@@` headers to track correct line numbers through hunks. It increments the line counter on `+` lines (added content), skips `-` lines (removed), and increments on context lines. This means every finding is anchored to the correct line in the new file — not an approximation.

### Dead Code Detection via Full-Repo Scanning

`dead_code.py` uses a two-phase approach: Phase 1 collects all function/class definitions across all `.py` files (skipping dunder methods, pytest fixtures, test files). Phase 2 concatenates the content of all *other* files and does a simple substring match for each defined name. This is brute-force but effective — it catches names that are defined in one file and never referenced elsewhere, even when the usage pattern is complex. The trade-off is O(n²) in the number of definitions and no support for qualified imports (`import module; module.func` won't match `func`).

### OSV API Integration for CVE Detection

`vuln_scan.py` queries the Google-maintained OSV.dev API for known vulnerabilities. It parses `requirements.txt` with regex (`^([a-zA-Z0-9_\-]+)[=!<>~]+(.*)$`) and `package.json` with `json.loads()`, then queries OSV per package per ecosystem. Version cleaning strips semver modifiers (`^`, `~`, `=`, `>`, `<`) to extract bare version numbers. The API is called with a 10-second timeout, and failures are silently swallowed — the scanner never blocks a CI pipeline.

### Regenerative Crash Telemetry

`telemetry.py` implements a "get smarter from failures" loop: when any scanner crashes, the exception details are POSTed to a configurable Discord webhook (`CIFORGE_CRASH_WEBHOOK`). The traceback is sanitized to just filenames and line numbers (no source code). AI review findings can also be aggregated via `CIFORGE_AI_WEBHOOK` to build a dataset for converting AI-caught patterns into deterministic static rules — closing the loop from AI review to static analysis.

## Design Decisions

### Optimized for: Zero dependencies, solo developer UX

The project's defining constraint is zero pip dependencies. Everything uses stdlib: `urllib.request` for HTTP, `ast` for Python parsing, `subprocess` for git, `json` for parsing, `argparse` for CLI. This eliminates dependency conflicts, supply chain risk, and install friction. The trade-off is manual HTTP request building and no retry logic, no connection pooling, no streaming.

### Optimized for: Broad surface area over depth

With ~30 scanners averaging 50-80 lines each, no single scanner is deeply sophisticated. The code quality scanner handles TODO detection, debug statements, large functions, and cyclomatic complexity — but has no support for any language other than Python for AST analysis. The secret scanner handles 3 regex patterns. The IaC scanner handles 3 specific anti-patterns. The value proposition is the breadth — a single tool that catches 80% of what 15 specialized tools would catch — not depth in any one category.

### Optimized for: Non-blocking CI

Every scanner is wrapped in `_run_scan()` which catches all exceptions and returns `[]`. The CLI's `main()` wraps the entire run in a try/except that prints a generic error message and exits 0. This design means ciforge will never block a CI pipeline, even if it encounters code it can't parse. The telemetry system then captures the failure for later improvement. The explicit stance: "false negatives are better than false CI failures."

### Sacrificed: Correctness of scan results

The dead code detector uses naive substring matching — `name in combined_content` — which means a function named `run` will be flagged as dead if no other file contains the literal word "run" (unlikely) but will incorrectly survive if `run` appears anywhere in any context. The duplication detector serializes AST to string and compares — which means whitespace/comment differences don't matter, but function naming and variable naming do. The cloud cost estimator uses hardcoded monthly costs (`aws_instance: 50`, etc.) rather than actual AWS pricing APIs. These are shortcuts that work for common cases but would not satisfy a security audit or financial controller.

### Sacrificed: Concurrency

The main scanning loop iterates files sequentially. For a large repo, calling `multi_ai.analyze()` (which makes an HTTP call to an LLM API) per file is expensive and slow. The incremental mode (`--incremental`) mitigates this by only scanning changed files, but there's no concurrent execution of independent HTTP calls or parallel file processing.

### Innovation: MCP Server as Stdio Interface

The MCP server (`mcp_server.py`) implements the Model Context Protocol over stdio JSON-RPC, making ciforge directly callable as a tool from Claude Desktop, Cursor, or any MCP-compatible agent. This is a novel distribution channel — the CI tool itself becomes a tool that agents can invoke. The MCP tool `ciforge_scan` runs a subset of scanners (vuln, dead_code, iac, dupe) and returns findings as JSON.

## Comparison Notes

Unlike **Metis (ARM's AI security code reviewer)**, which uses tree-sitter for deep call-graph analysis across 19 languages, ciforge's code quality module is Python-only for AST analysis and regex-based for other languages. The depth gap is significant, but ciforge's breadth (secret detection + IaC + dead code + cloud cost + l10n + etc.) far exceeds Metis's focused scope.

Unlike **OpenCodeReview (Alibaba)**, which has a hybrid deterministic+agent architecture with per-file concurrent subagents and dual-threshold context compression, ciforge uses a single flat pipeline with optional single-model AI review. ciforge's AI review sends the raw diff to the model with no context window management; OpenCodeReview has sophisticated truncation and batching.

Unlike **sem (semantic version control)**, which uses tree-sitter for true entity-level diffs, ciforge operates on raw unified diff output from git — it doesn't understand language semantics beyond Python AST for specific modules.

Unlike commercial tools (Snyk, SonarQube), ciforge has no proprietary vulnerability database — it relies solely on the public OSV.dev API for CVE detection. This means it misses vulnerabilities not yet in OSV, but also means it has zero license cost and zero vendor lock-in.

The MCP server integration is distinctive: ciforge is one of very few CI/code-quality tools that exposes itself via MCP. This positions it at the intersection of two trends — the "CI as an agent tool" pattern where agents can invoke quality checks as part of their workflow. Related to [[Building Agents for Production Systems with MCP]].

---
*Last updated: 2026-07-11*
