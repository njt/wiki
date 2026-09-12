---
url: https://github.com/perplexityai/bumblebee
title: "bumblebee: Endpoint Package Inventory Collector"
author: Perplexity AI
date_fetched: 2026-07-05
date_published: 2026-05-15
topics:
  - security-and-sandboxing
---

# bumblebee — Deep Architectural Analysis

Bumblebee is a read-only endpoint package inventory collector written in Go by Perplexity AI. It answers the question: when a supply-chain advisory names a package, extension, or version, which developer machines show a match in their on-disk metadata right now?

## Project Metrics

- **Language**: Go 1.25+
- **Dependencies**: Zero non-stdlib dependencies (go.mod contains only `module` and `go 1.25`)
- **Version**: 0.1.1
- **Core source**: ~3,664 lines across 9 key files; total repo ~9,000+ lines including tests, docs, and threat intel
- **Entry point**: `cmd/bumblebee/main.go` (449 lines)
- **Key modules**: `internal/scanner/` (562 lines orchestrator), `internal/ecosystem/mcp/` (853 lines, largest single file), `internal/model/` (389 lines), `internal/walk/` (266 lines), `internal/exposure/` (330 lines)

## Architecture

### Single Binary, Zero Dependencies

The most disciplined Go project this size. `go.mod` contains only the module path and `go 1.25` — no replaces, no indirects, no third-party imports. All JSON parsing uses `encoding/json`, all file I/O uses `os`/`io`, all hashing uses `crypto/sha256`. This is an explicit design choice: a supply-chain security scanner cannot itself be part of the supply-chain problem.

### Pipeline Architecture

```
CLI (main.go) → Resolve Roots (roots.go) → Walk Filesystem (walk/walk.go)
  → Dispatch to Ecosystem Scanners (scanner/scanner.go)
    → Parse Metadata → Emit NDJSON Records (output/output.go)
      → Optional: Match Against Exposure Catalog (exposure/exposure.go)
        → Emit Finding Records
```

The flow:
1. `main.go` parses flags, normalizes the profile, loads the exposure catalog if provided, opens the output sink (stdout/file/HTTP), then calls `scanner.Run()`.
2. `scanner.Run()` creates one scanner instance per ecosystem, starts a worker pool (default 4 goroutines), walks the configured roots, and dispatches matching files to the appropriate scanner via a buffered channel.
3. Each ecosystem scanner (npm, pypi, pnpm, yarn, bun, go, rubygems, composer, mcp, skills, editorext, browserext, homebrew) reads exactly the metadata files needed and calls `emit()` with a populated `model.Record`.
4. The emitter deduplicates by content-addressed `record_id`, writes NDJSON to the output sink, and if a catalog is loaded, matches each record against it to produce findings.

### Key Insight: Dispatch by Filename, Not by Walking node_modules

The walker (`internal/walk/walk.go`) visits directories but only opens files matched by exact filename patterns (e.g., `package-lock.json`, `METADATA`, `INSTALL_RECEIPT.json`). It never descends into `node_modules` subtrees — the npm/pnpm/yarn scanners walk their own bounded depth within those directories. This avoids the `node_modules` explosion problem that would make a naive `filepath.Walk` useless on a typical developer machine.

### Three Scan Profiles

| Profile | What It Scans | Use Case |
|---|---|---|
| `baseline` | Global/user package roots, toolchains, editor/browser extensions, MCP configs | Recurring fleet inventory (every 6h) |
| `project` | Configured dev directories (`~/code`, `~/src`, etc.) | Daily project sweep |
| `deep` | Explicit `--root` paths (including `$HOME`) | On-demand incident response |

`baseline` and `project` refuse bare-home roots (`$HOME`, `/Users/<name>`) — only `deep` walks them. This is a deliberate safety gate: `baseline` is a curated allowlist of known package roots, not a filesystem crawl.

## Key Techniques

### 1. Exclude-Driven Safety Rather Than Include-Driven Precision

The walker's `DefaultExcludes` list (162 lines in `internal/walk/walk.go`) is the project's most security-conscious code. It excludes credential directories (`.ssh`, `.gnupg`, `.aws`, `.kube`), browser profiles, macOS TCC-protected trees, email, messages, keychains, and dozens of Library subtrees. The excludes apply even when an operator passes `--root "$HOME"` — they only narrow what is walked under each root. This is the difference between "don't visit known-dangerous directories" and "visit everything and hope you don't find secrets."

The exclude matching uses both basename matching (`.git`, `.ssh`) and suffix-component matching (`Library/Caches`, `.config/gcloud`), so a path like `/Users/alex/Library/Caches/foo` is excluded by matching the `Library/Caches` suffix.

### 2. Content-Addressed Record Identity

Every record type (package, finding, scan_summary, diagnostic) computes a `record_id` via `sha256(recordType + "\x00" + joinWithUnitSeparator(canonicalFields))`. The field separator is ASCII unit separator (`\x1e`), chosen to avoid collisions with field content. This means:
- Records are stable across runs: the same package observed twice on the same host produces the same `record_id`.
- Deduplication is deterministic: `Emitter.ObservePackage()` checks a `map[string]struct{}` keyed by `DedupKey()`.
- Receivers can use `record_id` as a join key across runs without trusting `run_id`.

