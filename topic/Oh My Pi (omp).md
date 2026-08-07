# Oh My Pi (omp)

An open-source terminal-based coding agent harness that forks Mario Zechner's minimalist Pi and extends it into the most feature-complete agent surface available — 32 built-in tools, 40+ LLM providers, in-process native operations (~55K lines of Rust), and a thesis that the harness matters more than the model. TypeScript on Bun, Rust N-API addon for the hot path, MIT licensed.

Complements [[Pi Subagents]], Nico Bailon's multi-agent delegation extension for standard Pi. Where omp extends Pi by going deeper (more tools, native performance, dual memory), pi-subagents extends Pi by going wider (child agents, chain execution, workflow scripts, RPC protocol). The two could theoretically compose — omp's 32-tool surface with pi-subagents' delegation system — though compatibility depends on whether omp maintains Pi's extension API contract.

---

## Architecture

omp is a **two-layer monorepo**: TypeScript agent logic (14 npm packages, Bun runtime) + Rust native addon (6 crates, N-API `cdylib`). The Rust layer runs in-process — no fork/exec for search, shell, AST, syntax highlighting, or text processing.

**Key packages**:
- `packages/coding-agent/` (345K LoC) — main CLI, SDK, session management, tool implementations, slash commands, extension loading
- `packages/agent/` (13K LoC) — agent runtime loop, compaction strategies, tool execution, tokenizer, telemetry
- `packages/ai/` (88K LoC) — multi-provider LLM client with per-provider dialect adapters, schema coercion, OAuth flows, error classification
- `packages/tui/` (23K LoC) — differential terminal rendering, tool cards, option pickers
- `packages/mnemopi/` (19K LoC) — local SQLite+vector memory engine with beam store, Weibull MMR scoring, polyphonic recall
- `packages/hashline/` (5.7K LoC) — content-hash-anchored patch language: tokenizer → parser state machine → patcher

**Key crates** (`crates/pi-natives/src/`):
- `grep.rs` — ripgrep-backed search with PCRE2/regex engines, parallel file search via rayon
- `shell.rs` — vendored brush-shell for embedded bash with persistent sessions and output minimizer
- `summary.rs` — tree-sitter structural code summaries across 50+ languages
- `keys.rs` — Kitty keyboard protocol + xterm fallback, PHF perfect-hash key lookup

**Session storage** (`packages/coding-agent/src/session/`): Append-only JSONL files with tree-shaped branching (parentId/leafId). Content-addressed blob store (SHA-256). Compaction entries are first-class session entries, not plain messages.

**Four entry points**: Interactive TUI (`omp`), one-shot (`omp -p`), RPC over stdio (`omp --mode rpc`), and ACP for editor integration (`omp acp`). The RPC mode is Pi-compatible, so [[Pi-msg — XMPP Bridge for Pi Coding Agent]] could bridge omp to XMPP with minimal changes — only the companion extension would need porting.

## Key Techniques

### Hashline: content-hash-anchored editing

Instead of line numbers or string matching, models reference lines by content hash: `[file.ts#abc123]` then `SWAP 42:56` or `DEL 87:92`. Stale anchors cause rejection before corruption — the model can't silently edit the wrong lines. The parser (`packages/hashline/src/parser.ts`) is a token-driven state machine that validates range ordering and rejects bare literal values. If an anchor doesn't match, the applier tries fuzzy matching via `recovery.ts` before rejecting. Grok 4 Fast spends 61% fewer output tokens on the same work with hashline vs. str_replace.

### Time-Traveling Stream Rules (TTSR)

A regex match on the model's output stream aborts mid-token, injects a rule as a system reminder, and retries from the same point. Rules stay dormant until the model goes off-script — no context tax per turn. Injections survive compaction. This is a fundamentally different approach from prompt-based guardrails: it catches mistakes at runtime, not at prompt-construction time.

### Dual memory architecture

**Hindsight** — a markdown-based pipeline: Phase 1 extracts durable signal from session history (LLM read → raw memory block); Phase 2 consolidates across sessions into `MEMORY.md`, `memory_summary.md`, and `skills/` playbooks. The summary is injected at session start.

**Mnemopi** — a local SQLite database with vector embeddings (768d or 1024d). The beam store (`packages/mnemopi/src/core/beam/`) uses Weibull MMR scoring for recall diversity — a probabilistic alternative to the standard max-marginal-relevance formula that models human-like forgetting curves. Polyphonic recall (`polyphonic-recall.ts`) runs four parallel retrieval voices (vector, graph, fact, temporal) and fuses results with reciprocal rank fusion.

### In-process native operations

