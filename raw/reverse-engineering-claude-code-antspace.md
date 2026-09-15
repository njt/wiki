---
url: https://aprilnea.me/en/blog/reverse-engineering-claude-code-antspace
date_fetched: 2026-09-15
---

*What's inside Claude Code Web: an unstripped Go binary, Anthropic's secret deployment platform, and the architecture of an AI-native PaaS*

## The Starting Point

We are building ArcBox, a full-stack platform from Desktop to Platform, similar to Railway and E2B in positioning. Our core philosophy is local-cloud consistency: replacing OrbStack with a fully open-source ArcBox Desktop that provides Sandbox capabilities locally. Recently, we noticed more and more Coding Agent platforms launching web-based entry points, and remarkably, nearly all of them chose Firecracker under the hood. Claude Code is no exception. As practitioners in the same space, curiosity about its runtime environment led to some digging. What began as a casual `strace -p 1` turned into a full reverse-engineering session that uncovered unreleased Anthropic infrastructure, including an entirely undocumented application hosting platform.

Everything described here was discovered through standard Linux tooling (`strace`, `strings`, `objdump`, `go tool objdump`) running inside a Claude Code session. No exploits, no privilege escalation, no network attacks. The binary was sitting right there, unstripped, with full debug symbols.

## Layer 1: It's a Firecracker MicroVM

The first question: what exactly is this environment?

The ACPI tables are signed with OEM ID `FIRECK` and creator ID `FCAT`, both hardcoded in Firecracker's source code. This is the same MicroVM technology that powers AWS Lambda and Fargate.

The specs: 4 vCPUs (Intel Xeon Cascade Lake @ 2.80GHz), 16GB RAM, 252GB disk, Linux 6.18.5. No nested virtualization since Firecracker intentionally strips `vmx`/`svm` flags from guests.

The process tree is absurdly minimal:

No systemd. No sshd. No cron. No logging daemon. PID 1 is a custom binary that acts as both init and a WebSocket API gateway. The kernel command line confirms it:

`strace` on PID 1 shows it running an epoll event loop, periodically checking `/proc/*/children` and `/proc/*/status` to monitor child processes. Essentially a minimal init supervisor, listening on port 2024 (WebSocket API) and port 2025 (secondary endpoint).

## The Snapshot Architecture

Sessions don't boot from scratch — they're restored from frozen VM snapshots. The dmesg output reveals a 48.5-hour gap between template creation and session restore:

During restore, the Firecracker host **hot-swaps block devices**:

| Device | Template | After Restore | Content | 
|---|---|---|---|
| vda | placeholder | 256 GiB ext4 | Session rootfs (Ubuntu 24.04) | 
| vdb | placeholder | 63.7 MB squashfs | `/opt/claude-code` | 
| vdc | placeholder | 12.1 MB squashfs | `/opt/env-runner` | 

The initramfs is deliberately minimal: a 3.1MB cpio archive containing only `/process_api`. The actual Ubuntu rootfs is on the ext4 block device (vda), injected at restore time. The ext4 has mount count=11, indicating the image has been reused across 11 sessions.

### Snapstart: The Deferred Mount Pattern

**Template creation phase:**

- Firecracker boots: kernel + 3.1MB initramfs
- `process_api`performs minimal init: mount- `/proc`,- `/sys`,- `/dev`, cgroups; configure networking (IP=192.0.2.2/24, GW=192.0.2.1, MTU=1400)
- Signals `SNAPSTART_READY`to the host
- Host calls `PUT /snapshot/create`→ saves entire VM state

**Session restore phase:**

- Host prepares session-specific block devices (vda/vdb/vdc)
- Host calls `PUT /snapshot/load`with new device backends
- VM resumes — kernel detects device changes, reseeds CRNG
- `process_api`detects restore and runs:- **Drop page caches**— stale template cache would return garbage
- **Remount devtmpfs**— refresh device nodes
- **Mount ext4**→- `pivot_root`to new rootfs
- **Mount squashfs**overlays (claude-code, env-runner)
- **Fix wall clock**via- `clock_settime()`— otherwise stuck at template epoch
- **Drop CAP_SYS_RESOURCE**— security hardening
- **Accept connections**— WebSocket server ready
 

