---
url: https://github.com/Tahiram32/ciforge
title: "CI Forge (ciforge)"
author: Tahiram32
date_fetched: 2026-07-11
date_published: 2026-07-09
topics:
  - guardrails-and-feedback-loops
  - developer-tools
---

CI Forge (ciforge) is a zero-dependency Python CI tool (~3,040 lines) that bundles roughly 25 scanning and code-quality capabilities into a single package. It includes multi-provider AI review (OpenAI, Anthropic, Ollama via raw `urllib.request`), an MCP server interface, and four user interfaces (CLI, interactive wizard, desktop GUI, chat). Licensed AGPLv3 with commercial dual licensing via GitHub Sponsors, it targets solo developers as a free replacement for a constellation of paid services (Snyk, SonarQube, etc.).

The architecture is a flat module pipeline — independent scanner modules, each returning `List[Finding]` (a shared dataclass with file, line, message, severity), orchestrated by a single `cli.py`. Scanning is diff-first: most scanners operate on git diff output, not full files, anchoring findings to correct line numbers via unified-diff header parsing. Every scanner is wrapped in exception handlers so a crash in one never blocks a CI pipeline.

Notable technical choices include AST-based structural duplication detection (comparing `ast.dump()` output, catching identical code even with different variable names), a hallucination guard in the auto-fixer that validates AI output with `ast.parse()` before writing, and regenerative crash telemetry that posts failures to a Discord webhook to feed a "patterns → static rules" improvement loop. Supply-chain CVE detection queries the public OSV.dev API; dead code detection uses a brute-force two-phase substring matching approach across the full repo.

The defining trade-offs: zero pip dependencies at the cost of manual HTTP, no retry logic, and no connection pooling; breadth across ~30 scanners (averaging 50–80 lines each) over depth in any single category; and sequential file processing with no concurrency. The MCP server (`mcp_server.py`) implements the Model Context Protocol over stdio JSON-RPC, making ciforge directly callable from Claude Desktop or any MCP-compatible agent — a novel distribution channel for a CI tool.

Compared to tools like Metis (ARM), OpenCodeReview (Alibaba), and sem, ciforge trades depth for breadth and lacks proprietary vulnerability databases (relying solely on OSV.dev). Its distinctive feature is the MCP integration, positioning it at the intersection of CI tooling and agent-tool patterns.
