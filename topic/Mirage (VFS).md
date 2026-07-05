# Mirage (VFS)

A unified virtual filesystem for AI agents that mounts 27+ heterogeneous services (S3, Slack, Gmail, GitHub, Linear, Notion, Postgres, MongoDB, Redis, SSH, and more) behind a single POSIX-like tree. Agents interact with every backend through familiar bash commands — `cat`, `grep`, `ls`, `cp`, `find` — rather than learning per-service SDKs or MCP tools. Built by Strukto.AI, ~147K lines Python with a TypeScript port, Apache 2.0.

---

## Architecture

Mirage is a six-layer plugin architecture: **Resource → Core Ops → High-Level Ops → Mount Registry → Workspace → Agent Bridge**.

### Layers

1. **Resource** (`resource/`) — Each backend is a `BaseResource` subclass that registers commands and ops, and provides a typed `accessor` for low-level API calls. 27 resources in the registry from RAM/Disk to Slack/Linear/Postgres.

2. **Core Ops** (`core/`) — Per-resource implementations of filesystem primitives: `stat`, `read`, `write_bytes`, `readdir`, `mkdir`, `unlink`, `find`, `du`, `stream`. Each resource has its own directory with per-operation modules.

3. **High-Level Ops** (`ops/ops.py`) — Path-to-mount resolution via longest-prefix match, op recording for observability, async per-path locking, write invalidation.

4. **Mount Registry** (`workspace/mount/`) — Maps `/slack/`, `/s3/`, etc. to Mount objects. Each Mount has a resource, a mode (READ/WRITE/EXEC), per-mount command/op tables, and revision pins for snapshot replay.

5. **Workspace** (`workspace/workspace.py`) — The top-level API. Composes all layers, manages two-tier cache (RAM/Redis for both file bytes and index metadata), sessions with isolation, snapshot/replay with drift detection, and the shell executor.

6. **Agent Bridges** (`agents/`) — Thin adapters for OpenAI Agents SDK, LangChain/DeepAgents, Pydantic AI, CAMEL, and OpenHands. Each exposes the workspace as a sandbox or tool layer.

### Command Execution Flow

When `ws.execute("grep alert /slack/general/*.json | wc -l")` runs:

1. Tree-sitter-bash parses the command into an AST
2. The executor walks the AST, classifying tokens as paths or text
3. Each path's mount prefix is resolved by longest-prefix match
4. Commands dispatch to per-mount registered handlers
5. A command resolution hierarchy applies: `(name, filetype)` → `(name, None)` → `general[name]`
6. Output streams flow through async generators that record bytes consumed for observability

### Cross-Mount Operations

The executor detects when paths span mounts (e.g., `cp /s3/data.csv /disk/local.csv`) and routes through a cross-mount handler. For traversal commands (`find`, `tree`, `du`) and recursive `grep`, it fans out across all descendant mounts, adjusting depth flags and deduplicating shadowed paths.

### Cache Architecture

Two independent layers, each with RAM (default) and Redis backends:

- **File cache**: Object bytes. `cat /s3/data.csv` hits S3 on first access; subsequent reads serve from cache. Invalidated on write. Configurable size limit.
- **Index cache**: Directory listings. `ls /s3/data/` hits S3 LIST on first access; serves from cache until TTL (default 600s). Invalidated on any write to the directory.

Consistency policy: LAZY trusts cache; ALWAYS stats remote on every read and compares fingerprints.

---

## Key Techniques

### Tree-Sitter Shell Parsing

Uses `tree-sitter-bash` for proper bash AST construction, not regex or hand-rolled parsing. Handles pipes, redirects, subshells, variable expansion, compound commands. Syntax errors detected structurally and reported with position information (`shell/parse.py`).

### Monkey-Patched Builtins for Transparent VFS

`Workspace.__enter__` replaces `builtins.open` and `sys.modules["os"]` with Mirage-aware versions. Any Python code inside `with ws:` that calls `open()` or `os.listdir()` transparently hits the virtual filesystem. This is how the LangChain adapter works — DeepAgents' file operations go through standard Python I/O but hit Mirage mounts.

### Per-Resource Command Overrides

Commands resolve at three specificity levels: `(cat, .parquet)` renders rows as JSON instead of raw bytes; `(cat, None)` is the resource-specific default; `general["cat"]` is the fallback. Same command name, different behavior depending on filetype — no new vocabulary for the agent.

