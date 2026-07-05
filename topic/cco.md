# cco — Claude Container

cco is a zero-dependency bash wrapper that sandboxes AI coding agents (Claude Code, Codex, Gemini CLI, OpenCode, Pi, Droid) at the OS level. It auto-selects the best isolation backend available — macOS Seatbelt, Linux bubblewrap, or Docker — and lets agents run with full autonomy while preventing filesystem writes outside the project directory. The entire tool is a single 3,700-line bash script; it needs no Python, Node, or Rust runtime. If you want Claude Code's `--dangerously-skip-permissions` speed without the risk of it deleting your home directory, this is the lightest possible answer.

---

## Architecture

cco has two execution paths that share the same flag parser, credential system, agent config, and path rules:

**Native backend** (`run_native_sandbox`, `cco:1910`). Collects path rules into a flat array of `--write`, `--read-only`, `--deny` flags, then `exec`s the standalone `sandbox` script. On macOS this generates a Seatbelt policy with the Scheme-like `(version 1)` DSL; on Linux it builds a bubblewrap arg list with bind mounts and seccomp filters. The sandbox script is cross-platform — `run_linux` (`sandbox:122`) and `run_macos` (`sandbox:461`) produce equivalent isolation from different primitives.

**Docker backend** (`run_container`, `cco:2067`). Builds Docker args with UID/GID remapping (handles macOS UIDs like 501 correctly), volume mounts with deduplication (`add_mount_arg`, `cco:2129`), credential extraction and mounting, environment variable passthrough from a curated whitelist of ~30 vars, and SIGWINCH forwarding for proper terminal resizing. Supports `--persist` for container reuse with config-hash validation.

**Agent dispatch** (`lib/agents.sh`). Each supported agent gets ~10 lines of bash: a path configurator (mount `~/.codex`, `~/.gemini`, etc.) and an optional arg policy. Codex is the exception at 210 lines — it injects a PATH shim that strips Codex's own sandbox flags so cco's outer sandbox is the only one that applies.

**Credential lifecycle** (`cco:914-1498`). Extracts Claude credentials from macOS Keychain via `security find-generic-password`, detects OAuth token expiry from `expiresAt` field, runs unsandboxed refresh when needed, and optionally syncs refreshed tokens back to the host with backup-before-write and concurrent-modification detection.

---

## Key Techniques

**bjq — embedded JSON query in bash** (`cco:294-473`). A three-tier JSON accessor: pure bash path parsing for simple queries, falls through to `jq` if available, falls through to an inline Python script if not. Handles nested objects and array indexing. This is how cco reads Claude's `settings.json` to decide whether to inject `--dangerously-skip-permissions`.

**Deny-with-exceptions overlays.** When a parent directory is denied but a subpath must be allowed, both backends create temporary empty overlays (dir mode 111, files mode 000), bind-mount them over the denied path, then bind-mount the exception subpaths on top. This is the trickiest part of the sandbox script — matching bubblewrap's mount semantics with Seatbelt's LAST-MATCH-WINS ordering, correctly building directory scaffolding in the overlay so bind targets exist.

**Seccomp BPF filter** (`seccomp/tiocsti_filter.c`). A 282-line C program with all constants inlined (no kernel headers needed) that generates BPF to block TIOCSTI and TIOCLINUX ioctls (CVE-2017-5226, CVE-2023-1523). Architecture-aware at compile time; x86_64 filters additionally reject x32 ABI syscalls to prevent a known bypass. Pre-compiled filters shipped for x86_64 and aarch64; runtime compilation from source on exotic architectures with automatic cache invalidation.

**Codex shim hijacking** (`lib/agents/codex.sh:127-210`). Creates a temporary `codex` shell script placed earlier in PATH that filters out `--sandbox`/`-s` flags and re-injects `--dangerously-bypass-approvals-and-sandbox`. The shim finds the real `codex` binary by scanning PATH while skipping its own directory. This is necessary because Codex has its own sandbox — cco strips it so the outer cco sandbox is authoritative.