All performance-critical operations run as Rust code linked into the Bun process via N-API. ripgrep, bash (brush-shell), find, ripgrep, glob, AST tools — no fork/exec, no missing-binary failures. On Windows this means no WSL bridge. The shell module (`crates/pi-natives/src/shell.rs`) wraps a vendored brush-shell interpreter with persistent sessions, timeout/abort, and an output minimizer that reduces verbose command output before it reaches the LLM — an underappreciated technique that saves tokens on nearly every `bash` call.

### Compaction with multiple recovery strategies

The compaction system (`packages/agent/src/compaction/`) has six trigger paths, each with different rules: overflow recovery tries model promotion first, then compaction; incomplete-output recovery allows handoff strategy (where overflow doesn't); threshold maintenance runs post-turn and mid-turn. The compaction entry (`CompactionEntry`) stores `firstKeptEntryId` as the boundary, so context reconstruction knows exactly where the summary ends and live context begins.

### Internal URL schemes

Twelve internal schemes (`pr://`, `issue://`, `agent://`, `skill://`, `rule://`, `memory://`, `conflict://`, etc.) resolve transparently inside every FS-shaped tool. `read pr://1428` returns structured diff data. `search` walks a diff like a directory. This collapses many API-specific tool calls into the single `read`/`search` surface — a clever inversion of the tool-proliferation pattern seen in most agent frameworks.

## Design Decisions

**The harness, not the model, is the product.** The README quantifies this: Grok Code Fast 1 goes from 6.7% to 68.3% pass rate purely from hashline replacing a broken edit format. Gemini 3 Flash gains +5pp over Google's own best attempt at str_replace. MiniMax pass rate more than doubles. The same weights, the same prompt — only the harness changed.

**In-process over shell-out.** Most coding agents shell out to `rg`, `grep`, `find`, and `bash`. omp links the real implementations into the process. Trade-off: ~55K lines of Rust to maintain, platform-specific builds required — but eliminates both the fork/exec tax and the "binary not installed" failure mode.

**Batteries-included over minimalism.** Pi's original philosophy was minimalism. omp adds sessions, subagents, slash commands, extensions, memory, browser automation, collaboration, ACP, and 30+ more tools. This is a deliberate fork in philosophy: the project believes a coding agent should ship complete, not force users to assemble their own toolchain. The cost is complexity — 32 tools is a lot for any model to navigate, mitigated by the `search_tool_bm25` discovery mechanism.

**TypeScript-as-platform, not TypeScript-as-glue.** Unlike agents where TypeScript is a thin scripting layer over Python or Rust cores, omp's agent logic IS TypeScript — the Rust layer is strictly for performance-critical operations. This means the Rust footprint stays focused (~55K lines vs. ~345K TypeScript) and the extension API is native TypeScript, not a bridge.

**Configuration inheritance as adoption strategy.** omp reads 8 competitor config formats natively — Cursor MDC, Cline .clinerules, Codex AGENTS.md, Copilot applyTo, and others. No migration script. This is generous for adoption but creates a maintenance burden as upstream formats evolve.

## Comparison Notes

**Vs. Claude Code**: Claude Code uses str_replace; omp uses hashline. Claude shells out; omp runs in-process. Both have subagents, but omp returns typed objects (schema-validated) while Claude Code returns prose. omp's advisor pattern runs a second model reading every turn; Claude Code doesn't have a built-in equivalent. See [[Steering Claude Code]], [[Tuning Claude Code Into a Better Engineering Partner]].

**Vs. original Pi**: Pi was a minimal agent surface. omp adds sessions, subagents, slash commands, extensions, memory, compaction, browser, collaboration, ACP, and 30+ more tools. See [[Pi Coding Agent]] for the original philosophy and the tension with platform ambitions.

**Vs. MiMo Code**: Both target long-horizon tasks with independent writer subagents and checkpointing. MiMo uses code-based orchestration (Dynamic Workflow); omp uses prompt-based orchestration with typed tool results. MiMo splits a single task across time; omp splits across parallel workers. See [[MiMo Code]].

**Vs. Codex**: Codex uses apply_patch; omp uses hashline + ast_edit. omp explicitly detects and rejects apply_patch contamination in the hashline parser. omp's isolation is filesystem-level (pi-iso crate, APFS clones/reflinks) rather than container-level.

**Vs. OpenMonoAgent / CodeAlta**: Both are terminal coding agents in C#/.NET with local LLM support. omp is TypeScript+Rust with a much larger provider surface (40+ vs. <10) and more built-in tools (32 vs. ~20). See [[OpenMono Agent]], [[CodeAlta]].

**Vs. Aider**: Aider uses unified diffs; omp uses hashline anchors + ast_edit structural rewrites. Aider's edit format has known failure modes (whitespace, context matching) that hashline was designed to eliminate. See [[Components of a Coding Agent]].

---

*Source: [[raw/oh-my-pi]]*
*Last updated: 2026-07-11*
