---
url: https://github.com/losfair/zeroserve
title: zeroserve — Zero-config, fast, scriptable io_uring HTTPS server
author: losfair
date_fetched: 2026-06-15
date_published: 2025
---

# zeroserve — Full Architectural Analysis

## Summary

zeroserve is a Linux web server built on io_uring (via ByteDance's monoio runtime) that serves static sites from a single tarball and extends request handling with eBPF scripts JIT-compiled to native code. It can compile Caddyfiles to eBPF middleware, supports TLS 1.3 via BoringSSL with Encrypted Client Hello (ECH), and hardens itself with Linux namespace isolation and capability dropping. Single binary, no temp files, hot reload via SIGHUP.

_Version analyzed: v0.2.12-alpha.1_ — built from commit at origin/main (shallow clone).

## File Tree (key files)

```
src/
  main.rs              (512 lines) — entry point, CLI dispatch, namespace isolation, worker spawn
  server.rs            (6624 lines) — HTTP/1.1 + H2 connection handling, reverse proxy, TLS, ECH relay
  caddy_compile.rs     (13567 lines) — Caddy JSON → eBPF C middleware compiler
  script.rs            (2301 lines) — eBPF script runtime, helper dispatch, sandbox management
  site.rs              (454 lines) — tarball index, path→byte-range lookup, memfd-backed in-memory sites
  config.rs            (153 lines) — StaticConfig from CLI args
  cli.rs               (306 lines) — clap-derived CLI definition with ListenAddr, 40+ flags
  shared.rs            (199 lines) — ArcSwap-based shared state, tar file reading via io_uring
  pool.rs              (151 lines) — thread-local connection pool for reverse proxy
  oidc.rs              (751 lines) — OAuth2/OIDC client with XChaCha20-Poly1305 sealed cookies
  bpf_compiler.rs      (128 lines) — eBPF compiler selection (built-in tcc vs external clang)
  tinycc.rs            (618 lines) — FFI to built-in tinycc with eBPF backend
  reload.rs            (250 lines) — SIGHUP/reload coordination (canary worker pattern)
  ratelimit.rs         (146 lines) — in-memory rate limiter
  caddyfile/           — Caddyfile parser and Caddyfile→JSON adapter
    grammar.lalrpop    — LR(1) grammar for Caddyfile block interiors
    adapter/mod.rs     (5229 lines) — full Caddyfile adapter, ported from caddyconfig/httpcaddyfile
    parser.rs          (304 lines) — top-level server-block parser
    handlers.rs, matchers.rs, options.rs — directive-specific adapters
  http/h1.rs           (1342 lines) — custom HTTP/1.1 parser (no external dependency)
  ech/                 — Encrypted Client Hello key generation and config
  helpers/
    generic.rs         — request introspection (path, method, headers, query params)
    response.rs        — response construction
    crypto.rs          — SHA-256, HMAC, random
    encoding.rs        — base64, hex encode/decode
    json.rs            — JSON parsing/manipulation for eBPF scripts
    oidc.rs            — OIDC login/redirect/callback logic
    ratelimit.rs       — rate limiting helper
    aws_sign.rs        — AWS SigV4 request signing
    caddy.rs           — Caddy-specific runtime (response hooks, metadata, placeholders)
    call.rs            — inter-script calls via zs_call
    compress.rs        — gzip/zstd response compression
  server/caddy.rs      — Caddy runtime integration (TLS cert selection, access logging)
build.rs               (513 lines) — build script: generates lalrpop parser, downloads+compiles tinycc
sdk/
  zeroserve.h          — eBPF SDK header (full helper API declarations)
  zeroserve_caddy.h    — Caddy-specific SDK extensions
testing/               — Deno (TypeScript) e2e tests with ~18 test files
examples/              — 8 runnable C scripts (reverse proxy, OIDC, health check, logging, etc.)
benchmark/             — routing perf benchmarks, memory usage benchmarks
```

## Architecture

### Per-core worker model with io_uring

zeroserve spawns N worker threads (one per CPU core), each running its own monoio runtime (io_uring event loop). Each worker gets its own SO_REUSEPORT TCP listener, its own eBPF ScriptRuntime (programs are compiled and linked per-worker — never shared across threads), and its own reverse-proxy connection pool. This is not a thread-pool-on-top-of-event-loop model; it's full per-core isolation with kernel-level connection distribution.

The startup sequence in main.rs is precise about ordering: bind sockets → init TLS client CA → namespace isolation → capability dropping → load site → spawn workers. The reverse-proxy TLS client certs are loaded before namespace isolation because /etc gets replaced with an empty tmpfs afterward. This is the kind of detail that comes from real operational experience.

### Single-tarball static serving

A site is a tar archive. At load time, Site::load_from_file() iterates all tar entries and builds a HashMap<String, TarEntry> mapping normalized paths to {offset, size, etag, mtime}. Serving a file means a positional read on the still-open tar file. No extraction, no temp files. Directories and mtimes are also indexed for directory listing and conditional request support.

For the --caddy flow, the compiled eBPF middleware and site contents are packed into an in-memory tar via memfd_create(), so the site never touches disk.

### eBPF request scripting pipeline

The scripting model is:

1. Scripts are C files using the SDK (sdk/zeroserve.h), compiled to eBPF .o files
2. At startup, each worker compiles/loads scripts from all sites (plugins first, then main site)
3. Scripts export functions in specific ELF sections: "zeroserve.request" for HTTP request handling, "zeroserve.tls" for TLS certificate selection, "zeroserve.call.<name>" for inter-script calls
4. Each request invokes scripts in filename order; each script's entry function gets the current request state and can inspect, mutate, respond, proxy, or pass through

The helper API (declared in sdk/zeroserve.h, implemented in src/helpers/) includes: request path/method/header/query introspection, response construction, logging, time, random, SHA-256, HMAC, base64, hex, JSON parsing/manipulation, static file metadata, cookie get/set, reverse proxy, AWS SigV4 signing, rate limiting, OIDC login, VICI (strongSwan) lookups, template expansion, and inter-script calls.

### The pointer cage — eBPF sandboxing

This is zeroserve's most architecturally distinctive feature. Request scripts run in-process (same address space as the server), but are sandboxed by the "pointer cage" — a memory isolation technique implemented in the modified async-ebpf/uBPF JIT:

1. Each program gets a power-of-two sized anonymous mapping: stack region (RW), data region (R), guard pages (PROT_NONE) around and between
2. The JIT rewrites every load/store address to `(address & mask) + offset` — branchless, so no Spectre-v1 speculation window
3. A masked pointer landing in a guard region hits PROT_NONE → SIGSEGV handler translates to a clean program fault
4. After linking, data/code is frozen to read-only; all stores are confined to the stack region

The technique is a practical alternative to running eBPF in a kernel VM or separate process. It trades the overhead of context switches for the complexity of a custom JIT pointer-masking pass and SIGSEGV handling. Static region analysis is layered on top as a performance optimization (lets some loads skip a probe) but is explicitly not a security boundary — the cage holds regardless.

### Caddy compatibility

The Caddy support has three layers:

1. **Caddyfile adapter** (src/caddyfile/adapter/mod.rs, 5229 lines): a manual port of Caddy's httpcaddyfile package. Uses lalrpop for the block-interior grammar. Adapts Caddyfile syntax to Caddy JSON. Handles imports, snippets, heredocs, named routes, global options, and all supported directives.

2. **JSON compiler** (src/caddy_compile.rs, 13567 lines): compiles Caddy JSON into generated eBPF/C middleware. Emits C code that calls the zeroserve_caddy.h helper layer. Handles route matching, header mutation, URI rewriting, static responses, file_server, reverse_proxy, basic_auth, encode (gzip/zstd), and a zeroserve_call extension for bridging to native eBPF scripts inside route chains.

3. **Runtime layer** (src/server/caddy.rs, src/helpers/caddy.rs): Caddy-specific runtime behavior — response hooks via zs_call, metadata map shared between request/response phases, placeholder expansion, TLS client certificate verification.

The scope is pragmatic: supported behavior is validated with live comparison tests against stock Caddy. Unsupported features (body rewriting, multipart ranges, dynamic upstreams, PHP FastCGI runtime) are explicitly rejected rather than silently approximated.

### OIDC/OAuth2 without server-side state

The OIDC module (src/oidc.rs) implements an Authorization Code + PKCE relying party that stores all state in sealed cookies (XChaCha20-Poly1305 AEAD): login state cookie (PKCE verifier, CSRF state, nonce, 10-min TTL) and session cookie (id_token claims, 1-hour default TTL). No server-side session store — the encrypted cookies carry everything. id_token signature verification is skipped per OpenID Connect Core §3.1.3.7 note 2 (TLS channel provides equivalent assurance in the Authorization Code flow).

### Hot reload

SIGHUP triggers an atomic reload of the site tarball, TLS certificates, and scripts. The coordinator thread stages the reload: first one worker (the canary) loads the new configuration; if it succeeds, the rest follow. If a reload fails, the last known-good state is preserved. This is a simple but effective pattern for zero-downtime updates.

## Key Techniques

### Tar-as-filesystem

Instead of extracting, zeroserve indexes the tar at load time and serves files via positional reads. The tar file stays open for the process lifetime. ETags are computed at index time via BLAKE3 (first 16 bytes of the hash). This is a clever simplification: tar is a universal format, compatible with any tool that can create archives, and the indexing overhead is a one-time cost at startup/reload.

### Branchless pointer masking for Spectre resistance

The JIT's pointer rewrite `(address & mask) + offset` is branchless — it compiles to a single AND+ADD instruction pair on x86-64. A branch-based bounds check would leave a speculation window where the CPU could transiently access out-of-bounds memory before the branch resolves. The branchless mask/offset transform eliminates this entirely.

### memfd for in-memory tarballs

The --caddy flow and standalone script loading use memfd_create() to create file descriptors backed by anonymous memory. These behave exactly like regular files for positional reads (io_uring IORING_OP_READ with offset works), but never touch disk. This is the same mechanism behind Linux's "sealed memfd" but used creatively for in-memory site hosting.

### lalrpop for Caddyfile parsing

The Caddyfile block interior is parsed by an LR(1) grammar (src/caddyfile/grammar.lalrpop) with an external lexer. This is unusual — most projects would hand-write a recursive descent parser or use a PEG combinator library. The lalrpop approach ensures the parser matches Caddy's grammar precisely (Caddy uses yacc internally) and catches ambiguities at compile time.

### Built-in tinycc with eBPF backend

The build.rs downloads and compiles a patched tinycc (Tiny C Compiler) with an eBPF code generation backend. This is linked statically into the zeroserve binary. At runtime, script_compile.rs calls into tinycc via FFI to compile C source to eBPF bytecode. The entire compilation pipeline is in-process — no external compiler required. A build cache avoids recompiling tinycc on every cargo build.

### JA4 fingerprinting

The ja4.rs module computes JA4 TLS fingerprints from ClientHello parameters. JA4 is a modern replacement for JA3 that accounts for TLS 1.3, ECH, and QUIC. It's used for security/logging classification of connecting clients.

### Reload canary pattern

The reload coordinator (reload.rs) uses a "canary" pattern: one worker loads and validates the new configuration first; only after it succeeds do the remaining workers reload. This prevents a bad configuration from crashing all workers simultaneously.

## Design Decisions

### In-process eBPF vs out-of-process isolation

**Trade-off**: The pointer cage runs scripts in the same address space as the server (fast, no IPC overhead) but requires a custom JIT and SIGSEGV handler. The alternative — running scripts in a separate process or kernel eBPF VM — would be simpler but slower (context switch per request for separate process) or more limited (kernel eBPF can't do network I/O).

**Assessment**: This is the right call for a performance-oriented server. The Spectre-v1 resistant pointer masking is a genuine innovation, not just a reimplementation of known techniques. The specific risk is that the SIGSEGV handler becomes a safety-critical piece of code — any bug there compromises the entire sandbox.

### monoio (io_uring) vs tokio (epoll)

**Trade-off**: monoio gives true async file I/O (io_uring supports it; epoll doesn't — tokio spawns blocking threads for file ops). This matters for tarball reads. The downside is that monoio has a smaller ecosystem and no Tokio compatibility layer for most libraries. zeroserve works around this by implementing its own HTTP/1.1 parser and not using any async ecosystem libraries.

**Assessment**: The choice is coherent with the project's goals. If you're optimizing for zero-config fast serving from tarballs, you genuinely benefit from async file I/O. The cost is a significant amount of hand-rolled infrastructure (HTTP parser, TLS integration, etc.).

### Caddy compatibility depth vs breadth

**Trade-off**: The Caddy compiler supports a deep but specific subset of Caddy's behavior — enough to run real-world Caddyfiles from popular projects (Ghost, Mastodon, Immich, Gitea, etc.). It rejects unsupported features explicitly. The alternative would be to either be a complete drop-in replacement (impossible without reimplementing all of Caddy) or to support only a shallow set of directives.

**Assessment**: The "validate and reject what we can't do" approach is the right one. The CADDY_COMPAT.md is unusually honest about what's excluded and why. The live comparison tests against stock Caddy are the gold standard for compatibility claims.

### Stateless OIDC via sealed cookies

**Trade-off**: Storing session state in encrypted cookies avoids a server-side database, which fits the zero-config/no-temp-files philosophy. The trade-off is larger cookies (encrypted payload + AEAD tag) and no ability to revoke individual sessions (the server must cycle the cookie secret to invalidate all sessions).

**Assessment**: Appropriate for a single-binary web server that's meant to be simple to deploy. The 1-hour default TTL limits the blast radius of a stolen session cookie. XChaCha20-Poly1305 is a modern, well-regarded AEAD construction.

### Single binary, static linking

Everything is baked in: the tinycc compiler, the eBPF JIT, the HTTP parser, the TLS stack (BoringSSL via the `boring` crate), the Caddyfile parser, the OIDC client. The binary is self-contained — no runtime dependencies beyond the Linux kernel. This is the Go/Rust "single static binary" philosophy applied to a web server.

## Comparison Notes

- **vs Caddy**: zeroserve compiles Caddy configs to eBPF, not Go. It serves from tarballs, not directories. It has eBPF scripting that Caddy doesn't. Caddy has ACME automation that zeroserve doesn't. They occupy adjacent niches: Caddy is a general-purpose web server; zeroserve is a specialized server for packaged sites with programmable request pipelines.
- **vs Nginx/Apache**: zeroserve's eBPF scripting is a fundamentally different extension model than Nginx modules or Apache handlers. The pointer cage provides memory safety for extensions without process isolation — Nginx modules run in-process with no sandbox.
- **vs Lambda/Cloudflare Workers**: zeroserve's eBPF scripts are conceptually similar to edge workers (request interception, response generation, proxying) but run on your own hardware with io_uring performance. No cold starts, no platform lock-in.
- **vs [[Security and Sandboxing]]** tools: The pointer cage is a third category of sandbox — neither OS-level (like Seatbelt/Landlock) nor container-level (like Docker/gVisor), but JIT-level. It's more like WebAssembly's linear memory sandbox, but applied to native eBPF execution.

## Tags

#tool #project #server #ebpf #sandboxing #rust #io_uring #tls

## Source

- Repository: https://github.com/losfair/zeroserve
- Analyzed from shallow clone of main branch
- Key dependencies: monoio (io_uring runtime), async-ebpf (eBPF JIT), boring (BoringSSL), lalrpop (parser generator), tar, h2, serde_json

---
*Analyzed: 2026-06-15*
*Repo version: v0.2.12-alpha.1 (Cargo.toml)*
