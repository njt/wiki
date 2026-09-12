# zeroserve

A Linux web server that serves static sites from a single tarball and extends request handling with eBPF scripts JIT-compiled to native code. It compiles Caddyfiles to eBPF middleware, supports TLS 1.3 with ECH via BoringSSL, sandboxes scripts in-process with a branchless "pointer cage," and runs as a self-contained single binary. Built on monoio (io_uring) with per-core worker isolation.

---

## Architecture

zeroserve is a **per-core event-loop server** written in Rust. Each worker thread gets its own monoio runtime (io_uring), SO_REUSEPORT listener, eBPF ScriptRuntime, and reverse-proxy connection pool. Workers never share program state (`src/main.rs:355-410`).

**Core data flow**: TCP accept → HTTP parse (custom H1 parser, `src/http/h1.rs`, or h2 via the `h2` crate) → eBPF script chain (scripts ordered by filename, `src/script.rs:42-52`) → response or reverse proxy. Scripts run in-process but are contained by the pointer cage (see below).

**Site model** (`src/site.rs`): A tar archive indexed at load time into a `HashMap<String, TarEntry>` mapping paths to `{offset, size, etag}`. Serving a file is a single positional read on the open tar — no extraction. The `--caddy` flow builds the tar in-memory via `memfd_create()`.

**Caddy compatibility** is three layers:
1. **Caddyfile adapter** (`src/caddyfile/adapter/mod.rs`, 5229 lines) — lalrpop LR(1) parser, manual port of `caddyconfig/httpcaddyfile`
2. **JSON compiler** (`src/caddy_compile.rs`, 13567 lines) — Caddy JSON → generated eBPF/C middleware with helper calls
3. **Runtime** (`src/helpers/caddy.rs`, `src/server/caddy.rs`) — response hooks, metadata map, TLS cert selection, access logging

**TLS** (`src/tls.rs`, `src/boringtls.rs`): BoringSSL via the `boring` crate. SNI certificate selection from a PEM directory. ECH support with key rotation, rejection, and transparent relay fallback (`src/ech/`, `src/server.rs:429-488` for relay).

## Key Techniques

### The pointer cage — branchless in-process eBPF sandboxing

This is zeroserve's most distinctive technique. Rather than isolating scripts in a separate process or kernel VM, they run in-process with a JIT-level memory cage:

1. Each program gets a power-of-two anonymous mapping with RW stack + R data + PROT_NONE guard pages
2. The JIT (patched uBPF) rewrites every load/store to `(address & mask) + offset` — **branchless**, so no Spectre-v1 speculation window
3. Guard page hits → SIGSEGV → clean program fault (not server crash)
4. Data/code frozen to read-only after linking; all stores confined to stack region

Described in detail in the README and implemented across `src/script.rs`, the async-ebpf dependency, and the modified uBPF JIT. This is a novel approach — neither OS sandboxing (Seatbelt, Landlock) nor container isolation (Docker, gVisor) nor Wasm linear memory, though closest in spirit to Wasm's approach.

### Tar-as-filesystem with BLAKE3 ETags

Files are served directly from the indexed tar via positional reads (`src/shared.rs:92-158`). ETags are computed once at index time using BLAKE3's first 16 bytes (`src/site.rs:429-443`). Combined with `io_uring` async file I/O, this gives true non-blocking file serving — something epoll-based runtimes can't do without thread pools.

### In-process C→eBPF compilation via built-in tinycc

`build.rs` downloads and compiles a patched tinycc with an eBPF codegen backend, linked statically. At runtime, `src/script_compile.rs` calls tinycc via FFI (`src/tinycc.rs`) to compile C source to eBPF bytecode on the fly. This means no external compiler needed — scripts can be raw `.c` files and zeroserve handles everything.

### memfd for zero-disk site hosting

The `--caddy` and standalone script paths use `memfd_create()` (`src/site.rs:262-272`) to create file descriptors backed by anonymous memory that behave like regular files for positional reads. Sites exist entirely in RAM, with the same read path as on-disk tarballs.

