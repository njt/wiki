# Tailport

A single Go binary that turns "expose my local dev server to the tailnet" from a memorized `tailscale serve` invocation into a one-keystroke toggle in a terminal UI. Tailport lists your machine's locally listening TCP ports, derives each port's actual reachability (localhost / LAN / tailnet / public), and drives Tailscale's own CLI to serve, funnel, or unpublish each port. It matters less for what it *adds* — a thin Bubble Tea layer over a CLI that already exists — than for the engineering discipline it applies to a tool with real, irreversible consequences: public internet exposure is never automatic, every destructive edge mutation is verified against a confirm the user actually saw, and the "reachability" model is built on the truth of socket binds rather than on what the user *thinks* a port does.

---

## Architecture

Tailport is a **monolith with a layered internal package structure**, not a service. It never runs a daemon, never scans the network, and installs nothing system-wide: every effect it has is a `tailscale serve`/`funnel` mapping the user explicitly toggled. The only state it owns is a per-port registry in `~/.config/tailport/config.yaml`.

- **`internal/portscan`** (`portscan.go` + `portscan_linux.go`/`portscan_darwin.go`) — the read side. `List()` shells out to `ss -H -t -l -n -p` (Linux) or `lsof -iTCP -sTCP:LISTEN -n -P` (macOS) and parses the output with no external dependency. The core type is `Port{Number, Process, BindScope, BindHost}`.
- **`internal/tsserve`** (`tsserve.go`) — the write side. Wraps the `tailscale` CLI: `On`/`Off` (`serve --bg --http=<port>` / `serve --http=<port> off`), `FunnelOn`/`FunnelOff` (`funnel --bg --yes --https=<port>`), plus `Status()`/`FQDN()`/`PublicURL()`. `FunnelOn` runs under a 15s timeout because `tailscale funnel` hangs indefinitely on the admin-console enablement gate.
- **`internal/config`** (`config.go`) — the persisted port registry and display prefs, written atomically.
- **`internal/caddyedge`** (`caddyedge.go`) — a client for a Caddy edge's admin API, the third exposure path (`p` publish to a custom public hostname). Pure Go on the standard library, every external dependency a struct field for test injection.
- **`internal/statusreport`** (`statusreport.go`) — builds the headless `tailport status` report by calling the *same* functions the TUI reads, so the report can't drift from the interactive list.
- **`internal/selfupdate`, `internal/clip`, `internal/ui`** — self-update, OSC 52 clipboard, and the Bubble Tea TUI.
- **`cmd/tailport`** (`main.go`, `update.go`) — CLI dispatch: `tailport` (TUI), `quickstart`, `status`, `update`. `run()` returns an exit code instead of calling `os.Exit`, so every branch is unit-testable against plain `io.Writer`s.

The TUI (`internal/ui/ui.go`, 7.6k lines) is a Bubble Tea model whose list items are `portItem`s. Each item resolves a 7-state `reach()` classification — localhost, tailnet, LAN, served, funnel, publish, stale, offline — and renders a "moon-phase" fill ramp (🌕/🌔/🌒/🌑 vs ○/◔/◉/●) so exposure level reads at a glance.

## Key techniques

