# rdc — Remote Desktop Control for AI Agents

rdc is a single Rust binary that lets a coding agent see and operate another computer's desktop over a Tailscale tailnet — screenshots, mouse, keyboard, window focus, clipboard — and exposes that surface both as a CLI and as a stdio MCP server. What makes it worth study is not the desktop control (VNC did that decades ago) but the authentication architecture: no passwords, tokens or certificates anywhere. Identity comes from the network layer — the daemon resolves each peer IP through tailscaled's `whois` and matches it against a capability-scoped allowlist — so the tailnet itself is the security boundary. It is also a deliberately small tool: no shell, no file transfer, no agent loop, just a screen-and-input surface with an audit trail.

#tool #project #agents #security #mcp #rust

---

## Architecture

One crate, one binary, three roles chosen by subcommand (`main.rs`):

- **`rdc serve`** — the daemon on the machine being controlled. An axum server (`server/mod.rs`) exposing `/v1/state`, `/v1/screenshot`, `/v1/act`, `/v1/clipboard`, `/v1/whoami` behind a single auth middleware (`server/auth.rs`).
- **`rdc mcp`** — a stdio MCP server for the agent's machine, built on `rmcp` with its `#[tool]` macros (`mcp.rs`). Twelve tools: `screenshot`, `displays`, `windows`, `focus`, `mouse_move`, `click`, `drag`, `scroll`, `type`, `key`, `clipboard_get`, `clipboard_set`.
- **CLI** — the same operations as subcommands (`rdc -t studio-mac click 640 400`), where coordinates are desktop points rather than image pixels.

The load-bearing abstraction is the `Desktop` trait (`desktop/mod.rs`): `displays()`, `screenshot()`, `windows()`, `focus()`, `input()`, `clipboard_get/set()`. It has exactly two implementations — `LocalDesktop` (in-process: xcap for capture, enigo for input, arboard for clipboard, plus platform window modules `win_hypr`/`win_mac`/`win_windows` behind `cfg(target_os)`) and `RemoteDesktop` (an HTTP client in `desktop/remote.rs` that talks to a remote daemon). The CLI and MCP server only ever hold an `Arc<dyn Desktop>`, so `--target local` and `--target studio-mac` run identical code. `AGENTS.md` calls this the "one seam" rule: every capability lands in the trait first, then the wire API, then the CLI and MCP tools — nothing is reachable from one surface that isn't from the others.

On the wire, screenshots carry the desktop rectangle they cover (`x-rdc-rect`, `x-rdc-size` headers), so clients can map pixels back to logical points regardless of scaling or downscaling. A `rdc doctor` subcommand checks readiness (tailscaled reachable, OS permissions, displays), and `rdc service install` installs the daemon as a LaunchAgent (macOS, deliberately not a LaunchDaemon — it must run in the GUI session), a systemd `--user` unit (Linux), or a scheduled task (Windows).

## Key techniques

**Identity from the network, never the request.** `tailscale.rs` resolves the peer IP via tailscaled's LocalAPI `whois` (unix socket / named pipe / macOS loopback+token), with a `tailscale whois --json` CLI fallback that mirrors the response shape. `server/auth.rs` caches the IP→identity mapping for 30 seconds. Headers, bodies and query strings carry no identity at all — there is nothing to forge.

**Tagged devices are their tags.** Tailscale reports the *creating user's* profile for tagged nodes. `tailscale.rs`'s `identity()` deliberately drops that login when tags are present, so a server tagged by an allowed user does not inherit that user's desktop access. `AGENTS.md` flags this as a security invariant: "Do not reintroduce it."

**Bind-only-Tailscale enforced in code.** `server::is_tailscale_ip` accepts only 100.64.0.0/10 (the CGNAT range) or `fd7a:115c:a1e0::/48`; the daemon refuses any other bind address. An empty allowlist refuses to start. The single exception is `--dev-loopback`, which is loopback-only and unauthenticated by construction.

**Host-header allowlist as the DNS-rebinding defense.** `auth.rs` checks that the `Host` header names this machine (its Tailscale IPs, MagicDNS name, hostname, plus config extras) *before* authorization, answering anything else with 421 MISDIRECTED_REQUEST. This is the one line of defense against a browser on an allowed node being lured at the daemon.

**Capability-scoped grants.** `config.rs` parses `allow = ["alice@example.com", { who = "monitor-bot", can = "view" }]` into `ResolvedGrant`s; matching grants union their capabilities (`view`, `input`, `clipboard`). Every route is wrapped in an `audited()` helper that checks the capability, times the call, and writes the audit line — including denials and validation failures.

**A JSON-lines audit trail that resists its own log reader.** `server/audit.rs` writes one line per request or rejection: identity, method, path, an action summary ("input.click 640,400 Left x1"), outcome, status, duration. It never records typed text or clipboard contents, and `sanitize()` replaces control characters (terminal escapes, newlines) and truncates to 512 chars — so a hostile key chord or window title can neither break JSON-lines framing nor attack the terminal of whoever runs `rdc audit`. Rotation by size with N kept files.

**Screenshot-space coordinates.** `view.rs`'s `ViewMap` converts image pixels → logical desktop points. It clamps NaN/infinite inputs, and maps the far edge half-open so an edge pixel never lands on a neighbouring display (there's a test for exactly that). The MCP tools take pixels of the last screenshot — what a vision model naturally reads off the image — so the agent never needs to know about display scaling or multi-monitor offsets.

**Verify-after-action as the default.** Every MCP action returns a fresh screenshot (after a 350ms settle) unless the agent passes `then_screenshot: false`. Tool calls serialize behind a `tokio::sync::Mutex` (`ops`) so a burst of calls keeps its ordering and view mapping. `windows` reports each window's rect both in desktop points and, once a screenshot exists, in image pixels — so the agent can click a window without guessing.