### Reload canary pattern

SIGHUP reloads are staged: one "canary" worker loads the new configuration first; only after it succeeds do remaining workers reload (`src/reload.rs`). If reload fails, the last known-good state is preserved. Simple but prevents a bad config from crashing all workers at once.

### Branchless H2 preface detection

The connection handler peeks for the HTTP/2 preface (`PRI * HTTP/2.0\r\n\r\nSM\r\n\r\n`) using `MSG_PEEK` on the raw fd (`src/server.rs:518-549`), allowing H1/H2 detection without consuming bytes — the full preface is then consumed by whichever protocol handler takes over.

## Design Decisions

**In-process eBPF vs out-of-process isolation**: Chose in-process (pointer cage) for speed — no IPC per request. Cost is a custom JIT with SIGSEGV handling. Right call for this project's performance goals, but the SIGSEGV handler is safety-critical. A bug there compromises everything.

**monoio (io_uring) vs tokio (epoll)**: Chose monoio for true async file I/O — tarball serving benefits directly. Cost is a smaller ecosystem: zeroserve implements its own HTTP/1.1 parser and does not use any async ecosystem libraries. Coherent with the goal of fast zero-config serving.

**Caddy depth vs breadth**: Supports a deep subset (enough for real-world Ghost, Mastodon, Immich, Gitea Caddyfiles) but explicitly rejects unsupported features rather than approximating them. `CADDY_COMPAT.md` is unusually honest about exclusions.

**Stateless OIDC via sealed cookies**: Session state lives in XChaCha20-Poly1305 encrypted cookies — no server-side store. Fits the zero-config philosophy. Trade-off: no per-session revocation without cycling the cookie secret. `src/oidc.rs:1-17` documents the design explicitly.

**Single binary**: Everything is baked in — tinycc, BoringSSL, the eBPF JIT, the HTTP parser, the Caddyfile parser, the OIDC client. No runtime dependencies beyond the Linux kernel. Docker image is just the binary on debian:slim.

## Comparison Notes

- **vs Caddy**: zeroserve compiles Caddy configs to eBPF, serves from tarballs, has eBPF scripting — things Caddy doesn't do. Caddy has ACME automation, a plugin ecosystem, and broader protocol support that zeroserve doesn't. Adjacent niches: Caddy is general-purpose; zeroserve is for packaged sites with programmable pipelines.
- **vs Nginx/Apache**: eBPF scripting is a fundamentally different extension model. Nginx modules run in-process with no sandbox. zeroserve's pointer cage provides memory safety for extensions without process isolation.
- **vs Cloudflare Workers / edge functions**: Same concept (request interception, response generation, proxying) but on your own hardware with io_uring performance. No cold starts, no platform lock-in.
- **vs [[Security and Sandboxing]]** approaches: The pointer cage is a third category — JIT-level memory sandboxing, neither OS-level (Seatbelt, Landlock) nor container-level. Most similar to Wasm's linear memory model, but for native eBPF execution.
- **vs [[A Deep Dive on Agent Sandboxes]]**: zeroserve's approach inverts the "default sandboxed" philosophy at a different layer. Codex sandboxes the *process* (Seatbelt on macOS, Landlock+seccomp on Linux); zeroserve sandboxes *individual scripts within the process* via the JIT.

## Tags

#tool #project #server #ebpf #sandboxing #rust #io_uring #tls #caddy

## Cross-references

- [[Security and Sandboxing]] — the pointer cage as a third category of sandboxing
- [[A Deep Dive on Agent Sandboxes]] — OS-level process isolation vs JIT-level script isolation
- [[Local and Open Source Inference]] — single-binary self-contained deployment pattern
- [[Mirage (VFS)]] — tar-as-filesystem as a virtual filesystem technique

---
*Source: [[summary/zeroserve]]*
*Analysis from repository clone, 2026-06-15*
*Repo at v0.2.12-alpha.1, ~33K LoC Rust core*