### 3. MCP Config Parsing as Supply-Chain Surface

The MCP scanner (`internal/ecosystem/mcp/mcp.go`, 853 lines) is the most complex ecosystem scanner, reflecting the fact that MCP configs are a novel attack surface that conventional package scanners don't cover. It handles:

- **Three envelope shapes**: `{ mcpServers: {...} }`, `{ servers: {...} }`, and flat `{ "<id>": {...} }`.
- **Launcher inference**: Parses `npx -y @modelcontextprotocol/server-github`, `uvx mcp-server-time`, `docker run ...`, `python -m mypkg.server`, `pipx run --spec ...` to extract the actual package identity from command/args.
- **Docker image refs**: Splits `hashicorp/terraform-mcp-server:0.4.0` into name+version, handling registry-port refs (`localhost:5000/foo/bar:1.2.3`) and digest refs (`name@sha256:...`).
- **Credential sanitization**: Drops `env` values entirely from MCP records. Sanitizes remote URLs to `scheme://host` only (strips userinfo, path, query, fragment). Skips unresolved shell variables (`${CLAUDE_PLUGIN_ROOT}/foo`).
- **Package spec validation**: `looksLikePackageSpec()` rejects URLs, file paths, VCS refs, tarball references, and credential-bearing specs that could leak through `PackageName` or `RequestedSpec`.

This is the only v0.1 scanner that actively prevents credential leaks — most others just don't read credential files. The MCP scanner goes further because MCP configs carry credentials inline (`env` blocks, registry URLs with tokens, etc.).

### 4. npm Lifecycle Script Detection Without Running Anything

The npm scanner reads `package-lock.json` and `package.json` to detect packages with lifecycle scripts (`install`, `postinstall`, `preinstall`, `prepare`). It records the script names as `lifecycle_scripts` but never reads script bodies. This gives responders a signal (which packages could execute code on install) without the risk of executing anything or capturing script content that might itself be sensitive.

### 5. Browser Extension Coverage Without Reading Profile Data

The browser extension scanner walks only `Extensions/<id>/<version>/manifest.json` for Chromium-family browsers and `extensions.json` for Firefox. The walker explicitly excludes cookies, login data, IndexedDB, local storage, and cache through the curated root paths (only `Default` and `Profile 1`-`Profile 9` are enumerated) and the default excludes. On macOS, the entire browser app-support tree is excluded from deep walks via `Library/Application Support/<browser>` excludes.

### 6. Node Modules Without the Explosion

The npm scanner reads `node_modules/<pkg>/package.json` and `node_modules/@scope/<pkg>/package.json` at exactly two depths. It does not enumerate deeper — no walking into transitive dependencies' own `node_modules/`. The pnpm scanner reads from pnpm's content-addressed store layout (`.pnpm/<name>@<version>/node_modules/<name>/package.json`). Both approaches give installed-package coverage without the combinatorial explosion of a full `node_modules` walk.

### 7. Embedded Self-Test Fixtures

`cmd/bumblebee/selftest.go` embeds fixture directories via Go's `//go:embed` directive and runs the full scanner pipeline against them. The fixtures use deliberately fake package names (`bumblebee-selftest-evil@0.0.0`) and the embedded catalog expects exactly 5 findings. The test makes zero network calls and runs in ~1ms. This gives fleet operators a pre-deployment smoke test: `bumblebee selftest` before rolling to production.

### 8. HTTP Output with HMAC Signing

The HTTP sink (`internal/output/httpsink.go`) supports three auth modes: none, bearer token, and HMAC-SHA256. The HMAC mode signs the raw POST body (optionally with a timestamp prefix) and sends `X-Inventory-Signature: sha256=<hex>`. When gzip is enabled, compression happens before signing. This is a complete, documented wire protocol for receiving inventory data, not a hack.

## Design Decisions

### Sacrificed: Version Resolution and Rich Metadata

v0.1 deliberately does not resolve version ranges, compute integrity hashes, or capture rich metadata (resolved URLs, dependency graphs, hash digests). The `model.Record` struct is "intentionally slim" — 16 fields, no free-form notes, no hash digests. This is a trade-off: more records with less metadata each, optimized for the exact-match exposure check use case rather than comprehensive SBOM generation.

The `confidence` field (high/medium/low) is the compression mechanism: it tells consumers how much to trust the version without capturing all the evidence.

### Sacrificed: Ecosystem Breadth

Notable omissions in v0.1: Cargo (`Cargo.lock`), Maven/Gradle, NuGet, Hex (`mix.lock`), Swift PM (`Package.resolved`), Yarn PnP (`.pnp.data.json`), Bun binary lockfile (`bun.lockb`). The authors chose depth over breadth — covering npm/PyPI/Go/RubyGems/Composer deeply with their specific metadata formats rather than adding shallow coverage of more ecosystems.

### Sacrificed: Delta Tracking

