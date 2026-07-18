---
url: https://github.com/langchain-ai/openwiki
title: OpenWiki
author: LangChain
date_fetched: 2026-07-18
date_published: 2025
---

# OpenWiki — Source Analysis

OpenWiki is a CLI tool (npm: `openwiki`, v0.2.0, MIT licensed) that generates and maintains agent-targeted documentation wikis for codebases and personal knowledge bases. Built by LangChain using their own `deepagents` orchestration framework, it takes the approach of running an AI agent (with filesystem tools, git access, and connector-based data ingestion) that reads source evidence and produces structured wiki output in the Google Open Knowledge Format (OKF) v0.1.

Language: TypeScript, targeting Node.js >= 22. ~19,600 lines of TypeScript across 83 source files, plus 44 test files. Primary dependencies: `deepagents` (the agent framework), `@langchain/core` (the LLM abstraction layer), `ink` (React-based terminal UI), `better-sqlite3` (checkpoint persistence), `posthog-node` (telemetry), `marked` (markdown rendering in terminal).

## Architecture

### Top-Level Structure

```
src/
  cli.tsx (4,019 lines)        — Ink-based terminal UI, command parsing, run state machine
  commands.ts                   — CLI command dispatch
  constants.ts                  — Provider definitions, model presets, env keys
  credentials.tsx               — Interactive credential setup wizard
  ingestion.ts (421 lines)     — Ingestion orchestration: deterministic pull → agent synthesis
  onboarding.ts                 — First-run onboarding config
  openwiki-home.ts              — ~/.openwiki directory layout
  startup.ts                    — Startup command resolution

  agent/
    index.ts (1,637 lines)     — Agent creation, model instantiation, stream processing
    prompt.ts (489 lines)       — System and user prompt generation (the real "source of truth")
    types.ts                     — OpenWikiCommand, OpenWikiRunEvent, RunContext
    utils.ts (479 lines)        — Git context, content snapshots, no-op detection
    docs-only-backend.ts         — FilesystemBackend wrapper restricting writes to wiki dirs
    frontmatter-validator.ts     — OKF front matter validation on every write/edit tool call
    index-middleware.ts          — Post-run deterministic index.md generation
    skills.ts                    — Bundled skill syncing
    vertex-surface.ts            — Gemini Enterprise model routing
    openai-chatgpt-oauth.ts      — ChatGPT OAuth provider

  connectors/
    types.ts                     — ConnectorRuntime, ConnectorIngestResult, ConnectorConfig
    registry.ts                  — 7 built-in connectors
    tools.ts                     — Connector tools exposed to the agent
    io.ts, mcp-client.ts, mcp-runtime.ts — MCP infrastructure
    sources/
      git-repo.ts, gmail.ts, hackernews.ts, slack.ts, web-search.ts, x.ts, mcp.ts

  auth/
    oauth.ts, providers.ts, tokens.ts, configure.ts, ngrok.ts — OAuth flows for connectors

  telemetry/
    — PostHog-based telemetry with env-var opt-out
```

### Data Flow for a Wiki Update Run

1. **CLI parses command** (`cli.tsx` → `commands.ts`)
2. **Startup resolves context**: mode (code/personal), credentials, model, thread ID
3. **Agent is created** (`agent/index.ts`):
   - Model instantiated based on provider (`createModel` factory: OpenAI, Anthropic, Gemini, Vertex, Bedrock, OpenRouter, openai-compatible)
   - Checkpointer: SQLite for chat (persistent), in-memory for init/update (stateless)
   - Backend: `CompositeBackend` wrapping a `OpenWikiLocalShellBackend` (docs-only write restriction) + `/skills/` filesystem
   - Middleware: `OpenWikiIndexMiddleware` (frontmatter validation on tool calls, index regeneration after agent)
   - Agent: `createDeepAgent` with filesystem tools, connector tools, system prompt, skill directory
4. **Stream events processed**: protocol events (v3) parsed into `OpenWikiRunEvent` stream — text chunks, tool start/end
5. **Post-run**: content snapshot hash compared with pre-run to detect real changes; update metadata written

### Ingestion Pipeline (personal mode)

```
openwiki ingest <target>
  → runOpenWikiIngestion()
    → for each matching source:
        1. Deterministic pull (connector.ingest()) → raw JSON files under ~/.openwiki/connectors/<id>/raw/
        2. Agent run: "Update the wiki from these raw files" → agent reads raw files, synthesizes wiki pages
```

### Key Architecture Decisions

