# OpenWiki

LangChain's CLI tool that generates and maintains agent-targeted documentation wikis for codebases and personal knowledge bases. Rather than just extracting facts, it runs a full DeepAgent with filesystem tools, git access, and built-in data connectors to produce structured wiki output in Google's Open Knowledge Format (OKF) v0.1 — wikis designed to be read by both humans and future AI agents.

---

## Architecture

OpenWiki is a single-agent CLI with a layered architecture. The agent is the core, surrounded by infrastructure for model instantiation, tool provision, prompt engineering, content validation, and state management.

### The Agent Core (`src/agent/index.ts`)

The agent is created via `createDeepAgent` from LangChain's `deepagents` package. Each run:

1. **Model factory** (`createModel`): Instantiates the right LangChain chat model based on the configured provider — `ChatOpenAI`, `ChatAnthropic`, `ChatGoogle`, `ChatBedrockConverse`, `ChatOpenRouter`, or a custom ChatGPT-OAuth Codex client. The Gemini Enterprise path auto-routes model IDs across three API surfaces (native Gemini, Anthropic-on-Vertex, OpenAI-compatible MaaS) using `vertex-surface.ts`.

2. **Checkpointer**: SQLite-backed for interactive chat (persistent conversation threads), in-memory for init/update runs (stateless — each run starts fresh). This is a deliberate trade-off: checkpoint bloat is avoided at the cost of no intra-run incremental state.

3. **Backend**: A `CompositeBackend` wrapping two virtual filesystems — `OpenWikiLocalShellBackend` (restricts writes to `openwiki/` or `~/.openwiki/wiki` during init/update) and a read-only `/skills/` mount.

4. **Middleware**: `OpenWikiIndexMiddleware` wraps every tool call for front matter validation, then after the agent finishes, deterministically rebuilds all `index.md` files from parsed front matter metadata.

5. **System prompt** (`src/agent/prompt.ts`, ~500 lines): The most carefully engineered component. Parameterized by output mode (local-wiki vs repository) and command (chat/init/update), producing different filesystem roots, search boundaries, wiki paths, and instruction sets for each combination.

### Two Modes, Two Roots

- **Code mode** (`openwiki --init`, `openwiki --update`): Generates documentation in `<repo>/openwiki/`. Injects `<!-- OPENWIKI:START -->…<!-- OPENWIKI:END -->` blocks into root `AGENTS.md` and `CLAUDE.md`. Filesystem tools are rooted at the repository; writes are restricted to `openwiki/`.

- **Personal mode** (`openwiki personal --init`): Generates a personal knowledge wiki in `~/.openwiki/wiki/`. Filesystem tools are rooted at the wiki directory. Content comes from connector data sources.

### The Ingestion Pipeline

Personal mode ingestion (`openwiki ingest`) follows a two-phase pattern:

```
Source config → [Phase 1: Deterministic Pull] → raw JSON files
                 ↓
              [Phase 2: Agent Synthesis] → wiki pages
```

Phase 1 is pure TypeScript: connectors fetch data via APIs, handle OAuth, manage cursors and pagination, and write raw JSON to `~/.openwiki/connectors/<id>/raw/<run-id>/`. The agent never sees credentials.

Phase 2 launches an agent run with a prompt like "Read these raw files and update the wiki from this source's evidence." The agent reads the raw JSON, classifies and routes the content using connector-specific synthesis policies, and writes wiki pages.

This separation is the key security insight: credential-handling code is in auditable, deterministic TypeScript, not prompt-constrained LLM output.

### CLI Architecture (`src/cli.tsx`, 4,019 lines)

The terminal UI is built on Ink (React for terminals). It implements a full state machine with states: `idle`, `running`, `success`, `error`, `ingestion-running`, `ingestion-success`, `init-setup-saved`. The UI includes slash-command menus (`/model`, `/provider`, `/api-key`), streaming tool-call display with grouped and variant-randomized labels ("Reading 3 files" vs "Taking a look at 3 files"), credential masked input, and a chat history view.

---

## Key Techniques

### Content-Addressed No-Op Detection (`src/agent/utils.ts`)

Before an update run, the system takes a SHA-256 hash of all wiki files (excluding `.last-update.json`). After the run, it hashes again. If identical, no metadata is written. Additionally, `getUpdateNoopStatus` compares the current git HEAD with the last recorded update's git HEAD — if the commit hash hasn't changed and the worktree is clean, the entire agent run is skipped. This prevents burning tokens on runs that would produce zero changes.

