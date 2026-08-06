# Bumblebee

A read-only endpoint inventory collector from Perplexity AI that answers the critical supply-chain response question: when an advisory names a compromised package, extension, or version, which developer machines show a match in their on-disk metadata right now? A single static Go binary with zero non-stdlib dependencies that reads lockfiles, package-manager install metadata, extension manifests, and MCP configs — without ever executing a package manager.

---

## Architecture

Bumblebee is a single Go binary (v0.1.1, ~3,600 lines of core source) built on a pipeline architecture: resolve roots → walk filesystem (filename-dispatch) → parse ecosystem-specific metadata → emit NDJSON → optionally match against an operator-supplied exposure catalog.

The key structural insight is that **the walker visits directories but only opens files matching exact filename patterns**. It never descends into `node_modules` subtrees — individual ecosystem scanners walk their own bounded depth. This avoids the combinatorial explosion that makes naive filesystem walks useless on developer machines.

```
CLI (cmd/bumblebee/main.go:449)
  → Resolve Roots (cmd/bumblebee/roots.go:617)
    → Walk Filesystem (internal/walk/walk.go:266)
      → Dispatch to 13 Ecosystem Scanners (internal/scanner/scanner.go:562)
        → Emit NDJSON (internal/output/output.go:162)
          → Optional: Match Exposure Catalog (internal/exposure/exposure.go:330)
```

**Zero dependencies** (`go.mod` contains only the module path and `go 1.25`). This is not dogma — it's supply-chain hygiene. A security scanner that pulls in 200 transitive dependencies defeats its own purpose.

### Three Scan Profiles

| Profile | What It Scans | Cadence |
|---|---|---|
| `baseline` | Global/user package roots, toolchains, editor/browser extensions, MCP configs | Recurring (every 6h via cron/launchd) |
| `project` | Configured dev directories (`~/code`, `~/src`, `~/Developer`) | Daily |
| `deep` | Explicit `--root` paths including `$HOME` | On-demand incident response |

`baseline` and `project` refuse bare-home roots. Only `deep` accepts them. This is a deliberate safety gate — `baseline` is a curated allowlist, not a filesystem crawl.

### Ecosystems Covered

13 ecosystems: **npm** (lockfiles + `node_modules`), **pnpm** (lockfile + store layout), **Yarn** (Classic + Berry), **Bun** (text lockfile), **PyPI** (dist-info + egg-info), **Go** (`go.sum` + `go.mod`), **RubyGems** (`Gemfile.lock` + `.gemspec`), **Composer** (`composer.lock` + `installed.json`), **MCP** (JSON configs across 7 client types), **agent skills** (`skills.sh` lock files), **editor extensions** (VS Code, Cursor, Windsurf, VSCodium), **browser extensions** (Chromium-family + Firefox), **Homebrew** (formula receipts + cask metadata).

## Key Techniques

### Exclude-Driven Safety, Not Include-Driven Precision

The walker's `DefaultExcludes` list (`[[internal/walk/walk.go]]` lines 30-162) is the project's most security-conscious code — 130+ excluded directory patterns covering credential dirs (`.ssh`, `.gnupg`, `.aws`, `.kube`), browser profiles, macOS TCC-protected trees, email, messages, keychains, and media libraries. These apply even when an operator passes `--root "$HOME"`. Exclude matching uses both basename (`node_modules/.cache`) and suffix-component (`Library/Caches`) patterns.

### MCP Config Parsing as Supply-Chain Surface

The MCP scanner (`internal/ecosystem/mcp/mcp.go`, 853 lines) is the largest single source file, reflecting the fact that MCP configs are a novel attack surface conventional scanners don't cover. It handles three envelope shapes, infers package identity from command/args (`inferPackageFromArgs()`), splits Docker image refs into name+version (correctly handling registry ports), and aggressively sanitizes credentials: drops all `env` values, reduces remote URLs to `scheme://host`, rejects unresolved shell variables, and validates that extracted package specs aren't secretly URLs or file paths.

