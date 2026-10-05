# Odysseus (odysseus-dev)

Odysseus is a self-hosted, single-user AI workspace — chat, agent loop, deep research, documents, email, calendar, notes, model serving — as one Python FastAPI monolith (~42K lines of core Python plus a vanilla-JS SPA) deployed via Docker. This ingest of the *matured* project (now under `odysseus-dev`, AGPL, distro-packaged, CI-published images) updates the earlier [[Odysseus]] note from its June 2026 state, and what changed is telling: the agent loop grew from ~2,500 to ~6,500 lines, and almost all of the growth is security architecture, not features.

---

## Architecture

- **`app.py`** — deliberately "slim orchestrator": FastAPI app, middleware, route registration, plus Windows platform patches (ProactorEventLoop, MIME fixes, `.env` BOM tolerance — a real bug #142).
- **`src/agent_loop.py`** (6,455 lines) — streaming multi-round tool loop wrapped around `stream_llm()`. Tools are invoked two ways that converge on one dispatcher: fenced code blocks in the model's own output, and native function-calling schemas (`FUNCTION_TOOL_SCHEMAS`). Model-agnostic fallback (`stream_llm_with_fallback`) sits under the loop.
- **`src/` service layer** — ~113 modules: `context_compactor.py` (Cursor-style self-summarisation at 85% of the context window), `context_budget.py`, `deep_research.py` (IterResearch Think→Search→Extract→Synthesize), `chat_processor.py`, `task_scheduler.py`, `event_bus.py`.
- **`services/`** — memory (extractor + FAISS vector + skill extraction/import), research, hwfit (hardware-fit model recommender), TTS/STT, faces, docs.
- **`routes/`** — ~50 route modules, each owning its HTTP surface; `mcp_servers/` exposes email, image-gen, memory, and RAG as standalone MCP stdio servers for *external* agents.
- **`integrations/claude` and `integrations/codex`** — the workspace's memory/skills surface packaged as skills for Claude Code and Codex: the personal workspace becomes a tool other coding agents can call into.

## Key techniques

- **Layered tool security, not one bolt-on.** `tool_security.py` holds a single frozenset of built-in email tools from which fence tags, dispatch, schemas, *and* the non-admin blocklist all derive — so a new email tool can't become reachable without also being blocklisted. `tool_capabilities.py` tracks whether a tool ran with external untrusted context and whether its result "arms" a gate; `tool_approvals.py` approves on exact tool + document content digest, not tool name.
- **Prompt-injection containment as a protocol.** `prompt_security.py` wraps all external content in `<<<UNTRUSTED_SOURCE_DATA>>>` blocks, *escapes the guard markers inside the payload* to prevent sandbox breakout, and instructs the model never to mention the wrapper — injection defence treated as an engineering discipline with its own tests (`test_skill_index_prompt_injection.py`, ReDoS tests on tool parsers).
- **Adaptive context budget.** `context_budget.py` fixes a real bug where a 6,000-token default silently capped even 1M-context models: when the user hasn't set a budget, derive it from the discovered context window (85% headroom, 200K hard max).
- **Memory with an audit short-circuit.** `memory_extractor.py` fingerprints all of an owner's memories with an order-independent sha256; the LLM audit pass that dedupes/rewrites junk runs only when the fingerprint changed — saving a 30–120s LLM call per no-op tidy.
- **Hardware-fit tables as data.** `services/hwfit/fit.py` hard-codes memory bandwidth for every GPU from a 5090 down to a GB10, plus Apple unified-memory tables that prefer GPU core counts for binned chips — honest-by-construction speed estimates rather than vibes.
- **The repo practises what the wiki preaches**: a `specs/` directory of ~30 written specifications (auth-security, context-building, memory-skills, per-provider model specs) is committed alongside the code, and the test suite covers attacker-shaped cases (path confinement, owner scoping, IMAP leaks, runaway loops) rather than happy paths.

## Design decisions

Odysseus optimises for *one trusted operator*: single SQLite + JSON storage, owner-scoped everything, email/MCP capabilities admin-only, auth fail-closed (tests named `test_notes_fail_closed_auth`). It trades multi-tenant polish for breadth of agency — it will read your mail and run your shell, so the security surface is treated as a product feature. The Docker-first packaging with immutable `X.Y.Z-<sha>` tags shows the project graduating from hobby install to appliance.

## Comparison notes

- [[Odysseus]] — the earlier note on this same project (then `pewdiepie-archdaemon/odysseus`). Since then the agent loop doubled in size, prompt/tool security became its own subsystems, and Docker distribution replaced from-source installs; the thesis "LLM as OS kernel, everything else a peripheral" is unchanged.
- [[PiClaw]] — also a self-hosted AI workspace in a container, but PiClaw stays at the chat-UI layer; Odysseus goes further into email, calendar, and scheduled background tasks, and now exposes itself outward via MCP and coding-agent skills.
- [[clawdBot]] — clawdBot is messaging-platform-first, Odysseus is web-workspace-first; both bet that a personal agent needs persistent memory and local model flexibility.
- [[Agent Memory and Context]] — Odysseus is a working case study for this topic's themes: summarisation-based compaction, hybrid BM25+vector retrieval, and an LLM-audited fact store with a cheap-change-detection trick that most memory papers don't mention.

---
*Sources: [[raw/odysseus-github]], [[summary/odysseus-github]]*
*Last updated: 2026-10-05*
