---
url: https://github.com/strukto-ai/mirage
title: "Mirage: A Unified Virtual File System for AI Agents"
author: "Zecheng Zhang / Strukto.AI"
date_fetched: 2026-05-18
date_published: 2026-05-01
---

# Mirage — Full Repo Analysis

## Overview

Mirage is a unified virtual filesystem for AI agents, built in Python (~147K lines) with a TypeScript port. It mounts heterogeneous services (S3, Slack, Gmail, GitHub, Linear, Notion, Postgres, MongoDB, Redis, SSH, Google Drive, Discord, Telegram, Trello, Langfuse, and more) behind a single POSIX-like filesystem tree. Agents interact with all backends through familiar bash commands — `cat`, `grep`, `ls`, `find`, `cp`, `wc` — rather than learning per-service SDKs or MCP tools.

The project is from Strukto.AI, founded by Zecheng Zhang. Licensed Apache 2.0. Version 0.0.2a0 at time of analysis.

## Project Structure

```
python/mirage/
  accessor/         — Typed accessor objects per resource (SlackAccessor, S3Accessor, etc.)
  agents/           — Framework adapters: OpenAI Agents SDK, LangChain/DeepAgents, Pydantic AI, CAMEL, OpenHands
  bridge/           — sync/async interop (run_async_from_sync, thread management)
  cache/            — Two-layer cache: file cache (RAM/Redis) and index cache (RAM/Redis)
  cli/              — Typer-based CLI: daemon, execute, provision, session management, workspace ops
  commands/         — Command registration system with spec-based parsing and per-resource builtins
  core/             — Per-resource backend implementations: read, write, stat, readdir, stream, etc.
  fuse/             — Optional FUSE mount for native kernel-level filesystem
  io/               — Async stream abstractions, ByteSource, IOResult
  observe/          — Op recording for audit trails and observability
  ops/              — Higher-level ops layer: stat, read, write, mkdir, etc. with caching and recording
  provision/        — Dry-run provisioning that computes what a command would do without executing
  resource/         — Resource class definitions wrapping core backends with command/op registration
  runtime/          — Runtime utilities (assert_mount_allowed, etc.)
  server/           — FastAPI server with auth, routers for remote workspace access
  shell/            — Tree-sitter bash parser, job table, call stack, barrier policy
  types.py          — Core types: FileStat, FileType, MountMode, ConsistencyPolicy, PathSpec
  utils/            — Misc utilities
  vfp/              — Virtual file protocol (schema definitions)
  workspace/        — Core workspace: mount registry, executor, session management, snapshots, history
```

## Architecture Deep Dive

### Layered Architecture

1. **Resource Layer** (`resource/`) — Each backend (S3, Slack, GitHub, etc.) is wrapped as a `BaseResource` subclass. The resource registers commands and ops, and provides an `accessor` object that core ops use for low-level API calls.

2. **Core Ops Layer** (`core/`) — Per-resource implementations of filesystem primitives: `stat`, `read`, `write_bytes`, `readdir`, `mkdir`, `unlink`, `rm`, `find`, `du`, `stream`. Each resource has its own directory under `core/` with per-operation modules.

3. **Ops Layer** (`ops/`) — Higher-level dispatch: resolves paths to mounts, records ops for observability, manages async locks per path, handles write invalidation.

4. **Mount Registry** (`workspace/mount/`) — Maps path prefixes to Mount objects. Uses longest-prefix matching. Each Mount holds a resource, a mode (READ/WRITE/EXEC), and per-mount command/op lookup tables.

5. **Workspace** (`workspace/workspace.py`, 776 lines) — The top-level API. Composes all layers: creates mounts from resources, initializes caches, sets up the shell executor, manages sessions, handles snapshots.

6. **Shell Executor** (`workspace/executor/command.py`, 754 lines) — Parses bash via tree-sitter, resolves commands against mounts, handles cross-mount operations, fan-out traversal, job control builtins, shell functions.

7. **Agent Bridges** (`agents/`) — Thin adapters that expose the workspace as a sandbox or tool layer for specific agent frameworks.

### Path Resolution

Paths are resolved by longest-prefix match against the mount registry. `/slack/general/2026-05-18.json` resolves to the Slack mount with relative path `/general/2026-05-18.json`. The default mount (cache-backed RAM) catches paths that don't match any registered prefix.

### Command Dispatch

When `ws.execute("grep alert /slack/general/*.json | wc -l")` runs:

1. Tree-sitter parses the command into an AST
2. The executor walks the AST, classifying tokens as paths or text
3. For each simple command, the path's mount prefix is resolved
4. If paths span multiple mounts, a cross-mount handler kicks in (for known cross-mount ops like `cp`)
5. The mount dispatches to its registered command handler
6. Output streams flow through the pipeline

