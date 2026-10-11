# AI Layer — DuckDB Brain and LSP MCP Servers for Claude Code

ai-layer is an MIT Claude Code plugin (Kris Codes LLC) that bolts a persistent memory and a code-intelligence layer onto a Claude Code session: a DuckDB "brain" that survives restarts, a TypeScript language server that answers "where is this symbol and who calls it" without reading whole files, and 88 skills / 25 agents spanning a plan→review→quality-gate→ship lifecycle. Its most interesting contribution is not the features but its failure-mode honesty: a three-mode resolver that refuses to silently degrade.

---

The README's engineering thesis is that agent tooling should have *no silent degradation paths*. The brain runs in exactly one of three modes — `gateway` (commercial Koinessa control plane, not in the open release), `local` (single-machine DuckDB file), or `setup` (dependencies not installed, one status tool pointing at `/ai-layer:doctor --fix`). Crucially, the precedence is "gateway then local, and it is not a fallback":

> Precedence is `gateway` then `local`, and it is not a fallback: a configured gateway that is unreachable stays in `gateway` mode and its writers fail open with one named line, rather than quietly switching you to a different brain.

That is a subtle and correct call. A silent downgrade would produce an agent that *appears* to remember while actually writing to a different store — the kind of partial failure that is far worse than an error. Compare the softer "best-effort" stance in most harness write-ups ([[Harness Engineering for Coding Agent Users]], [[The Harness Is the Company]]); ai-layer treats ambiguity as a bug and spends real code on refusing it. Even the documentation is wired into this discipline: `tests/docs/readme.test.mjs` validates the README's architecture diagram against the resolver, "a fourth mode fails the test rather than shipping a stale diagram."

The second thesis is retrieval over reading. The retrieval-first SOP routes symbol lookups through `typescript-lsp` (`doxygen-lookup`, `sync-symbols`) instead of grep, with `LSP_WARMUP_FILES` opening entry points on startup so tsserver has the right files loaded. This is the same idea JetBrains argues for in [[Building a RAG Pipeline for Semantic Code Search (JetBrains)]] — the agent should query a code index, not scan text — but implemented with a boring, proven tool (a language server) rather than embeddings.

The third is memory as a local file. `CLAUDE_BRAIN_DB` defaults to `~/ClaudeBrain/brain.duckdb`; the pre-commit hook ingests session transcripts from `~/.claude/projects/` into it at commit time, and a Stop hook writes a review file plus a row. It is a deliberately modest design — no graph, no hybrid search in local mode — that aligns with [[SQLite is All You Need for Durable Workflows]]' argument that single-file embedded databases are the right substrate for durable agent state, and with [[Three-Tier Memory]]'s layered view of what agents should remember. The open-core boundary is worth noting: the local brain is free and MIT, while the coordination plane (shared multi-session brain, hybrid search, graph) is the commercial gateway. The coordination tier is exactly what an open-core company would charge for.

Privacy is stated as an invariant rather than a policy: local mode "reaches nothing," gateway mode talks to exactly one configured host, and "No analytics, no telemetry, no account, no phone-home, in any mode. If that ever changes, it will be stated here and in `CHANGELOG.md` with a version bump."

## Take

- **Fail loud, not soft.** The no-silent-degradation resolver and the setup-mode MCP servers that only expose a status tool are the best pattern here. Most plugins would quietly fall back; this one names its failure and the fix in one line.
- **Docs-as-tests.** Testing the README diagram against the resolver is a small, transferable trick for any repo whose architecture doc is load-bearing.
- **Scope honesty.** 88 skills and 25 agents sounds like plugin sprawl, but the core/marketing/infra split keeps optional packs optional, and "core never requires either."
- **The LSP move generalizes.** Any agent harness serving typed languages should reach for the language server before reaching for grep or embeddings; this plugin is a concrete, MIT-licensed existence proof.
- **Skeptical note.** The lifecycle (create-prd → plan-review → quality-gate → ship) is a heavy process for a plugin, and the GitLab snapshot distribution (one-commit releases, MRs "re-applied upstream") makes external contribution friction high — you can read the artifact but not its history.

## Related

- [[Steering Claude Code]] — Anthropic's taxonomy of instruction-delivery mechanisms; ai-layer is a worked example of skills + hooks + MCP servers composing under that model.
- [[Claude Code Skills System]] — the official skills architecture ai-layer is built on; its 88 skills are a stress test of the SKILL.md format at scale.
- [[SQLite is All You Need for Durable Workflows]] — strengthens the embedded-single-file-database thesis with DuckDB as a sibling data point for agent memory.
- [[Building a RAG Pipeline for Semantic Code Search (JetBrains)]] — complicates the "index over grep" consensus: JetBrains reaches for semantic search where ai-layer gets symbol precision from a language server, two different answers to the same retrieval problem.

---
*Sources: [[raw/ai-layer]], [[summary/ai-layer]]*
*Last updated: 2026-10-11*