**Two-phase ingestion**: Connectors do deterministic data fetching (API calls, OAuth, pagination, cursors) and write raw JSON. The LLM agent never sees credentials — it only reads the raw files. This separates the trust boundary: credential-handling code is TypeScript, not prompt-constrained.

**Wiki-first question answering**: The system prompt instructs the agent to inspect the synthesized wiki before raw connector data. Raw data is only consulted when the wiki is "missing the needed detail, clearly stale, ambiguous, contradicted, or the user explicitly asks for source-level evidence."

**Content-addressed no-op detection**: Before an update run, the system takes a SHA-256 hash of all wiki files (excluding `.last-update.json`). After the run, it hashes again. If identical, no metadata is written — the run is effectively a no-op. Additionally, git-head comparison can skip the run entirely when no source changes are detected.

**Post-run index generation**: Rather than having the agent maintain `index.md` files, the `index-middleware.ts` afterAgent hook reads all markdown files, parses their YAML front matter for `title` and `description`, and deterministically rebuilds every directory's `index.md`. The root index gets `okf_version: "0.1"` front matter; nested indexes are plain markdown with file/directory sections.

**Front matter validation middleware**: On every `write_file` or `edit_file` tool call that targets a wiki markdown file, `frontmatter-validator.ts` reads the persisted file, validates YAML front matter (requires `type` field, validates string/array types for standard fields), and injects a warning message into the tool result if invalid. This is an inline feedback loop — the agent sees the warning and can fix it.

**Gemini Enterprise multi-surface routing**: `vertex-surface.ts` routes model IDs to three different API surfaces: native Gemini (generateContent), Anthropic on Vertex (rawPredict via `@anthropic-ai/vertex-sdk`), and OpenAI-compatible MaaS (partner models like Llama, Mistral). Auth is uniformly ADC (Application Default Credentials) — no API keys.

**Gemini 3.x thought-signature workaround**: Gemini 3.x requires `thoughtSignature` on function-call parts for multi-turn tool calling. LangChain's streaming aggregator drops this signature. The fix: disable streaming for Gemini, route through `invoke()`/`generate` which preserves `outputVersion: "v0"` raw content parts.

**ChatGPT OAuth provider**: Uses ChatGPT subscription billing (Codex backend) instead of API key billing. Full OAuth flow with token refresh, account metadata capture.

**Code mode → AGENTS.md/CLAUDE.md injection**: On code-mode runs, OpenWiki adds `<!-- OPENWIKI:START -->…<!-- OPENWIKI:END -->` blocks to repo-root `AGENTS.md` and `CLAUDE.md`, telling coding agents to reference the `openwiki/` directory.

## Connector Architecture

7 built-in connectors, each implementing `ConnectorRuntime`:

| Connector | Backend | Credentials | Ingest Style |
|-----------|---------|-------------|--------------|
| git-repo | local-git | none | Compact manifests (path, branch, HEAD, status, changed files, recent commits) |
| google | direct-api | OAuth (Gmail API) | Fetches recent mail via query, writes `gmail-messages.json` |
| notion | mcp-http | MCP access token | Discovery via `openwiki_list_mcp_tools`, then MCP tool calls |
| slack | direct-api | OAuth | Self-message search + bounded recent conversation ingestion |
| x | direct-api | OAuth 2.0 PKCE | Home timeline, user posts, mentions, bookmarks, list posts |
| web-search | direct-api | TAVILY_API_KEY | Tavily search via LangChain, writes results JSON |
| hackernews | direct-api | none | Public HN feed + Algolia search API |

Connectors that can do deterministic pulling (`supportsAgenticDiscovery === false`) run in two phases: deterministic pull → agent synthesis. Connectors with `supportsAgenticDiscovery === true` (e.g., Notion via MCP) let the agent do discovery during the agent run.

Each connector gets:
- `~/.openwiki/connectors/<id>/config.json` — connector configuration
- `~/.openwiki/connectors/<id>/state.json` — cursors, last run timestamps, run summaries
- `~/.openwiki/connectors/<id>/raw/<run-id>/` — raw data from each ingestion run
- `~/.openwiki/connectors/<id>/logs/` — ingestion logs

## Prompt Engineering

The system prompt (`agent/prompt.ts`, ~500 lines) is the most carefully engineered component. Key patterns:

**Conditional output configurations**: The prompt is parameterized by `OpenWikiOutputMode` (`local-wiki` vs `repository`), producing different instructions for filesystem roots, wiki paths, search boundaries, and agent instruction file handling.

**Mode-specific behavior switches**: Chat, init, and update each get different instruction blocks. Chat: answer directly, don't modify docs. Init: build from scratch, max 8 pages, backlog deferred areas. Update: surgical edits only, soft diff budget, check no-op possibility.

