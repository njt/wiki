# CodeGraph — A Pre-Built Code Knowledge Graph Under the Agent

CodeGraph (Colby McHenry, MIT, v1.6.2) is a local semantic code-indexing layer for AI coding agents: it parses a repo into a SQLite knowledge graph of symbols, edges, and framework routes, exposes exactly one MCP tool (`codegraph_explore`) that returns verbatim source plus call paths and blast radius in a single call, and keeps the graph live with a debounced native file watcher. It matters because it attacks the biggest tax on agentic coding — structure discovery via grep/glob/Read — with a precomputed index, and it publishes honest benchmarks including the metric where it *loses*.

---

## Architecture

Four layers, roughly 140k lines total:

1. **Extraction kernel** (`codegraph-kernel/src/`, ~23k lines Rust, napi-rs). Tree-sitter grammars compiled into the native binary; per-language walker modules (`tsjs/`, `python.rs`, `swift.rs`, …) replace only the parse+extract step inside the existing TS `ParseWorkerPool`, behind the `ExtractionResult` contract — one boundary crossing per file, output as five flat typed tables (`buffers.rs`) mirrored by `src/extraction/kernel/layout.ts`. Graphs must be byte-for-byte identical to the portable TypeScript engine (`src/extraction/tree-sitter.ts`, alone ~10k lines); a parity script gates this, and files with syntax errors or on platforms without a prebuilt binary fall back per-file to WASM — same graph either way. A stack-pointer red-zone guard (`stack.rs`) makes deep recursive walkers bail gracefully instead of segfaulting.

2. **Resolution + synthesis** (`src/resolution/`, ~94k lines). The bulk of the project. Beyond name matching (`name-matcher.ts`, 33k lines) there are ~40 "synthesizers" that add edges static parsing can't see: framework routers (Django, Express, Next.js, Expo Router, Angular, …), Swift↔ObjC `@objc` bridging, React Native legacy bridge/TurboModules/Fabric, Expo Modules DSL, Tauri `invoke` flows. Every synthesized edge carries provenance metadata (`resolvedBy: 'framework'`, `provenance: 'heuristic'`).

3. **Storage and freshness** (`src/db/`, `src/sync/`). Nodes/edges in SQLite with FTS5, WAL. The watcher (`src/sync/watcher.ts`) uses recursive `fs.watch` on macOS/Windows (O(1) descriptors via one FSEvents/ReadDirectoryChangesW stream — explicitly fixing a prior per-fd design that exhausted `kern.maxfiles` system-wide) and per-directory inotify on Linux (O(directories)). 2s debounce, incremental sync, 30s lock-retry cooldown before degrading auto-sync.

4. **MCP serving** (`src/mcp/`). The clever part is `daemon.ts`: one detached daemon per project root behind a Unix socket, with a thin *proxy* process as what the agent host actually spawns (proxy carries the PPID watchdog, so a killed host reaps cleanly; daemon survives clients and lingers 300s idle). All sessions share one watcher, one SQLite writer, one tree-sitter warm-up.

## Key techniques

- **One tool, not a menu.** The default MCP surface is `codegraph_explore` alone (others exist behind `CODEGRAPH_MCP_TOOLS`). Measured agent behavior showed one strong tool steers better than narrow ones: fewer mis-picks, less context. The tool's instructions, delivered in the MCP `initialize` response (`server-instructions.ts`), tell the agent to treat results as *already read*, not re-verify with grep — the index and the prompt are engineered as one system.
- **Honest self-benchmarking.** The README's 88%-fewer-tools/53%-faster numbers were re-measured after discovering the control arm was contaminated — without blocking, 26 of 28 control runs found the `codegraph` CLI on PATH and used it. They now block the CLI in both arms via a `PreToolUse` hook and report 0/28 contamination. They also publish the loss: ~80% more residual context occupancy at session end (67k vs 18k tokens on VS Code), because one dense payload lingers where many small reads get evicted.
- **Adaptive sizing.** Worker pools and caches sized from cgroup-aware core counts, measured available RAM, and measured per-project resolution cost — a 2-core/6GB VPS indexes the Linux kernel (2M symbols) in ~12 minutes where RAM-first designs die before 1%.
- **Provenance on every synthesized edge**, so the agent can distinguish measured structure from heuristic inference.
- **Staleness honesty.** During the 2s debounce window, tool responses that would reference a pending file get a `⚠️` banner telling the agent to Read it directly — the system treats "index is briefly behind" as a first-class answer state rather than silently returning stale code.

## Design decisions

The core bet is **precomputation over per-session discovery**: pay once at `init` (Swift compiler repo, 27k files, ~100s) and amortize across every session, rather than having each agent run re-derive structure. This trades disk, RAM, and sync complexity for tokens and latency — and only pays off because the watcher keeps staleness near zero; a stale index would be worse than no index, hence the daemon/watcher/banner machinery.

The counter-bet is context *footprint* over context *churn*: CodeGraph knowingly leaves a heavier resident footprint. For long sessions in small windows that's a real cost, and the README says so rather than burying it.

Weaknesses worth naming: the Rust kernel only covers 20 of the ~35 advertised languages (the rest ride the slow portable engine); the 33k-line name matcher is where correctness lives and dies for dynamic languages; and the single-tool philosophy means the guidance quality of `server-instructions.ts` is load-bearing — a model that ignores prompt steering gets a big payload it didn't ask for.

## Comparison notes

- [[Your Knowledge Graph Is Making Your Agent Dumber]] found Graphify's code knowledge graph returned 5%-relevant noise vs ripgrep's 100% in 0.01s — CodeGraph is the counter-argument: the failure mode isn't graphs of code per se, it's *semantic-embedding* graphs with no call edges. CodeGraph's graph is deterministic structural extraction with verbatim source attached, which is why its benchmark shows file reads dropping to zero rather than rising.
- [[How Claude Code Works in Large Codebases]] describes the grep/Read discovery loop CodeGraph is designed to replace — the "up to 43 tool calls to answer one question" control arm is exactly that pattern, quantified.
- [[What Is a Context Graph]] frames context-as-retrieved-artifact; CodeGraph is the strongest concrete instance: the index is precomputed context, and the MCP instructions exist to make the agent *consume* it instead of rebuilding it.
- Unlike [[Three Kinds of Agentic Search]]' taxonomy, CodeGraph occupies the "structured index" slot and argues it should *crowd out* the grep slot — a claim the two-arm benchmarks support for architecture questions but not, by its own admission, for token residency in long sessions.

#tool #project #agents #database #mcp #static-analysis

---
*Sources: [[raw/codegraph]], [[summary/codegraph]]*
*Last updated: 2026-11-06*
