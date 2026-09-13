# Rootshell Terminal

rootshell is a free, MIT-licensed terminal emulator for iPhone, iPad, Vision Pro, and Mac by Kit Knox — ~460K lines of Swift in one Xcode project, built on an embedded libghostty fork for Metal rendering. It matters for two reasons: it is the most complete attempt yet to make the *phone* a first-class client for remote development (native SSH with post-quantum crypto, a from-scratch Swift mosh, tmux control mode, a WASI sandbox for your own CLI tools), and it reframes the terminal as a **supervision surface for coding agents** — an on-device "Agent Inbox" that watches Claude Code, Codex, Cursor, Copilot, and friends work and pings you only when they need a human. #tool #project #agents #terminal

---

## Architecture

A **feature-oriented monolith** with satellite processes:

- `rootshell/App` (entry points, `RootShellApp.swift`), `rootshell/Core` (Ghostty wrapper, Terminal, Persistence, Security, Shell), `rootshell/UI`, and 36 modules under `rootshell/Features/` — SSH (54K lines), AIAgent (30K), Mosh (17K), LocalShell, Cloud, Git, AgentAttention, TSSH, Tmux, MCP, Wasm, VPN, VNC, Kubernetes…
- Satellites: `rootshell-helper/` (unsandboxed macOS PTY broker), `rootshellvpn` + `tunnel/` (native VPN host and system extension), `PushNotificationService` (APNs decryption extension), `Packages/RootshellPushKit` (shared push crypto), and `push/` — a Go CLI (`rootshell-notify`), relay protocol spec (`push/PROTOCOL.md`), and Claude Code/Codex hook installer.
- Three product flavors from one project: sandboxed App Store (iOS/visionOS), unsandboxed Mac Catalyst "Standalone" (needed for the helper and system extension), and a China build where the entire AIAgent feature is compiled out (`#if !CHINA_BUILD`).
- Dependencies are ~20 kitknox-owned forks: `ghosttykit-rootshell`, `Citadel-rootshell` (Swift SSH on NIOSSH), `ios_system-rootshell`, `libgit2-rootshell`, `helix-rootshell`, `bat`, `ripgrep`, `vim`, `curl`, `libarchive`, `jq`, `trzsz-ssh-rootshell` (Go), `yubikit-swift`… The session layer (`SSHTerminalSession` and friends) unifies SSH (NIOSSH + Citadel), mosh, tssh, local shells, and tmux `-CC` gateways behind one PTY surface abstraction.

## Key techniques

**On-device agent detection by screen scraping.** `Features/AgentAttention/` is the headline. Since agents run on remote servers where you can't install anything, detection reads what the terminal already renders: a rules VM ported from herdr (`AgentDetectionManifest.swift`, manifest data a hand-maintained 2,380-line JSON embedded in the binary, overridable via `Documents/AgentDetectionRules.json`) matches a 40-row snapshot of the pane with `contains`/`regex`/`lineRegex` gates composed through `all`/`any`/`not`. The hard-won details:

- **Wrap tolerance** — needles match a whitespace-collapsed form of the region, so a rule survives TUI soft-wrapping that differs between narrow and wide terminals.
- **Multiplexer chrome peeling** — zellij's `│` and tmux's `┃`/`║` pane borders are stripped (with a fixed 4-row allowance for frame corners and status bars) so line-anchored rules work inside panes; the comments cite field captures where a stale identity persisted through an agent change because of this.
- **Alt-screen ownership reasoning** — a TUI banner is trustworthy only when the alternate screen is owned by the agent, not by a multiplexer; `❯ opencode` typed in a shell on a herdr screen otherwise minted a false opencode session.
- **Event-driven cost control** — Ghostty emits one coalesced content edge per changed surface and only that pane is rescanned; the settings toggle is a hard kill switch, and idle means no timers at all.

**Native mosh (SSP).** `Features/Mosh/` is a genuine Swift port of mosh's State Synchronization Protocol: `MoshStateSync` drives an `OutboundSynchronizer`/`InboundAssembler` pair over a UDP transport with OCB encryption using Apple's hardware AES (`MoshOCBCryptor`), an RTT estimator, predictive local-echo overlays, framebuffer diffing via `VTDisplayRenderer`, tick coalescing, and a throttle that drops unfocused tabs from 250ms to 5s ticks. STUN hole-punching handles NATs; sessions survive app termination and reboots.

**Post-quantum push notifications.** The relay is stateless: the device mints per-computer sender credentials, and every notification is sealed with HPKE (RFC 9180) using X-Wing (ML-KEM-768 + X25519), the event id bound as AAD so envelopes can't be relabeled or replayed. The Go side (`push/hook/`) flattens markdown, strips URLs, and redacts secrets before anything leaves the machine; hooks report Claude Code permission prompts and Codex turns without sending prompts, transcripts, or env vars.

