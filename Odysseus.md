# Odysseus

Self-hosted AI workspace — the open-source equivalent of ChatGPT/Claude's UI experience, running on your own hardware. Chat, agents with tools, deep research, document editing, email with AI triage, calendar (CalDAV), notes/tasks, and model serving — all in one Python FastAPI monolith with a vanilla JS frontend.

**Why it matters**: Most self-hosted chat UIs stop at chat. Odysseus goes all-in on *agency*: 50+ tools, MCP integration, persistent memory, skill extraction, and scheduled background tasks. It treats the LLM as an operating system kernel — everything else (email, calendar, documents, models) is a peripheral the agent can control.

## Architecture

Python FastAPI monolith (~77K lines) + vanilla JS SPA frontend (~96K lines). SQLite for structured data, JSON files for sessions/memory, ChromaDB + fastembed for vector search.

**Key modules**:
- `app.py` — FastAPI entry point, middleware, route registration
- `src/llm_core.py` (1671 lines) — Universal LLM client: OpenAI, Anthropic, Ollama, OpenRouter. Circuit breaker, response cache, connection pooling
- `src/agent_loop.py` (2493 lines) — Streaming agent loop with ~600 lines of operational prompt engineering
- `src/tool_implementations.py` (4457 lines) — All 50+ tool execution functions
- `src/chat_processor.py` — Hybrid memory retrieval (BM25 + vector), RAG, web search injection
- `src/context_budget.py` — Adaptive input budget scaling from 6K to 85% of context window
- `src/context_compactor.py` — Cursor-style self-summarization at 85% context threshold
- `src/deep_research.py` (917 lines) — IterResearch: Think→Search→Extract→Synthesize loop
- `services/memory/` — Memory service with skill extraction from conversations
- `companion/` — LAN pairing system for phone-as-thin-client

**Frontend**: No framework. Vanilla JS organized by feature: `chat.js` (4948 lines), `document.js` (9736 lines), `slashCommands.js` (6207 lines), `settings.js` (5094 lines), `emailLibrary.js` (5217 lines). PWA with service worker, responsive, touch gestures.

## Key Techniques

### Dual Tool Execution Model
The agent supports **two** tool execution paths that converge on the same dispatcher:
1. **Fenced code blocks** — the LLM writes ` ```bash\nls -la\n``` ` in markdown, parsed by regex. Works with ANY model.
2. **Native function calling** — OpenAI-style `tool_calls` via JSON schemas for capable models.

This is the critical design insight: most frameworks lock you into one approach. Odysseus lets you use the same agent with a 3B local model or Claude Opus, degrading gracefully. The tool schemas module (`tool_schemas.py`, 1358 lines) defines the function schemas; `function_call_to_tool_block()` normalizes native calls into the internal ToolBlock format.

### Prompt Engineering as System Design
The ~600-line agent preamble isn't just a system prompt — it's an **operations manual**. It encodes:
- Tool anti-patterns ("NEVER use bash to create files")
- Email UID semantics (IMAP UIDs vs display row numbers)
- Cookbook lifecycle (model serving, not recipe management)
- UI link conventions (`[Name](#kind-id)` format for clickable entity links)
- Error recovery protocols ("AFTER A TOOL FAILS, DO NOT GO SILENT")
- Multi-account routing rules
- Background job directives (`#!bg` prefix for long-running commands)

The prompt IS the product. Without this density of operational knowledge, the agent would be theoretically capable but practically useless.