**Persist config hashing** (`cco:2615-2676`). `--persist` mode computes a hash of all configuration (image, scope root, UID, custom packages, Docker args, `.env` content) and stores it as a Docker label. On reconnection, cco checks that the existing container's config hash matches — if not, it errors rather than silently reusing wrong configuration. This prevents the most common failure mode of persistent containers: stale assumptions about what's mounted.

---

## Design Decisions

**Bash-only, zero runtime deps.** The tool IS bash. No package.json, no pip install, no `cargo build`. The seccomp filter generator is C but pre-compiled for common architectures. This is a deliberate choice: a sandbox that requires a runtime to install is a sandbox you won't install when you need it.

**Filesystem-only isolation, no network restriction.** Both backends pass through full network access. This is a trade-off: coding agents need network for package registries, API calls, and MCP servers. The threat model is filesystem damage, not data exfiltration. If you need egress filtering, combine cco with [[yolo-cage]]'s egress proxy approach.

**Credentials read-only by default, refresh opt-in.** Docker mounts credentials as read-only unless you pass `--allow-oauth-refresh`. The sync-back mechanism is sophisticated (backup-before-write, concurrent-modification detection) but explicitly opt-in because writing credentials from inside a sandbox to the host is inherently risky.

**Project as security boundary.** The default grants write access to `$PWD` and blocks everything else. Same model as Codex's default sandbox. Contrast with Claude Code's command-whitelist approach — cco doesn't care what command you run, only where it writes.

**Agent-agnostic by design.** The `--command` flag and `CCO_COMMAND` env var let you wrap any binary. The agent subcommands (`cco codex`, `cco gemini`, etc.) are convenience wrappers that mount the right config directories. Adding a new agent is ~10 lines of bash in `lib/agents/`.

---

## Comparison Notes

**vs. [[A Deep Dive on Agent Sandboxes | Codex's built-in sandbox]]**: Codex ships Seatbelt/Landlock+seccomp sandboxing. cco can sandbox Codex too — codex-mode strips Codex's sandbox flags and wraps it in cco's sandbox instead. Useful when you want consistent isolation across multiple agents or Docker-based isolation for Codex.

**vs. [[yolo-cage]]**: yolo-cage uses Vagrant VMs (8GB RAM minimum) with egress proxy and branch isolation. cco is lighter (native sandbox has near-zero overhead) but doesn't filter outbound traffic. Complementary: yolo-cage for exfiltration prevention, cco for filesystem isolation.

**vs. [[OpenSandbox]]**: OpenSandbox is a platform — Docker, Kubernetes, gVisor, Kata Containers, Firecracker, multi-language SDKs, CNCF Landscape. cco is a single bash script. Different scale entirely.

**vs. [[Zeroclaw]]**: Zeroclaw is a Rust agent runtime with sandboxing built into the agent itself. cco wraps the agent binary — it's agent-agnostic. Zeroclaw IS the agent; cco wraps whatever agent you use.

**vs. [[agentsh]]**: agentsh sits at the syscall boundary (FUSE + eBPF + seccomp) with `redirect` as a policy primitive. cco operates at the mount namespace level. agentsh is more granular but Linux-only; cco is simpler but cross-platform.

**vs. [[claude-code-config (Trail of Bits)]]**: Trail of Bits' approach uses Claude Code's hook system (PreToolUse/PostToolUse) for security enforcement. cco operates below Claude Code entirely — it doesn't matter what hooks are configured because the sandbox prevents filesystem access directly.

---

## Tags
#tool #security #sandboxing #coding-agents #claude-code #bash #docker

## Source
- Repository: [nikvdp/cco](https://github.com/nikvdp/cco)
- Analysis date: 2026-06-24
- [[summary/cco|Full raw analysis]]