### Front Matter Validation Middleware (`src/agent/frontmatter-validator.ts`)

On every `write_file` or `edit_file` tool call targeting a wiki markdown file, the middleware reads the persisted file, validates YAML front matter (requires `type` field, validates `title`/`description`/`resource`/`timestamp` as non-empty strings, `tags` as string array), and injects a warning into the tool result message if invalid. The agent sees: `WARNING: YAML front matter was NOT formatted properly in /quickstart.md. [missing_type] Required field 'type' is missing. You MUST correct this file's YAML front matter before continuing.` This is a feedback loop at the tool level — deterministic enforcement, not prompt-level pleading.

### Deterministic Index Generation (`src/agent/index-middleware.ts`)

After the agent completes, the `afterAgent` hook collects all wiki directories, reads every markdown file in each, parses their YAML front matter for `title` and `description`, and rebuilds each directory's `index.md` with sorted file/directory listings. The root index gets `okf_version: "0.1"` front matter. This prevents the agent from producing malformed or inconsistent indexes, and ensures index files always reflect the actual file tree.

### Per-Connector Synthesis Policies (`src/ingestion.ts`)

Each connector has a hand-written synthesis policy paragraph injected into the agent's user prompt. For example, Gmail gets a 13-label classification taxonomy (action_required, scheduled_commitment, decision_or_approval, …, noise) plus priority and durability assignments; Hacker News items default to "watchlist" unless corroborated; Slack messages route to commitments with Owner tags. This is essentially curriculum design — teaching the agent source-specific editorial judgment rather than relying on generic "synthesize this" prompts.

### Gemini 3.x Thought-Signature Workaround (`src/agent/index.ts`)

Gemini 3.x requires `thoughtSignature` on function-call parts for multi-turn tool calling. LangChain's streaming aggregator unconditionally re-emits messages as v1 content blocks, which drop that provider-specific signature. The fix: `disableStreaming: true` and `outputVersion: "v0"` for both Gemini AI Studio and Gemini Enterprise, routing through `invoke()`/`generate` which preserves raw content parts. This is the kind of provider-specific edge case that only shows up in real multi-turn agent use.

### Wiki-First Answering Strategy

The system prompt instructs the agent to inspect the synthesized wiki before raw connector data. Raw data is consulted only when the wiki is "missing the needed detail, clearly stale, ambiguous, contradicted, or the user explicitly asks for source-level evidence." This makes the wiki a compounding artifact — each update builds on previous synthesis rather than re-deriving everything from raw data.

### Canonical Cross-Source Files (Personal Mode)

Rather than one source page per connector, the system prompt specifies canonical cross-source files:
- `quickstart.md` — navigation and current status
- `open-questions.md` — Active/Answered/Stale sections for memory/wiki uncertainty
- `themes.md` — compact recurring signal index (prefer table rows, cap prose at 1-2 sentences)
- `commitments.md` — work tasks with Owner tags (me, team, other:name, unknown)
- `personal-logistics.md` — non-work errands, appointments, travel
- `sources/<connector>.md` — compact evidence index, not primary synthesis

This is a deliberate information architecture: connectors feed into topics, not the reverse. The structure encodes the belief that cross-source synthesis is more valuable than per-source filing.

---

## Design Decisions

### Prompt Engineering as the Control Surface

The system prompt is ~500 lines. The tools are standard filesystem operations (read, write, edit, ls, glob, grep, execute). There's no specialized knowledge-graph tool, no RAG pipeline, no embedding-based retrieval. Compared to [[Wuphf — Karpathy-Style Agent Wiki]] (BM25 + SQLite retrieval) or [[QMD]] (BM25 + vector + LLM re-ranking), OpenWiki is much simpler in its tool surface and much heavier in its prompt. The trade: simpler infrastructure, but prompt quality is brittle — a model upgrade that changes instruction-following behavior could break the pipeline.

### Built-In Connectors Only, No Plugin Marketplace

New connectors require a PR to the OSS repo. This optimizes for security (all credential-handling code is reviewed) over extensibility. Compare with [[DeerFlow]]'s MCP/skills plugin system or [[Mirage (VFS)]]'s mount-anything approach. The trade is defensible for a tool that handles OAuth tokens and API credentials, but it limits the connector ecosystem.

