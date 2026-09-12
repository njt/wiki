---
url: https://github.com/nikvdp/cco
title: cco — Claude Container (Sandboxed AI Coding Agent Wrapper)
author: nikvdp
date_fetched: 2026-06-24
date_published: 2025
topics:
  - security-and-sandboxing
---

# cco — Full Architectural Analysis

## What it is

cco ("Claude Container" or "Claude Condom") is a bash-based sandboxing wrapper that runs AI coding agents (Claude Code, OpenAI Codex, Gemini CLI, OpenCode, Pi, Droid) in OS-level isolation. It auto-selects the best available backend: native OS sandboxing (macOS Seatbelt `sandbox-exec` or Linux `bubblewrap`) or Docker containers. The goal is to let agents run with `--dangerously-skip-permissions` while preventing filesystem damage outside the project directory.

## Project scale

- **3,702 lines** in `cco` (main entry point)
- **671 lines** in `sandbox` (cross-platform sandbox executor)
- **118 lines** in `docker-entrypoint.sh`
- **265 lines** across 6 agent modules (`lib/agents/*.sh`)
- **282 lines** in `tiocsti_filter.c` (seccomp BPF filter generator)
- **35+ test files** in `tests/`
- **~4,756 lines** total shell code, all bash (no Python/JS runtime — the whole tool IS bash)

## Architecture

### Dual-backend design

The core architectural decision: two sandboxing paths that share the same flag parser, credential system, agent configuration, and path rules.

**Backend selection** (`detect_sandbox_backend`, line 164):
- `auto` (default): macOS → `sandbox-exec` if available; Linux → `bubblewrap` if available (with AppArmor probe, line 149-161); falls back to Docker
- `native`: force OS sandbox, fail if unavailable
- `docker`: force Docker regardless of platform

**Native backend** (`run_native_sandbox`, line 1910): Collects all path rules into `sandbox_rules` array, calls `sandbox` script with `--write`, `--read-only`, `--deny` flags, plus `--safe` and `--allow-keychain` as needed. Loads `.env` file and `--env` flags, then `exec`s the sandbox script with the agent command.

**Docker backend** (`run_container`, line 2067): Constructs Docker args with UID/GID mapping (line 2067-2076), volume mounts with deduplication (`add_mount_arg`/`find_mount_spec_index`, lines 2103-2161), credential extraction and mounting, environment variable passthrough, networking setup, and SIGWINCH forwarding. Supports persistent containers with config hashing.

### Agent abstraction layer

The agent system (`lib/agents.sh`) is a simple dispatch pattern:
- `configure_agent_subcommand()` maps agent names to `command_flag` values and calls agent-specific path configurators
- `apply_agent_arg_policies()` applies agent-specific argument transformations
- Each agent module (`lib/agents/codex.sh`, `lib/agents/droid.sh`, `lib/agents/gemini.sh`, `lib/agents/opencode.sh`, `lib/agents/pi.sh`) implements `configure_<agent>_mode_paths()` and `apply_<agent>_arg_policies()`

Most agents are dead simple — just mount their config directories (e.g., `~/.gemini`, `~/.pi`, `~/.factory`). Codex is the exception with 210 lines of logic for shim-based PATH hijacking and argument normalization.

### sandbox script architecture

The `sandbox` script (line 1-671) is a standalone cross-platform sandbox executor:

**Linux path** (`run_linux`, line 122): Creates bubblewrap args with:
1. Base: RO root, /dev, /proc, tmpfs /tmp, cap-drop ALL (line 190)
2. Seccomp filter loading for TIOCSTI/TIOCLINUX blocking (line 134-187)
3. Common mounts RO: /home, /var, /root, etc. (line 205)
4. Safe mode: tmpfs over $HOME (line 210-212)
5. Overlay rules: rw, ro, deny with subpath exception handling (line 216-433)
6. Architecture-specific seccomp compilation from C source (line 150-179)

**macOS path** (`run_macos`, line 461): Generates a Seatbelt policy file using `(version 1)` scheme DSL:
1. Deny all file-write* globally (line 530)
2. Safe mode: deny file-read* under $HOME, allow only metadata reads (line 537-543)
3. Deny paths with LAST-MATCH-WINS ordering (line 548-562)
4. Allow writes to CWD, -w paths, temp dirs, Library/Caches, device files (line 566-633)
5. Keychain access if `--allow-keychain`: Mach service lookups + syscall 73 (line 639-647)

The deny-with-exceptions pattern in both backends is sophisticated: when a parent directory is denied but a subpath is allowed, the code creates temporary empty overlays (dirs with mode 111, files with mode 000) and then bind-mounts the exception subpaths on top.