### Dead-Host Circuit Breaker
In-memory tracking of upstream LLM failures: 2 consecutive failures → 20s cooldown. Thread-safe via `threading.Lock()`. Any success resets the counter. Prevents one misconfigured local model from jamming all chats — a real problem the author hit and fixed (issue #659).

### Hybrid Memory: BM25 + Vector
Not purely embedding-based. `chat_processor.py` computes BM25-style IDF-weighted keyword scores from the memory corpus, optionally combined with ChromaDB vector similarity. Recency tiebreaks within score buckets. RAG threshold: 0.35. This catches exact keyword matches that pure embedding search misses.

### Adaptive Context Budget
`context_budget.py` auto-scales: if the user didn't set a budget, use 85% of the model's discovered context window (capped at 200K). A 128K model gets ~109K; a 4K model stays at 3.4K. The user doesn't need to know their model's context size.

### Skill Extraction from Conversations
`services/memory/skill_extractor.py` uses an LLM to extract reusable procedures from conversations. Skills are stored as formatted markdown and loaded as few-shot examples. This creates a self-improving loop — the agent gets better at your specific tasks the more you use it.

## Design Decisions

| Decision | Why | Trade-off |
|----------|-----|-----------|
| **Monolith, not microservices** | One `docker compose up`, operational simplicity | No horizontal scaling; one crash takes down everything |
| **Prompt-based tools as default** | Works with any model, including tiny local ones | Consumes more context tokens; regex parsing can fail |
| **JSON files for sessions/memory** | Simple to inspect, backup, git-track | Not great for concurrent access |
| **No auth by default** | Zero-friction first run | Network-exposed if you bind to 0.0.0.0 |
| **Massive agent prompt** | Makes the agent dramatically more capable | Eats context budget; hurts small local models |
| **Vanilla JS, no framework** | Zero build step, simple deployment | 96K lines without framework structure; CSS is "Calypso's island" |
| **SQLite for structured data** | Zero-config, serverless, portable | No concurrent write scaling (fine for single-user) |

## Comparison

**vs [[Open WebUI]]**: Open WebUI is more polished with a larger community. Odysseus has deeper agent capabilities (50+ tools, scheduled tasks, skill extraction) and broader scope (email, calendar, documents built-in, not plugin-dependent).

**vs [[PiClaw]]**: Both are self-hosted AI workspaces in Docker. Odysseus is more feature-rich (calendar, email, deep research). PiClaw is simpler, more focused.

**vs [[clawdBot]]**: clawdBot works across messaging platforms (Telegram, Discord, etc). Odysseus is a web UI. clawdBot is conversation-first; Odysseus is workspace-first (documents, calendar, email are first-class, not just chat topics).

**vs [[LibreChat]]**: LibreChat is a ChatGPT clone with multi-model support. Odysseus is a full workspace with agent automation. Different philosophies: LibreChat replicates a hosted service; Odysseus builds something new.

**vs Claude Code / Codex**: CLI-first coding agents. Odysseus has a browser UI and broader scope — it can code (shell, file ops, Python) but also triage email, manage calendars, and edit documents. Same fundamental pattern (agent with tools) but different surface area.

## Critical Take

Odysseus is what happens when a single developer refuses to accept that "self-hosted AI" means "chat with a model selector dropdown." It's ambitious to the point of being slightly unhinged — an email client, calendar, document editor, model server manager, and research engine bolted onto a chat UI, all driven by one of the most detailed agent prompts in open source.

The architecture is pragmatic, not elegant. JSON files next to SQLite, regex tool parsing next to native function calling, vanilla JS next to FastAPI — but it WORKS. The dual tool execution model (fenced blocks + native calling) is genuinely innovative and should be standard in every agent framework.

The main weakness is the tension between the massive agent prompt and small local models — the ROADMAP acknowledges this. The frontend CSS is a known problem area. But these are scaling/refinement issues, not fundamental design flaws.

For anyone building a personal AI agent, Odysseus is worth studying for three things: (1) the dual tool execution model, (2) the density and specificity of its agent prompt, and (3) the adaptive context budget. These aren't academic ideas — they're battle-tested solutions to real problems.

#tool #project #agents #self-hosted #local-first #chat #mcp

## Related
- [[Personal Agents]] — Hub page for personal AI frameworks
- [[PiClaw]] — Self-hosted AI workspace, simpler scope
- [[clawdBot]] — Personal AI across messaging platforms
- [[Local and Open Source Inference]] — Running models on your own hardware
- [[Self-Hosted LLMs]] — The self-hosted LLM landscape
- [[A Deep Dive on Agent Sandboxes]] — Agent security patterns
- [[Building Agents for Production Systems with MCP]] — MCP as agent integration layer
- [[Agent Memory and Context]] — Memory architectures for agents

*Source: [github.com/pewdiepie-archdaemon/odysseus](https://github.com/pewdiepie-archdaemon/odysseus) — fetched 2026-06-05*