### Content-Addressed Record Identity

`record_id` is computed via `sha256(recordType + "\x00" + joinWithUnitSeparator(canonicalFields))` using ASCII unit separator (`\x1e`) as delimiter. This means records are stable across runs — the same package on the same host produces the same `record_id` — and deduplication is deterministic without trusting `run_id`. Receivers can join across runs on `record_id`.

### npm Lifecycle Script Detection Without Execution

The npm scanner detects packages with lifecycle scripts (`install`, `postinstall`, `preinstall`, `prepare`) from lockfile metadata. It records script names as `lifecycle_scripts` but never reads script bodies. Responders get the signal (which packages could execute code on install) without the risk.

### Browser Extension Coverage Without Reading Profile Data

Only `Extensions/<id>/<version>/manifest.json` for Chromium-family and `extensions.json` for Firefox are ever opened. Profile data (cookies, login data, IndexedDB, local storage) is excluded through curated root paths and default excludes. On macOS, the entire browser app-support tree is excluded from deep walks.

### Embedded Self-Test

`bumblebee selftest` runs the full scanner pipeline against embedded fixtures (`//go:embed selftest/fixtures`) with deliberately fake package names. Expects exactly 5 findings against an embedded catalog. Zero network calls, runs in ~1ms — a pre-deployment smoke test for fleet rollouts.

## Design Decisions

**Snapshot-only, not delta**: Each run emits a complete inventory and exits. No endpoint-side state, no diff against previous runs. The receiver derives current state from complete snapshots. This avoids bad deltas after missed runs or parser changes. The cost is more data per run.

**Slim schema, not rich metadata**: v0.1 deliberately excludes resolved URLs, integrity hashes, dependency graphs, and `description` bodies. The `model.Record` has 16 fields — enough for exact-match exposure checking, not enough for comprehensive SBOM generation. `confidence` (high/medium/low) compresses evidence strength into a single field.

**Depth over breadth**: Notable omissions include Cargo, Maven/Gradle, NuGet, Hex, Swift PM, Yarn PnP, and Bun binary lockfiles. The authors prioritized deep parsing of npm/PyPI/Go/RubyGems/Composer metadata formats over shallow coverage of more ecosystems.

**One profile per invocation**: You can't run `baseline` and `project` in a single scan. This separates populations cleanly — `baseline` runs never retire packages from `project` inventory.

**Operator-supplied threat intel, not bundled**: Bumblebee ships no built-in advisory feed. The `threat_intel/` directory provides 12 maintained exposure catalogs for recent campaigns, generated from public threat-intelligence reporting. The scanner just does presence checking against whatever catalogs the operator provides. This is the opposite of bundling a vulnerability database.

## Comparison Notes

Unlike **SBOM tools** (SPDX/CycloneDX) which answer "what shipped," bumblebee answers "what's on disk right now" — including transient, system-wide, and tooling packages no SBOM captures.

Unlike **EDR/osquery** which answer "what ran or touched the network," bumblebee answers "what could run if triggered" — dormant attacker footholds EDR would miss until activation.

Unlike **Grype/Trivy** which scan container images, bumblebee scans developer endpoints — the machines where code is written. A compromised editor extension or MCP server can exfiltrate source code without touching a container.

Unlike **npm audit/pip-audit** which check project manifests against advisory DBs, bumblebee checks EVERYTHING on the endpoint against an operator-supplied catalog — beyond what any single project declares.

The closest architectural relative in the wiki is [[Supply Chain Security for Software Developers]], which recommends pinning versions and disabling install scripts — bumblebee operationalizes that advice at fleet scale by detecting exactly which endpoints have pinned the wrong version. [[Ship Safe]] covers the complementary angle: scanning the project source itself for supply-chain threats (slopsquatting, dependency confusion, risky install scripts, unpinned AI actions) rather than the endpoint's installed inventory.

---
*Sources: [[summary/bumblebee]]*
*Last updated: 2026-07-05*