**SSH stack.** Citadel/NIOSwift plus globally-registered custom algorithms (`SSHCustomAlgorithms.swift`): `sntrup761x25519-sha512`, `mlkem768x25519-sha256` (gated on iOS 26 crypto availability), ML-DSA host/user keys, and Apple FIDO2 `sk-ecdsa` via AuthenticationServices. Keys can live in the Secure Enclave; MPTCP keeps WiFi+cellular subflows alive for handover.

**WASI sandbox.** `Features/Wasm/` runs user-compiled `.wasm` (WASI Preview 1) on device with a path sandbox that ships its own self-test (`WasmPathSandboxSelfTest.swift`) and a host socket ABI for TCP/UDP/TLS/DNS.

**macOS helper.** The Standalone build can't spawn PTYs sandboxed, so `rootshell-helper` (separate background app) validates callers via `LOCAL_PEERTOKEN` code-signature checks against the app's bundle id and team *before* decoding any command, then passes PTY file descriptors over Unix sockets in a shared App Group container.

**AI agent.** A single tool-using agent (`AIAgentSession`) across Anthropic/OpenAI/Google/Bedrock/OpenRouter with streaming, a thinking parser, batched tool calls with approval gates, and delimiter-based tool-call parsers (`TextToolCallParser`, `<|tool_call_begin|>` markers) for models without native tool calling. Context is a `HostFingerprint` (OS, arch, shell, sudo, cwd) plus terminal scrollback; `lastPromptTokens` is tracked as a context-window-fill proxy. The voice agent is bidirectional audio over a Gemini WebSocket.

## Design decisions

- **Scraping over instrumentation.** Detection could have required agents to emit OSC sequences (cmux's approach) or read harness session stores (Cargento's), but that fails for remote SSH where you control nothing. rootshell accepts a fragile, hand-maintained rules manifest in exchange for working everywhere — then invests heavily in making the fragility survivable: wrap-collapsed matching, chrome peeling, capture-replay debugging, forward-tolerant manifest loading.
- **Fork everything.** Every third-party dependency is a kitknox fork. That buys iOS build fixes in one place and no upstream veto, at the cost of a large maintenance and supply-chain surface concentrated on one maintainer, and it makes the README's "native SSH… no external dependencies" claim misleading — the SSH client is Swift, but Citadel and NIOSSH are external packages.
- **Pragmatic language mixing.** QUIC/KCP for tssh come from the Go trzsz-ssh via gomobile bindings behind a call gate with coalescing latches (`TSSHGoTransport.swift`), rather than a native rewrite. The loose edge is visible: `SSHSession` and `CitadelSSHSession` are two parallel SSH implementations living side by side.
- **Trust the platform, verify peers.** Secure Enclave keys, biometric-gated signing, code-signature checks on every helper and agent-server client — but a "YOLO" auto-approve mode exists in the MCP server's session settings, which is a striking name for the footgun it is.
- **Breadth over focus.** Terminal + SSH + mosh + VPN + VNC + git + editor + AI + WASM + Kubernetes in one free app with no monetization is an enormous surface for one person; the loose seams (dual implementations, commented-out imports, one-commit squashed history) are the price.

## Comparison notes

- Unlike [[cmux]] — the macOS-native libghostty terminal for managing agent sessions — rootshell pushes the same "terminal as agent control room" idea to iOS/Vision Pro and can't rely on agents emitting OSC markers, so it scrapes the screen with a rules VM instead of asking the agent for anything. cmux consumes libghostty; rootshell maintains a fork.
- It solves the same problem as [[Cargento (Agent Cartography Dashboard)]] — reconstructing working/idle/needs-input state without an SDK — but Cargento reads harness session stores on the desktop while rootshell reads the rendered pane, which makes it work for remote hosts and inside tmux/zellij at the cost of a 2,400-line hand-tuned manifest.
- Where [[Orca]] orchestrates 20+ agents in worktrees from a desktop daemon with a mobile *companion*, rootshell inverts the topology: the phone is the primary client, the terminal is the app, and the only server-side piece is a stateless encrypted push relay.
- The remote-access story overlaps [[Tessera]]'s consent-gated, human-approved tunnels: rootshell instead hardens the transport itself (post-quantum KEX, Secure Enclave keys, E2EE notifications), treating the phone's terminal as the privileged surface rather than brokering access through a coordinator.

---

*Sources: [[raw/rootshell]], [[summary/rootshell]]*
*Last updated: 2026-09-13*