- **Reachability is a property of the bind, not of `serve`** (`portscan.go`'s `classifyBindScope`). A wildcard-bound socket is already reachable by tailnet peers at the IP layer (WireGuard, subject to ACLs); `tailscale serve` is a separate app-layer reverse proxy that only matters for loopback-bound apps. Ports seen on several bind rows (dual-stack `0.0.0.0` + `[::]`, or `127.0.0.1` + `0.0.0.0`) are aggregated to the *widest* scope. The conservative default is deliberate: an unparseable host is classified `LAN`, never `Wildcard`/`Tailnet`, so tailport never over-promises reachability.
- **Dangling-forward detection via address-range filtering.** tailscaled's own serve-proxy listeners bind to tailnet addresses (`100.64.0.0/10`, `fd7a:115c:a1e0::/48`); filtering those out *before* aggregation means a port whose only bind is tailscaled's own socket reads as "not listening" — which is exactly how the UI detects a stale serve mapping ("bound to tailnet, but stale").
- **Schema-resilient JSON parsing.** `tailscale serve status --json` is walked generically (`collectPorts` recurses over `map[string]any`, matching bare-int keys or keys ending `:<port>`) rather than bound to a strict struct, on the grounds that the schema drifts across tailscaled versions across a fleet.
- **Optimistic concurrency against a shared remote config** (`caddyedge.go`). Every Caddy mutation is a read-then-write guarded by `If-Match`/ETag, with a bounded retry budget (3) on HTTP 412 and a `ErrEdgeNoIfMatch` refusal on pre-2.5.2 edges that ignore If-Match — an unconditional delete a concurrent array shift could aim at the wrong route is refused outright.
- **Byte-faithful capture and undo.** Force-purge captures the deleted route's *exact* JSON element bytes (routes can carry fields tailport doesn't model) and re-POSTs those bytes verbatim on undo; `semanticallyEqual` compares normalized JSON with `json.Decoder.UseNumber()` so two distinct integers above 2^53 don't collapse to one float64 and produce a false "restored".
- **Conflict classification ladder.** A refused publish is classified — `OwnedDiffBackend`, `IdHijacked` (our `@id` kept but re-pointed), or `ForeignOverlap` — and only a fully host-disclosable foreign route (every match block exactly `{host:[...]}`) is offered for force-delete, with its full host list disclosed so the user never deletes a catch-all under a confirm that named one hostname. Identity is pinned by an FNV-64a hash of the exact element bytes for id-less routes.
- **Atomic, symlink-safe config writes** (`config.go`). Save writes a 0600 temp file in the target's real directory and renames it over the target, so a bcrypt `auth_hash` is never transiently world-readable and a torn write is impossible. `SaveCaddyDomain`/`SaveCaddyHostname` re-read the file into a `yaml.Node` tree and mutate only one scalar, preserving comments *and* any unknown keys a newer/foreign tailport wrote — avoiding last-writer-wins clobber.
- **Verbatim-only CLI plumbing.** The keybinding legend, help text, and prerequisites are generated from one source (`keyMap.groups()`), so `tailport quickstart` and the in-TUI `?` overlay can't drift.

## Design decisions

- **Safety over convenience.** `serve` is tailnet-only plain HTTP by default; `funnel` and Caddy `publish` are explicit per-port opt-ins behind strong confirms; `:22` is hard-blocked and ships locked; public exposure is never automatic. Plain HTTP is a *deliberate* call: WireGuard already encrypts the tailnet hop, so app-layer TLS would add certificate handling for no confidentiality gain.
- **Delegate, don't reimplement.** There is no Go Tailscale client library here and no WireGuard data plane — tailport shells out to the `tailscale` CLI, which keeps it control-plane-agnostic on the serve path and lets it inherit Tailscale's own permission model (it maps the "operator not set" remedy string to a typed error rather than guessing).
- **Registry edits ≠ exposure edits.** Undo/redo covers only the registry (favorite/label/lock/add), stored as per-port deltas so it never silently reverts bookkeeping the user didn't ask for — and it never flips what's exposed, because serve/funnel have their own keys and confirms.
- **Correctness-at-the-edge over feature breadth.** The Caddy path refuses to rank funnel against publish (mutually exclusive per port); external dual-exposure is surfaced as drift, never silently resolved.

## Comparison notes

Where [[Tunnet]] builds the whole mesh — QUIC transport, CRDT membership, ACL engine, serve, tunnel, SSH — tailport builds *none of it*: it treats Tailscale as the platform and only layers a TUI, a registry, and a publish path on top. The trade is scope for depth: tailport's genuinely novel code is the concurrent-safety and confirm-integrity machinery around Caddy, not the networking.

[[Portless]] solves the adjacent local-dev problem from the *naming* side — replace `localhost:3000` with `https://myapp.localhost` — whereas tailport solves it from the *exposure* side: keep the port number, make it tailnet- (or publicly) reachable. The two are complementary; Portless even integrates Tailscale for LAN sharing.

Against [[Headscale]] (the self-hosted Tailscale control plane), tailport is a pure client: it talks only to the CLI, so its tailnet-`serve` path works against any control plane that CLI supports, while its `funnel` path depends on Tailscale's own HTTPS certs and Funnel node attribute.

#tool #project #networking #tailscale #tui #go

---
*Sources: [[raw/tailport]], [[summary/tailport]]*
*Last updated: 2026-09-04*