### Credential management system

cco has an elaborate credential lifecycle manager (lines 914-1498):

1. **Credential discovery**: macOS Keychain via `security find-generic-password` (line 1000), file-based via `~/.claude/.credentials.json`
2. **OAuth expiry detection**: Parses `expiresAt` from credential payload, handles both millisecond and second epochs (line 1033-1065)
3. **Startup refresh**: If token is expired and backend can't persist refresh (native sandbox), runs `claude -p "Reply with OK"` unsandboxed to trigger OAuth refresh (line 1126-1148)
4. **Background refresh**: If token expires within 2 hours, spawns background refresh process with lockdir-based mutual exclusion (line 1150-1165)
5. **Sync-back**: After Docker sessions with `--allow-oauth-refresh`, syncs updated credentials back to host (Keychain or file) with backup-then-update pattern and concurrent-modification detection (line 1400-1498)

### Permission mode handling

cco builds permission arguments through `build_claude_permission_args` (line 548-560):
- Reads Claude's settings.json to check if `disableAutoMode` is set
- Reads `defaultMode` setting
- If user already passed `--permission-mode`, leaves it alone
- Otherwise adds `--dangerously-skip-permissions` when auto mode isn't enabled
- Agent-specific policies can inject additional args (Codex gets `--dangerously-bypass-approvals-and-sandbox`)

## Key techniques

### 1. bjq — embedded JSON query in bash

`bjq` (lines 294-473) is a custom JSON path query tool that works without `jq`:
- Pure bash path parsing for simple dot-notation and array indexing: `.permissions.disableAutoMode`, `additionalDirectories[0]`
- Falls through to `jq` if available (with sophisticated path traversal that handles both objects and arrays)
- Falls through to `python3` if jq unavailable (embedded Python script, lines 415-469)
- Supports `BJQ_OUTPUT=type` mode for checking JSON value types
- Used throughout for parsing Claude's settings.json

This is the most technically impressive single function — a three-tier JSON access layer in bash with graceful degradation.

### 2. Codex shim hijacking

For Codex mode in Docker (lines 127-210 of `lib/agents/codex.sh`):
- Creates a temporary `codex` shell script that sits earlier in PATH
- Filters out sandbox-mode flags (`--sandbox`, `-s`, sandbox_mode config)
- Re-injects `--dangerously-bypass-approvals-and-sandbox` if not already present
- Finds the real `codex` binary by scanning PATH and skipping the shim directory
- The shim is mounted read-only inside the Docker container and PATH-prefixed via `CCO_PREPEND_PATH`

This is necessary because Codex has its own sandboxing — cco needs to strip that so the outer cco sandbox is the one that applies.

### 3. Persist mode with config hashing

When `--persist` is used with Docker (lines 2615-2676):
- Computes a hash of all configuration: image name, scope root, host UID/GID, custom packages, `.env` content, filtered Docker args
- Stores as Docker label `cco.persist.config-hash`
- On subsequent runs, checks if existing container has matching hash — if not, refuses to reuse (prevents misconfiguration)
- Config-aware dedup: only the mutable aspects (mounts, env vars, packages) contribute to the hash; the project directory and `.claude` mounts are excluded since they change every invocation

### 4. SIGWINCH forwarding

Terminal resize handling (lines 2679-2697):
- Spawns a background monitor process that watches for the Docker container
- Traps SIGWINCH and forwards it via `docker exec ... pkill -SIGWINCH`
- Critical for Claude Code's TUI to reflow properly on window resize
- The monitor auto-terminates when the container stops

### 5. Seccomp filter with architecture detection

The C file `tiocsti_filter.c` (282 lines) generates BPF filters at either build time or runtime:
- All constants inlined — no kernel headers needed (line 24: "ALL CONSTANTS INLINED - NO KERNEL HEADERS REQUIRED")
- Architecture detection at compile time via `#if defined(__x86_64__)`
- x86_64 filter includes x32 ABI rejection (syscall number bit 30, line 60-61: "Prevents high-bit bypass CVE-2019-10063")
- Loads only low 32 bits of ioctl cmd arg to prevent 64-bit bypass
- Runtime compilation cache: compiles from source on exotic architectures, caches in `~/.cache/cco/`
- Checks source file mtime vs compiled filter for automatic rebuild

### 6. SSH keychain unlock flow