### Cache Architecture

Two independent cache layers, each with RAM and Redis backends:

- **File cache** (`cache/file/`): Stores object bytes. First `cat /s3/data.csv` hits S3 GET; subsequent reads serve from cache. Invalidated on write. Configurable size limit with drain behavior.
- **Index cache** (`cache/index/`): Stores directory listings and metadata. First `ls /s3/data/` hits S3 LIST; subsequent listings serve from cache until TTL expires (default 600s). Invalidated on any write to the directory.

Consistency policy controls freshness: LAZY (default) trusts cache; ALWAYS stats the remote on every read and compares fingerprints.

### Snapshot & Replay

`ws.snapshot("state.tar")` serializes mount configs, sessions, history, finished jobs, cache bytes, and per-path content fingerprints. `Workspace.load("state.tar")` reconstructs the workspace and runs drift detection: for each recorded read, it compares the saved fingerprint against the current remote state. STRICT mode raises on mismatch; OFF mode evicts stale cache and serves current state.

Resources with stable revision support (S3 family with VersionId) can pin to specific versions during replay, guaranteeing byte-identical reads.

### Mount Guardrails

The executor has comprehensive safety checks baked into the command dispatch layer:
- Cannot `rm`, `mv`, `mkdir`, or `touch` mount roots
- Session-level `allowed_mounts` restricts which prefixes a session can access
- Mount modes (READ/WRITE/EXEC) enforced per-prefix
- Infrastructure mounts (cache, dev, observer) are always allowed

### Cross-Mount Fan-Out

For traversal commands (`find`, `tree`, `du`) and recursive `grep`/`ls -R`, the executor detects descendant mounts under the target path and fans out execution across all relevant mounts. It adjusts `-maxdepth`/`-mindepth` flags for each child mount and filters parent-mount output to avoid duplicates where child mounts shadow parent paths.

## Key Techniques

### Tree-Sitter Shell Parsing

Instead of regex-based command parsing, Mirage uses tree-sitter-bash for full bash AST construction. This gives it proper handling of pipes, redirects, subshells, variable expansion, and compound commands. Syntax errors are detected structurally (not heuristically) and reported with position information.

### Monkey-Patching Builtins for Transparent VFS

`Workspace.__enter__` replaces `builtins.open` and `sys.modules["os"]` with Mirage-aware versions. This means any Python code running inside a `with ws:` block that calls `open()` or `os.listdir()` transparently hits the virtual filesystem. This is how the LangChain adapter works — DeepAgents' file operations go through standard Python I/O but hit Mirage mounts.

### Per-Resource Command Overrides

Commands are registered at multiple specificity levels:
1. `(command_name, filetype_extension)` — e.g., `cat` on `.parquet` files renders rows as JSON
2. `(command_name, None)` — resource-specific default for the command
3. `general[command_name]` — fallback that works on any resource

This lets `cat /s3/events.parquet` produce structured JSON while `cat /s3/README.md` produces plain text, using the same command name.

### Session Forking for Ephemeral State

When `execute()` receives a `cwd` or `env` override, it forks the session rather than mutating it. The fork is ephemeral — `cd` and `export` inside the command don't leak back to the persistent session. This mirrors bash subshell semantics.

### Async Stream Wrapping for Observability

Output streams from command handlers are wrapped in thin async generators that record bytes as they flow through. This means observability data (op records) captures actual consumption, not just dispatch — if a pipe's consumer only reads 100 bytes, the op record shows 100 bytes read, not the full file size.

### Provision Mode

Before executing a destructive or expensive command, `ws.execute(command, provision=True)` returns a `ProvisionResult` listing exactly which paths would be read and written, without actually performing the operations. This is a dry-run mechanism for agent safety.

## Design Decisions

### Bash as the Universal Interface

The core bet: LLMs are most fluent in bash and filesystem semantics (heavily represented in training data), so teaching them new tools is worse than making everything look like a filesystem. This is the opposite approach from MCP, which adds per-service tools. Mirage argues that `grep` across Slack messages and `cat` on a GitHub PR is more natural for agents than learning `slack_search` and `github_get_pr` tools.

**Trade-off**: Simpler for agents, but the filesystem metaphor breaks down for some resources. A Slack channel as a directory of date-named JSON files is clever but lossy — channel metadata, thread relationships, and reaction data require bespoke path conventions that the agent must learn.

### Virtual-First, FUSE-Optional

Mirage operates purely in-process by default (virtual mode). FUSE is an optional optimization for when you want actual kernel-level filesystem performance or need non-Python tools to access the mount tree. This avoids the complexity and platform-dependence of FUSE for the common case.