**Source-specific synthesis policies**: Each connector has a hand-written synthesis policy (`createConnectorSynthesisGuidance`) telling the agent how to classify, route, and deduplicate content from that source. Gmail gets a 13-label classification taxonomy; Hacker News items default to watchlist unless corroborated; Slack messages route to commitments with Owner tags.

**Canonical cross-source files**: The personal mode prompt specifies canonical files: `quickstart.md` (navigation), `open-questions.md` (Active/Answered/Stale), `themes.md` (compact recurring signal index), `commitments.md` (work tasks with Owner), `personal-logistics.md` (non-work logistics), `sources/<connector>.md` (evidence index per source).

**Confidence labeling**: Four tiers: confirmed, source-backed, watchlist, saved-context. Each has distinct treatment — watchlist items stay out of quickstart, saved-context items don't imply truth.

**OKF compliance**: Explicit front matter requirements with a template, relationship modeling rules (every concept should connect to 2+ others), section quality rules (no thin pages, merge stubs).

**Run discipline**: Targeted discovery (no `glob **/*`), grep over full reads for large files, plan-first workflow (`_plan.md`), delete plan before finishing.

**Subagent discipline**: Parallelize read-only research for multi-domain repos, 1-2 subagents default, 3-4 only for small/medium repos. Subagents inspect and summarize; main agent synthesizes and writes.

## Telemetry

PostHog-based. Sends a single `openwiki_run` event per init/update run (not chat/auth/ingest). Records: command, outcome (success/failure/no-op), coarse error category, and at setup-time: brain mode, provider, connector names. Keyed by random install ID. Opt-out via `OPENWIKI_TELEMETRY_DISABLED=1` or `DO_NOT_TRACK=1`. CI runs tagged separately under a shared CI identifier.

## Skills

Two bundled skills (`skills/` directory, synced to `~/.openwiki/skills/` on every run):
- `write-connector/SKILL.md` — how to add a new built-in connector to the OSS repo
- `migrate-wiki-to-okf/SKILL.md` — migrate existing wiki content to OKF v0.1 format

## Tests

44 test files covering: telemetry, prompts, OKF prompt, skills, commands, startup, credentials, environment behavior, connector config overrides, model resolution, OAuth, OpenAI ChatGPT provider, Vertex surface, Gemini retry, redaction, code mode, checkpoint policy, docs-only backend, index middleware, utils, onboarding, run metadata, frontmatter validator, update noop detection, file system errors.

## Design Trade-offs & Notable Choices

**Monolith over multi-agent**: OpenWiki uses a single DeepAgent with subagent delegation for parallel read-only research (1-2 subagents typical). This is a deliberate choice for reliability over the approach of frameworks like DeerFlow (27-middleware pipeline) or Cloudflare's security audit skill (6 specialized agents). The trade-off: simpler orchestration, but less parallelism for large repos.

**Built-in connectors only**: No plugin marketplace, no dynamic connector loading. New connectors require a PR to the OSS repo. This optimizes for security (credential-handling code is reviewed) over extensibility. The `write-connector` skill provides the template, but the gate is explicit.

**Prompt engineering over tool complexity**: The system prompt is ~500 lines of carefully tuned instructions. The tools are standard filesystem operations (read, write, edit, ls, glob, grep, execute). There's no specialized knowledge-graph tool, no RAG pipeline, no embedding-based retrieval. The prompt IS the control surface.

**No embeddings**: Unlike QMD (BM25 + vector + LLM re-ranking) or WUPHF (BM25 + SQLite), OpenWiki uses no vector search or embeddings. Content discovery is done through grep, glob, and direct reads. This is a simplicity trade: no index freshness problems, but less semantic search capability.

**In-memory checkpoints for init/update**: Only chat mode gets persistent checkpointing (SQLite). Init and update runs use `:memory:` checkpoints, meaning they're stateless — each run starts fresh. This prevents checkpoint bloat but means there's no incremental state within a run.

**Deterministic indexes over agent-generated ones**: The index middleware generates `index.md` files programmatically from front matter metadata, not from agent output. This prevents the agent from creating bad indexes and ensures consistency, but means indexes are purely structural (file listings) rather than conceptual (curated overviews).

**Per-connector synthesis policies**: Each connector has a hand-written paragraph telling the agent exactly how to classify and route its content. This is essentially curriculum design — teaching the agent domain-specific editorial judgment. It's labor-intensive but produces more consistent output than generic "synthesize this" prompts.