On macOS over SSH (lines 1190-1300):
- Detects SSH session via `SSH_CONNECTION`/`SSH_TTY` env vars
- Attempts Keychain credential read; if it fails with "user interaction is not allowed" error
- Checks if login keychain itself appears locked
- Prompts user to unlock (with `--yes` auto-accept support)
- Runs `security unlock-keychain` interactively

### 7. Container name determinism

Container naming (lines 85-110):
- Non-persist: `cco-<sanitized-dirname>-<epoch>-<pid>` (unique per run)
- Persist: `cco-<dir-slug>-persist-<name-slug>-<name-hash>-<path-hash>` (deterministic per project)
- Name collision detection: if a named-persist container already exists with different config hash, errors out rather than silently reusing wrong config

## Design decisions

### Bash-only by design (no runtime dependency)

The tool IS bash. No Python, Node, or Rust runtime. The `bjq` function has a Python fallback but explicitly checks `command -v python3` first. The seccomp filter generator is C but pre-compiled BPF blobs are shipped for x86_64 and aarch64. This is a deliberate choice: zero runtime dependencies beyond what ships with the OS.

### Agent-first, Claude-default

Claude Code is the default, but the architecture treats all agents uniformly through the `CCO_COMMAND`/`--command` mechanism and agent subcommand dispatch. Adding a new agent requires ~10 lines of bash. The agent abstraction is intentionally minimal — no shared protocol, just "mount this config dir" and "maybe tweak these CLI args."

### Credential read-only by default

Docker mounts credentials read-only unless `--allow-oauth-refresh` is explicitly set. This is a security trade-off: read-only means the container can't update credentials even if the agent writes to the file (it writes to the temp copy, not the host mount). The sync-back mechanism with concurrent-modification detection is there for users who want refresh, but it's opt-in.

### Filesystem-first isolation, not network

cco does NOT restrict network access. The Docker backend uses `--network=host` when available or `host.docker.internal` bridge. The native backend doesn't touch networking at all. This is a deliberate choice: coding agents need network access for package registries, API calls, MCP servers. The threat model is filesystem damage, not network exfiltration (that's left to egress proxies like yolo-cage's approach).

### Path precedence: deny > allow

In both backends, deny rules take precedence over allow rules. The sandbox creates empty overlays with no permissions for denied paths. If a subpath is allowed under a denied parent, the overlay system creates exec-only directory scaffolding (mode 111) and then bind-mounts the exception on top. This is non-trivial to get right in both bubblewrap and Seatbelt simultaneously.

### Project as security boundary

The default sandbox grants write access to `$PWD` and blocks writes everywhere else. This is the same model as Codex's default sandbox mode and contrasts with Claude Code's command-whitelist approach. The project IS the security boundary — if you clone a malicious repo and run `cco` inside it, the agent has full write access to that repo.

## Comparison to related projects

### vs. OpenSandbox (Alibaba)
OpenSandbox is a platform — Docker, Kubernetes, gVisor, Kata Containers, Firecracker, network policies, multi-language SDKs. cco is a single bash script. Different scale entirely. OpenSandbox is for production agent platforms; cco is for individual developers in their terminal.

### vs. yolo-cage
yolo-cage uses Vagrant VMs (8GB RAM min) and adds egress proxy + branch isolation + secret scanning. cco is much lighter (native sandbox has near-zero overhead) but doesn't do egress filtering. yolo-cage has a richer threat model (exfiltration prevention); cco has a lighter touch (filesystem-only, zero config).

### vs. agentsh
agentsh sits at the syscall boundary (FUSE + eBPF + seccomp) and does `redirect` as a policy primitive — transparently swapping commands, paths, DNS responses. cco operates at the mount namespace level. agentsh is more granular but more complex and Linux-only; cco is simpler but cross-platform.

### vs. Codex's built-in sandbox
Codex ships its own sandbox (Seatbelt on macOS, Landlock+seccomp on Linux). cco can sandbox Codex too — the codex-mode strips Codex's sandbox flags and wraps it in cco's sandbox instead. This is useful when you want consistent sandboxing across multiple agents or when you want Docker-based isolation for Codex.

### vs. Zeroclaw
Zeroclaw (31k stars) is a Rust agent runtime with trait-based architecture, 30+ channels, and OS-level sandboxing. cco is agent-agnostic — it wraps the agent binary, not the agent logic. Zeroclaw IS the agent; cco wraps whatever agent you want.

### vs. claudebox
claudebox is mentioned in cco's README comparison table as a feature-rich container environment with 15+ development profiles and per-project Docker images. cco positions itself as the simpler, zero-config alternative.

## Tags
#tool #security #sandboxing #coding-agents #claude-code #bash #docker #seatbelt #bubblewrap #seccomp