**Trade-off**: Virtual mode means all I/O goes through Python async, which is slower than kernel-level I/O. But it works everywhere without root, kernel modules, or platform-specific setup.

### Cache-Heavy by Default

Remote resources are aggressively cached. The first read hits the network; subsequent reads (within TTL) are local. This is the right default for agent workloads, where the same file is often read multiple times in a pipeline or across turns.

**Trade-off**: Stale data is possible with LAZY consistency. The ALWAYS mode exists but costs a stat call per read. There's no intermediate "check ETag" mode that would be cheaper than full stat but more current than LAZY.

### Comprehensive Mount Safety vs. Flexibility

The mount guardrails (can't rm mount roots, session-level allowlists, mode enforcement) add safety but also complexity. The `_check_mount_root_guard_raw` function is a 50-line case analysis of which commands can target mount roots under which flags. This is defensive engineering that pays off when agents run unsupervised.

### Snapshot for Reproducibility, Not Full State Capture

Snapshots capture fingerprints and cache bytes, not the full remote state. This is a pragmatic choice — you can't snapshot all of S3. But it means replay fidelity depends on remote data still existing and matching the fingerprint. The snapshot is a "what the agent saw" record, not a full backup.

## Agent Framework Integration

Mirage provides adapters for five agent frameworks, each with a different integration strategy:

- **OpenAI Agents SDK**: `MirageSandboxClient` plugs in as a sandbox backend; the agent's bash commands execute against Mirage mounts. Also provides `MirageRunner` for resolving Mirage paths as multimodal attachment blocks (images as base64 data URIs, PDFs via OpenAI Files API).
- **LangChain/DeepAgents**: `LangchainWorkspace` implements `SandboxBackendProtocol` — file ops go through the Ops layer directly, shell ops through `Workspace.execute()`. The monkey-patched `builtins.open` makes standard Python file I/O hit Mirage.
- **Pydantic AI**: Backend adapter providing tool registration that wraps Mirage ops.
- **CAMEL**: File and terminal adapters that expose the workspace to CAMEL's agent loop.
- **OpenHands**: Terminal and workspace adapters for OpenHands' sandbox environment.

## Resource Coverage

27 resources registered in `resource/registry.py`:

| Category | Resources |
|----------|-----------|
| Storage | RAM, Disk, Redis, S3, R2, OCI, Supabase, GCS |
| Communication | Slack, Discord, Telegram, Gmail, Email |
| Productivity | GDocs, GSheets, GSlides, GDrive, Notion, Trello, Linear |
| Development | GitHub, GitHub CI, SSH |
| Databases | MongoDB, Postgres |
| Observability | Langfuse |
| Misc | Paperclip (generic HTTP/file) |

Each resource follows the same pattern: `Config` pydantic model → `Resource` class → `core/` ops → `accessor/` typed accessor → `commands/builtin/` command implementations → `ops/` high-level ops → tests.

## Code Size

- ~147K total lines Python (source + tests)
- Largest files: `execute_node.py` (850 lines), `builtin_specs.py` (837 lines), `workspace.py` (776 lines), `builtins.py` (759 lines), `command.py` (754 lines)
- Most resource backends are 150-400 lines for their readdir implementation
- Tests are comprehensive per-resource with coverage of stat, read, readdir, scope, streaming, and integration

## Comparison to Related Approaches

### vs. MCP (Model Context Protocol)

MCP adds per-service tools that agents call by name. Mirage makes everything a filesystem that agents navigate with bash. The Mirage argument: LLMs already know bash and filesystem semantics from training data; adding N new tool names per service creates a vocabulary problem. The MCP argument: filesystem metaphors are lossy for structured APIs.

### vs. Locker (Open-source Dropbox alternative)

Locker provides a virtual bash shell over multiple storage backends. Similar idea but focused on human file management rather than agent tooling. Mirage adds agent-framework adapters, session isolation, snapshot/replay, and the command override system.

### vs. FUSE Filesystems (s3fs, gcsfuse, etc.)

Individual FUSE filesystems mount one backend. Mirage mounts many backends behind one tree with unified caching, session management, and agent integration. The per-backend FUSE approach requires the agent to know which path prefixes map to which backends; Mirage handles this transparently.

### vs. Direct SDK Usage

The obvious alternative: give the agent an S3 SDK, a Slack SDK, a GitHub SDK, etc. This is what most agent harnesses do today. Mirage argues this approach doesn't scale — each SDK has different semantics, error patterns, and authentication models. The filesystem abstraction normalizes all of these into one interface the agent already knows.