**Input and clipboard on dedicated OS threads.** `desktop/local/input.rs` owns the enigo handle on one thread and arboard on another, fed over channels, because neither is happily shared across threads on every platform. A clipboard operation times out after 5s, so a hung Wayland clipboard owner (Wayland transfers have no deadline) can never block mouse and keyboard.

**Pinning enigo's Linux backends by sabotage.** enigo opens every compiled-in Linux backend and broadcasts events to all of them; under Wayland that would also drive Xwayland with the wrong coordinate space. rdc pins exactly one backend by pointing the other at an endpoint that cannot exist — `x11_display = ":rdc-disabled"` (unparseable as a display name) or a wayland socket at `/nonexistent/rdc-disabled/wayland.sock` — and filters enigo's error logs.

**Shelling out to `screencapture` for speed.** On recent macOS, `CGWindowListCreateImage` is shimmed through ScreenCaptureKit and takes seconds per call; `capture.rs` instead invokes `/usr/sbin/screencapture -x -D <n>` into a temp PNG (fast, SCK-direct), falling back to xcap. Multi-monitor captures are composited at the primary display's density, downscaled to the requested long edge (default 1568px, Triangle filter), and encoded as fast-compression PNG or JPEG 85.

**Wayland honesty.** xcap cannot see Wayland-native windows and no cross-compositor focus API exists, so on Hyprland, `win_hypr.rs` drives `hyprctl -j` for window listing and focus. GNOME/KDE Wayland is capture-only until the RemoteDesktop portal is wired up — stated plainly in the README's status table rather than papered over.

**Fail-closed config.** `deny_unknown_fields` on every config struct, malformed grants fail at load, unknown capability names are errors, and since 0.3.0 `config::enforce_permissions` refuses to run if `config.toml` or its directory is owned by another user or writable by group/others — because editing the allowlist is equivalent to desktop access (escape hatch: `RDC_INSECURE_CONFIG=1`).

## Design decisions

- **Zero-credential auth, priced in Tailscale.** The elegant part of "your tailnet is the security boundary" is that WireGuard encryption, transport, and identity all come for free. The price: rdc is useless without a tailnet, and its security model silently assumes tailnet hygiene (device hygiene, ACLs) that rdc can only partially check — hence `rdc doctor` and the docs mapping every grant example to a Tailscale policy rule.
- **Pixels over structured APIs.** rdc bets on vision where [[Computer Use is 45x More Expensive Than Structured APIs]] says pixels cost ~45x more than endpoints. It mitigates — 1568px downscale, JPEG, `then_screenshot: false` batching, window rects given in image space — but accepts the structural tax as the price of universality: a native macOS dialog, a BIOS screen, or a DAW has no API. Notably it doesn't even try the accessibility-tree middle path.
- **A surface, not a platform.** No shell, no file transfer, no VMs, no agent loop. This keeps the attack surface tiny, keeps the audit log meaningful (every line is something you'd want reviewed), and resists scope creep toward "a remote access trojan that happens to have an MCP server." The cost is that real workflows straddle rdc and SSH, and the agent has to know which tool to reach for.
- **Fresh screenshots cost tokens, verification buys correctness.** Returning an image after every action is the same verify-over-generate bet the rest of the agent tooling world is making; the default-on design makes the lazy path the safe path.
- **One binary, three roles.** Daemon, CLI, and MCP server in one ~4,900-line executable simplifies deployment to "install the same thing on both machines." GPLv3+; CI builds and tests all three OSes; release binaries unsigned, with signing left to setup docs.
- **Residual trust questions.** The 30s identity cache means a tailnet device that changes hands keeps its predecessor's access for half a minute; input injection into an unlocked desktop session inherits whatever that session can do (the docs say as much — the machine is trusted, the caller is what's checked); and `--dev-loopback` is a permanent, documented footgun kept deliberately loud.

## Comparison notes

Where [[Authenticating MCPs]] catalogs three MCP auth patterns — OAuth SSO, no auth, URL tokens — rdc is a fourth: **network-layer identity**. There is no token to leak, no OAuth dance to break, no credential in the agent's config file at all; the cost is that it only works where every caller is already on the tailnet. It's the same "auth as a function of where you are, not what you present" idea that zero-trust docs gesture at, implemented in ~300 lines against an off-the-shelf daemon (tailscaled).

The pixel-first design is the exact inversion of [[xa11y — Desktop Automation via Accessibility APIs]], which drives desktops through structured accessibility trees to avoid the vision tax. rdc's coordinates work on anything visible — canvas apps, dialogs, machines where no a11y tree exists — while xa11y is precise and cheap where trees are rich and brittle. The two are complements more than rivals: a mature setup would try the tree first and fall back to pixels, which is roughly what rdc's `windows` listing in image space approximates from the other side.

Compared to [[Cua — Computer Use Agent Platform]], rdc is plumbing where Cua is a platform: no VM orchestration, no model adaptation, no agent loop — just the remote screen-and-input seam plus auth. Cua controls disposable macOS VMs; rdc controls *your actual machines*, which is why nearly all of its design effort goes into "who may click" rather than "how does the model click."

And it complicates [[Computer Use is 45x More Expensive Than Structured APIs]]: the 45x gap is real but assumes an API exists. The last mile of machine control — a permission dialog on a headless Mac mini, the exact case rdc names — has no endpoint to call, and a tool like this is what closes the loop when the agent hits it.

---

*Sources: [[raw/rdc]], [[summary/rdc]]*
*Last updated: 2026-09-13*