### Session Forking for Ephemeral State

When `execute()` gets a `cwd` or `env` override, it forks the session rather than mutating it. `cd` and `export` inside the command don't leak back — mirrors bash subshell semantics.

### Provision Mode (Dry-Run)

`ws.execute(command, provision=True)` returns a `ProvisionResult` listing exactly which paths would be read/written, without executing. Safety mechanism for agents before destructive operations.

### Snapshot with Content Drift Detection

`ws.snapshot()` serializes mount configs, sessions, history, cache bytes, and per-path content fingerprints. `Workspace.load()` reconstructs and runs drift detection: compares saved fingerprints against live remote state. STRICT mode raises on mismatch; OFF mode evicts stale cache. Resources with stable revision support (S3 with VersionId) can pin to specific versions for byte-identical replay.

---

## Design Decisions

**Bash as the universal interface.** The core bet: LLMs are most fluent in bash and filesystem semantics from training data. Teaching agents per-service tool names (the MCP approach) creates a vocabulary scaling problem. Making everything a filesystem means the agent uses the same 10 commands everywhere. Trade-off: the filesystem metaphor gets lossy for structured APIs — Slack channels as date-named JSON files is clever but hides thread relationships and reaction metadata.

**Virtual-first, FUSE-optional.** Operates in-process by default; FUSE is an optional optimization. Avoids platform-dependence and root requirements for the common case. Trade-off: pure-Python async I/O is slower than kernel-level.

**Cache-heavy by default.** Remote resources aggressively cached. First read hits network; subsequent reads (within TTL) are local. Right for agent workloads where the same file gets read multiple times in a pipeline. Trade-off: LAZY consistency can serve stale data.

**Comprehensive mount guardrails.** Can't `rm`/`mv`/`mkdir` mount roots. Session-level `allowed_mounts` restricts prefix access. Mode enforcement (READ/WRITE/EXEC) per mount. The `_check_mount_root_guard_raw` function is a 50-line case analysis — defensive engineering that pays off when agents run unsupervised.

**Snapshots capture what the agent saw, not full state.** Fingerprints and cache bytes, not complete remote data. Replay fidelity depends on remote data still existing. Pragmatic — you can't snapshot all of S3.

---

## Comparison Notes

**vs. MCP**: MCP adds per-service tools agents call by name; Mirage makes everything a filesystem agents navigate with bash. Mirage argues LLMs' training-data fluency in bash beats the tool-name vocabulary problem. MCP argues filesystem metaphors are lossy for structured APIs. These aren't mutually exclusive — an MCP server could expose a Mirage workspace as a tool.

**vs. Locker**: Both provide virtual filesystems over storage backends. Locker targets human file management; Mirage targets agent tooling with framework adapters, session isolation, snapshot/replay, and command overrides.

**vs. Individual FUSE filesystems** (s3fs, gcsfuse): Each mounts one backend. Mirage mounts many backends behind one unified tree with shared caching and agent integration.

**vs. Direct SDK usage**: Most agent harnesses give agents per-service SDKs. Mirage argues this doesn't scale — every SDK has different semantics and auth models. The filesystem abstraction normalizes these into one interface.

---

## Related

- [[Locker]] — similar virtual filesystem over storage backends, but human-focused rather than agent-tooling
- [[agentsh]] — FUSE-based security gateway; Mirage takes the opposite approach (virtual-first, FUSE-optional)
- [[StrongDM Factory Techniques]] — the "Filesystem-as-memory" pattern from the dark factory floor
- [[Components of a Coding Agent]] — the harness matters more than the model; Mirage is harness infrastructure
- [[Agent-Native Architectures (Every)]] — files as universal interface principle
- [[MCP Is Dead; Long Live MCP]] — the CLI vs MCP debate that Mirage enters on the CLI side
- [[Smart Models Dumb Pipes]] — similar philosophy: the model makes judgments, the infrastructure executes
- [[Scaling Long-Running Agents]] — context management is the real challenge; Mirage's cache reduces network round-trips
- [[Planning With Files]] — filesystem as the persistence primitive

Tags: #tool #project #agents #filesystem #infrastructure

---
*Sources: [[summary/mirage]]*
*Last updated: 2026-05-18*