Bumblebee is snapshot-only. Each run emits a complete inventory and exits. It does not keep an endpoint-side delta database, does not diff against previous runs, and does not cache state. The receiver derives current state from complete snapshots. This keeps the endpoint simple and avoids bad deltas after missed runs, parser changes, or local state corruption. The cost is more data to transmit each run.

### Chosen: Single Profile per Run

Bumblebee is intentionally one profile per invocation. You can't run `baseline` and `project` in a single scan. The caller (cron, launchd, MDM) controls cadence per profile. This separates populations cleanly: receivers use `(endpoint_id, profile)` as the primary key for current state, and `baseline` runs never retire packages discovered by `project` runs.

### Chosen: Go 1.25+ with Zero Dependencies

This is out-of-step with the Go ecosystem's dependency norms but correct for a supply-chain tool. The scanner cannot be compromised through a transitive dependency of its own. The cost is reinventing some wheels (no YAML library — the pnpm lockfile parser is a hand-rolled line parser; no TOML — Codex configs are explicitly not parsed).

### Chosen: NDJSON Over Structured Formats

Records are newline-delimited JSON, not a JSON array, not protobuf, not CSV. NDJSON is streamable (the HTTP sink can batch and flush mid-scan), human-readable for debugging, and compatible with `jq`, `grep`, and log shippers. The cost is larger wire size than binary formats.

## Threat Intelligence Integration

The `threat_intel/` directory contains 12 exposure catalogs covering recent supply-chain campaigns: Mini Shai-Hulud (npm/PyPI), GlassWorm (VS Code extensions), Laravel Lang (Composer), GemStuffer (RubyGems), TrapDoor (cross-ecosystem), and others. These are maintained JSON files generated from public threat-intel reporting, assembled with Perplexity Computer and updated via PRs.

The `tools/osvcatalog/` tool converts OSV (Open Source Vulnerabilities) database snapshots into exposure catalogs. It filters to malicious-package records only (`MAL-` IDs) and skips entries with version ranges (v0.1 only does exact-version matching, which drops ~90% of the OSSF corpus).

This is the pipeline's most opinionated design choice: instead of bundling a vulnerability database or querying OSV at scan time, bumblebee separates threat intel from the scanner entirely. Operators bring their own catalogs. The scanner just does presence checking.

## Comparison With Related Approaches

### vs SBOM (SPDX/CycloneDX)
SBOMs answer "what shipped." An SBOM tells you what a project declares as its dependencies at build time. Bumblebee answers "what's on disk right now" — it reads the actual installed state from lockfiles and package-manager metadata, including transient, system-wide, and editor-extension packages that no SBOM captures.

### vs EDR / osquery
EDR answers "what ran or touched the network." Bumblebee answers "what could run if triggered." It captures installed packages that haven't been executed yet — dormant attacker footholds that EDR would miss until activation. It also captures development tooling (editor extensions, MCP servers, browser extensions) that EDR agents typically don't instrument.

### vs npm audit / pip-audit / dependency scanning
Dependency scanners check a project's declared dependencies against advisory databases. Bumblebee checks EVERYTHING on the endpoint against an operator-supplied catalog — not just what a project's manifest declares. This is the difference between "is my project vulnerable?" and "is there any package on this machine that matches the advisory?"

### vs Grype / Trivy
Container vulnerability scanners check container images. Bumblebee checks developer endpoints — the machines where code is written, not where it runs in production. The threat model is different: a compromised development tool (editor extension, MCP server, browser extension) can exfiltrate source code, API keys, and SSH keys without ever touching a container.

## Notable Engineering Details

- **Symlink-loop protection via device+inode tracking**: The walker uses `dirKey()` (platform-specific, `internal/walk/dirkey_unix.go`) to get `(dev, ino)` for each directory. Seen directories are tracked in a map; duplicates are skipped.

- **Error classification**: `isExpectedAccessError()` distinguishes EACCES (expected on macOS privacy-protected trees) from genuine I/O errors. `isMissingPathError()` distinguishes ENOENT (expected when a default-candidate root doesn't exist on this host) from real problems. These surface at debug/info level rather than warn, keeping fleet pipeline noise low.

- **Yarn Classic + Berry**: The Yarn scanner handles both Classic v1 and Berry (v2+) lockfiles from a single parser. Berry's `__metadata` block is recognized and skipped by detecting the header pattern. Berry protocols (`workspace:`, `patch:`, `portal:`) are not decoded but don't break parsing — version is read from the entry body.

- **pnpm store encoding**: pnpm encodes scoped names with `+` in its store directory layout: `@tanstack+query-core@5.0.0`. The pnpm scanner converts this back to `@tanstack/query-core`, matching npm's normalization.

- **Cask deduplication**: Homebrew casks can have `.internal.json`, `.json`, and `.rb` marker files in their metadata directory. The scanner's `IsCaskMetadataMarker()` does a tiny sibling check to emit only one record per cask, avoiding duplicates from multiple marker files.

- **Version precedence**: `bumblebee version` resolves from three sources: `-ldflags` override (highest), module version recorded by `go install`, then the in-tree `VERSION` file. This means production builds can be traced to specific revisions.