### Single-Agent Over Multi-Agent Swarm

OpenWiki uses one DeepAgent with optional subagent delegation for parallel read-only research (1-2 subagents typical). Subagents inspect and summarize; the main agent synthesizes and writes. This is simpler than the multi-agent patterns in [[Cloudflare Security Audit Skill]] (6 specialized agents + coordinator) or [[Orchestrating AI Code Review at Scale]] (7 agents + judge), but offers less parallelism and no adversarial review of wiki content. The "loop-until-dry" and "adversarial verify" patterns from the Workflow tool are absent.

### No Embeddings, No Vector Search

Content discovery is done through grep, glob, and targeted file reads. There's no semantic search over the wiki. This keeps the architecture simple but means the agent can't find "conceptually similar" content — it can only find textually matching content. For a personal knowledge wiki that might grow to hundreds of pages, this is a real gap compared to tools that use embeddings for retrieval.

### Stateless Init/Update, Stateful Chat

Init and update runs use in-memory checkpoints — each run is self-contained. Chat mode uses SQLite-persisted checkpoints for conversation continuity. The trade: init/update can't build on partial progress within a run, but checkpoint storage doesn't accumulate across runs.

### OKF Compliance as Enforced, Not Suggested

The front matter validator is middleware, not a prompt instruction. If the agent writes a file with bad front matter, it gets an error in the tool result. This is the [[Guardrails and Feedback Loops]] pattern: deterministic enforcement beats prompt-level pleading. Most agent wiki tools suggest structure; OpenWiki enforces it at the tool level.

---

## Comparison Notes

**vs. [[DeepWiki]]**: DeepWiki is read-only, cloud-hosted, and gives instant answers from a pre-built code graph. OpenWiki is local, write-capable, and produces persistent markdown wikis that improve with updates. DeepWiki is a lookup tool; OpenWiki is a maintenance tool. They're complementary: DeepWiki for quick comprehension, OpenWiki for durable documentation.

**vs. [[Wuphf — Karpathy-Style Agent Wiki]]**: Both produce markdown wikis for agents, but Wuphf has draft-to-promote gates, BM25 retrieval, and daily lint crons. OpenWiki has no retrieval layer, no promotion flow — the agent writes directly to the wiki. Wuphf is optimized for team coordination; OpenWiki is optimized for initial generation and incremental update.

**vs. [[graphify]]**: graphify produces multimodal knowledge graphs from codebases. OpenWiki produces structured markdown wikis. graphify is about structural visualization; OpenWiki is about narrative documentation.

**vs. [[Understand-Anything]]**: Both are Claude Code-adjacent codebase documentation tools, but Understand-Anything uses tree-sitter + LLM for knowledge graphs with incremental git-hook updates. OpenWiki uses a full agent loop with filesystem tools and prompt-driven synthesis.

**vs. [[docmason]]**: docmason targets office documents with citations and source tracing. OpenWiki targets codebases and personal data sources with OKF-structured output. Different domain, similar concept.

**vs. [[Kiso]]**: Kiso is an OKF-to-static-site publisher. OpenWiki is an OKF producer. They're complementary: OpenWiki generates the OKF, Kiso could publish it.

---

## What's Missing

**No adversarial review**: The agent writes wiki pages with no second agent checking them. The front matter validator catches format errors but not factual errors. Compare with the Workflow tool's "adversarial verify" pattern (N independent skeptics per finding, kill if majority refute). A wiki that grows without adversarial review accumulates confident wrongness.

**No staleness tracking**: Unlike [[Wuphf — Karpathy-Style Agent Wiki]]'s daily lint cron, OpenWiki has no mechanism to detect or flag stale content. The "update" command can surgically edit pages, but there's no systematic staleness audit. The `log.md` file records ingests but isn't used for freshness checks.

**No retrieval**: Agents reading the wiki must grep and read files linearly. There's no semantic search, no BM25, no embeddings. For a personal wiki that might accumulate hundreds of pages, this becomes a navigation problem.

**No multi-agent debate**: The single-agent architecture means there's no mechanism for agents to challenge each other's wiki entries. The [[Immaculate Knowledge Graph]]'s "co-occurrence is not truth" problem applies equally here.

---

*Tags: #tool #agent-wiki #documentation-generation #langchain #deepagents #okf #cli #project*

*Sources: [[raw/openwiki-langchain]]*
*Last updated: 2026-07-18*