### Security Measures

| Measure | Purpose | 
|---|---|
| `init_on_free=1` | Zero freed pages between sessions | 
| CAP_SYS_RESOURCE drop | Limit PID 1's capabilities post-init | 
| CRNG reseed | Prevent crypto predictability across snapshot forks | 
| `--block-local-connections` | Block localhost WebSocket access | 
| JWT auth | WebSocket connection verification | 
| Token scrubbing | Remove secrets from configs after use | 

## process_api: The Wire Protocol

PID 1 exposes two network interfaces — a WebSocket API for process management and an HTTP API for container control. Unlike typical init systems, `process_api` is a Rust/tokio binary that implements a full remote process supervisor.

### WebSocket API (port 2024)

Connection handshake: optional JWT → `ProcessConnection` JSON → process creation or reattach.

Process creation accepts a `CreateProcess` struct:

I/O uses a two-phase binary protocol:

- **Stdin**:- `ExpectStdIn`(text) → binary frame
- **Stdout/Stderr**:- `ExpectStdOut`/- `ExpectStdErr`(text) → binary frame →- `StdOutEOF`/- `StdErrEOF`

Process termination reasons include: normal exit, signal, per-process OOM, container-level OOM, timeout, and server shutdown.

Internally, `process_api` tracks per-process cgroups (v1 at `/sys/fs/cgroup/memory/process_api/`, v2 at `/sys/fs/cgroup/process_api/`), implements orphan adoption (reparenting to PID 1), and runs a configurable OOM polling loop.

### HTTP Control API (port 2025)

Six endpoints manage container lifecycle:

| Endpoint | Purpose | 
|---|---|
| `GET /status` | Health check | 
| `POST /fs_sync` | Flush filesystem buffers | 
| `POST /shutdown` | Graceful shutdown with page cache drop | 
| `POST /auth_public_key` | Set JWT verification key | 
| `POST /mount_root` | Mount rootfs (snapstart restore) | 
| `POST /container_name` | Set container identity | 

The `/mount_root` endpoint accepts a `MountRootConfig` with network config (`etc_hosts`, `resolv_conf`), CA certs, squashfs mounts, FUSE mounts (with VFS cache config), and the wall clock timestamp — everything needed to initialize a session from a blank snapshot. During mount, root is frozen via `FIFREEZE`/`FITHAW` ioctls.

## Layer 2: The Unstripped Go Binary

The real discovery was `/usr/local/bin/environment-runner` (symlinked as `environment-manager`):

A 27MB Go binary. **Not stripped. Full debug info. Full symbol table.** Built from Anthropic's private monorepo at `github.com/anthropics/anthropic/api-go/environment-manager/`.

Using `go tool objdump` and `strings`, the complete internal package structure can be extracted:

Key dependencies extracted from the binary:

| Dependency | Purpose | 
|---|---|
| `github.com/anthropics/anthropic/api-go` | Internal Anthropic Go SDK | 
| `github.com/gorilla/websocket` | WebSocket tunnel to API | 
| `github.com/mark3labs/mcp-go v0.37.0` | Model Context Protocol | 
| `github.com/DataDog/datadog-go v5` | Metrics reporting | 
| `go.opentelemetry.io/otel v1.39.0` | Distributed tracing | 
| `google.golang.org/grpc v1.79.0` | gRPC (session routing) | 
| `github.com/spf13/cobra` | CLI framework | 

## Layer 3: Antspace, Anthropic's Hidden PaaS

Inside the `tunnel/actions/deploy/` package, there are function symbols for two deployment clients:

**VercelClient**, the expected one:

- `CreateDeployment`→ POST- `/v13/deployments`
- `UploadFile`→ PUT- `/v2/files`with- `x-vercel-digest`header
- `WaitForReady`→ Poll until- `readyState == "READY"`

And then, **AntspaceClient**, the unexpected one:

Extracting the associated strings from the binary revealed a complete deployment protocol:

**Phase 1: Create Deployment**

**Phase 2: Upload Build Artifact**

**Phase 3: Stream Deployment Status**

A search for "Antspace" across the entire public internet turned up nothing: Anthropic's website, GitHub, blog, documentation, LinkedIn, job postings, conference talks, patent filings. **Zero results.** This platform has never been publicly mentioned anywhere.

The name likely derives from "Ant" (reportedly an internal nickname for Anthropic employees) + "Space" (hosting space), following the same naming pattern as platforms like Heroku or Vercel.

### Antspace vs. Vercel: Architectural Differences

| Aspect | Vercel | Antspace | 
|---|---|---|
| File upload | SHA-based dedup, per-file | Single tar.gz archive | 
| Build | Remote (Vercel builds it) | Local `npm run build`, upload output | 
| Status | Polling-based | Streaming NDJSON | 
| Auth | Vercel API token + Team ID | Bearer token + dynamic control plane URL | 
| Public API | Yes, documented | No, completely internal | 

The fact that Anthropic built a full deployment protocol from scratch, rather than just wrapping Vercel's API, signals this is a strategic platform investment, not a quick integration.

## Layer 4: Baku, The Web App Builder

"Baku" is the internal codename for the web app builder experience on claude.ai. When you ask Claude on the web to build you a web application, it launches a Baku environment.

From the embedded resources extracted from the binary:

**Project Template:**

- Source: `/opt/baku-templates/vite-template`
- Stack: Vite + React + TypeScript
- Auto-managed dev server via supervisord, logs to `/tmp/vite-dev.log`

**Supabase Auto-Provisioning:**

Six MCP tools are automatically available:

- `provision_database`: create a Supabase project on demand
- `execute_query`: run SQL queries
- `apply_migration`: versioned schema changes with auto type generation
- `list_migrations`: list applied migrations
- `generate_types`: regenerate TypeScript types from DB schema
- `deploy_function`: deploy Supabase Edge Functions

Environment variables auto-written to `.env.local`:

**Stop Hooks (embedded shell scripts):**

The Baku environment has a pre-stop hook that prevents the session from ending if:

- There are uncommitted or unpushed git changes
- The Vite dev server log contains errors
- `tsc --noEmit`reports TypeScript type errors

**Default Deploy Target: Antspace**, not Vercel. Vercel exists as an alternative, but Baku's native deployment path goes through Anthropic's own platform.

**Internal Organization:**

- Drafts stored in `.baku/drafts/`
- Explorations in `.baku/explorations/`
- Git commits use `[email protected]`as the author
- No git remote configured (local-only version control)

## Layer 5: BYOC (Bring Your Own Cloud)

The `envtype/` package contains two environment implementations:

- `anthropic`
- `byoc`

BYOC allows enterprise customers to run `environment-runner` in their own infrastructure while sessions are orchestrated by Anthropic's API. Key characteristics:

- Default session mode: `resume-cached`(fastest restarts, reuses existing state)
- Custom auth: `containProvideAuthRoundTripper`injects container-level credentials
- Smart git handling: checks if task branch exists on remote before fetch
- Sub-types: `antspace`(Anthropic internal) and`baku`(Vite project builder)
- Kubernetes integration: `podmonitor`package implements lease management

The BYOC API surface includes 7 endpoints:

| Endpoint | Purpose | 
|---|---|
| `/v1/environments/whoami` | Identity discovery | 
| Work polling + ack | Job queue | 
| Session context | Configuration retrieval | 
| Code signing | Binary verification | 
| Worker WebSocket | Real-time tunnel | 
| Supabase DB query proxy | Database access relay | 

## The Strategic Picture

What we're looking at is a **vertically integrated AI application platform**:

This is not just an AI coding assistant. It's the architecture of an **AI-native PaaS** where the user's journey from idea to production happens entirely within Anthropic's infrastructure.

The competitive implications are significant. This positions Anthropic against:

- **Vercel / Netlify**in hosting and deployment
- **Replit / Lovable / Bolt**in AI app generation
- **Supabase / Firebase**in managed backend (via tight integration)

But with one structural advantage none of these competitors have: **Anthropic owns the entire stack**, from the LLM that understands your intent, to the runtime that builds your code, to the platform that hosts your application.

## Methodology

All findings were obtained through standard Linux tools running inside my own Claude Code session. Here's the step-by-step approach:

**Step 1 — Identify the hypervisor.** `dmesg | grep FIRECK` — ACPI tables carry the OEM ID, immediately fingerprinting Firecracker.

**Step 2 — Identify PID 1.** `cat /proc/1/cmdline` — reveals `/process_api` as a custom init, not systemd. The `--firecracker-init` flag confirms it.

**Step 3 — Extract Go build metadata.** `go version -m` on the environment-runner binary yields the Go version, module path (`github.com/anthropics/anthropic/api-go`), `(devel)` monorepo marker, and full dependency list.

**Step 4 — Recover package structure from symbols.** Because the binary is not stripped, `objdump -t` gives fully qualified function names. Filtering for `environment-manager/internal/` and extracting unique paths produces the complete architecture tree.

**Step 5 — Targeted string extraction.** Naive `strings | grep` on Go binaries is noisy (the linker concatenates all string literals). Two better approaches:

- *Known-pattern search*: scan for specific byte sequences (- `dist.tar.gz`,- `application/x-ndjson`, etc.)
- *Struct tag extraction*: Go embeds struct field tags as string literals. Searching for- `json:"status"`patterns reveals wire protocol formats

**Step 6 — Symbol-table-driven feature mapping.** `objdump -t binary | grep 'deploy\.'` yields method inventories per component — this is how both Vercel and Antspace clients were discovered with full method signatures.

**Step 7 — Runtime observation.** `strace -p 1 -e trace=epoll_pwait,read,write -f` confirms the epoll event loop, child process monitoring, and WebSocket communication patterns.

| Technique | What it reveals | 
|---|---|
| `dmesg`ACPI OEM ID | Hypervisor identity | 
| `go version -m` | Toolchain, dependencies, monorepo structure | 
| Symbol table ( `objdump -t`) | Package layout, type names, method signatures | 
| Struct tag strings | Wire protocol / JSON formats | 
| Targeted byte search | Error messages, status strings, flow logic | 
| `strace`on PID 1 | Runtime behavior, IPC patterns | 

**What made this possible:** (1) The binary was not stripped — the single biggest factor; a stripped binary would have required Ghidra. (2) Go's build metadata embedding provides a free dependency manifest. (3) Go's string concatenation model preserves struct tags and error strings. (4) Root access inside the VM allowed `strace` on PID 1.

## Closing Thoughts

Shipping an unstripped binary with full debug symbols to production is... a choice. It made this analysis trivial. What would normally require Ghidra and hours of decompilation was accomplished with `go tool objdump` and `grep`.

Antspace is clearly still in an early or internal stage (the version string is prefixed with `staging-`), but the deployment protocol is mature and production-grade. Whether Anthropic plans to launch this as a public product or keep it as internal infrastructure for Claude's web experience remains to be seen.

What's clear is that Anthropic's ambitions extend far beyond being just an LLM and AI agent company. They're building the infrastructure for a world where applications are spoken into existence, and they want to own every layer of that stack.

*All analysis was performed on March 18, 2026, inside a Claude Code Web session running on a Firecracker MicroVM with kernel 6.18.5, environment-runner version staging-68f0dff496.*

*The complete raw analysis files (wire protocol specs, snapshot architecture details, decompiled code) are available at github.com/AprilNEA/reverse-engineering-claude-code-antspace.*
