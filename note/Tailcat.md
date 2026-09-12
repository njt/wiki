# Tailcat

Tailcat is Tailscale's data plane stripped of its control plane: a netcat-like pipe that gives you point-to-point WireGuard-encrypted tunnels with NAT traversal, packaged as a Go library plus a CLI. One side prints a short `tc…` address; the other dials it. No account, no coordination server, no root, no changes to routing tables or DNS. It matters because it isolates the two halves of Tailscale that are usually fused — encryption + hole-punching (the hard, reusable part) from identity, ACLs, and coordination (the part you might not want) — and proves the data plane alone is a useful product.

---

## Architecture

One Go module, `github.com/tailscale/tailcat`, built on `tailscale.com` as a normal library dependency (not a fork). Three layers:

- **Library** — `tailcat.go` (2,423 lines) is the heart. `Server` and `Client` are the two public entry points; both have usable zero values. The `locoBackend` struct (tailcat.go:328) is described in its own comment as "like tailscaled's LocalBackend, but crazier … there's no controlclient involved, because there's no control plane." It assembles the reused Tailscale subsystems — `tsd.System`, `wgengine` (userspace WireGuard), `netstack` (gVisor TCP/IP in-process), `magicsock` (STUN + UDP hole-punching), and `derp` (relay) — into a hub-of-the-world object.
- **CLI** — `cmd/tailcat/tailcat.go` (1,858 lines) is a `ff` command tree: `serve`, `ping`, `socks`, `recv`, `ssh`, `cp`, `ls`, `forward`, `parse`, `resolve`, `genkey`, `readme`. Each subcommand is a thin shim over the library's `OnTCP`/`OnUDP`/`OnTCPForward` callbacks.
- **WASM** — `web/` compiles the library to js/wasm; the browser peer is DERP-only (no UDP in a browser, so no direct connections — WebRTC is tracked as issue #4).

The **tailcat address** is the core abstraction. `ConnInfo` (tailcat.go:154) holds four things, CBOR-encoded then base64url-ed behind a `tc` prefix:

1. `ServerPublic` — the server's WireGuard node key (Curve25519).
2. `ServerDiscoPublic` — a *separate* path-discovery key (see below).
3. `PresharedKey` — an independent 256-bit WireGuard pre-shared key.
4. `Region` (embedded DERP node metadata, long but self-contained) or `RegionID` (an integer into the default DERP map, short but requires a fetch).

`wire.go` defines the wire types with single-character CBOR field names (`p`, `k`, `q`, `r`, `i`, `N`, `n`, `h`, `t`, `4`, `6`, `s`, `d`, `x`) so a typical address is ~140 bytes; a test (`TestWireFieldNames`) locks the field names as a wire format, keeping it independent of upstream `tailcfg` churn.

## Key techniques

- **The "meow" handshake replaces the control plane's endpoint distribution.** A client joining sends a `meow` ping (magic `'m','e','o','w'` + type byte + node key + disco key, `disco.go`) over raw DERP. The server's `onMeow` adds the client to its WireGuard peer list and network map, then replies `meowed`. Because DERP drops packets to not-yet-connected keys, the client resends every second rather than betting one packet on a timeout. Separately, `advertiseEndpoints` (tailcat.go:1435) hand-frames Tailscale `disco.CallMeMaybe` messages and sends them over DERP whenever local UDP endpoints change — reimplementing, by hand, the endpoint exchange the control plane normally performs. This is the single most surprising move: they didn't build a minimal control plane, they substituted a two-message handshake and let magicsock's disco machinery do the rest.

- **A separate disco key, derived by HMAC.** `discoPrivateForNode` (tailcat.go:2406) derives the path-discovery key from the node key via `HMAC-SHA256(nodeKey, "github.com/tailscale/tailcat disco key v1")` with Curve25519 clamping. Rationale: disco frames carry the disco public key in cleartext on direct UDP paths, while knowledge of the *node* public key grants access to an open server — so the two keys must be unlinkable.

- **The pre-shared key makes the address a bearer capability.** The 256-bit PSK is baked into the address, so sharing the address is the entire access-control story. It provides post-quantum confidentiality against recorded traffic and stops a DERP operator who observes both peers' public keys from joining the tunnel. `--psk=false` exists only for v0.5.0 compatibility and shortens the address by weakening it.

- **Deterministic IPv6 addressing.** `tcAddrForKey` (tailcat.go:1655) maps a node key into Tailscale's ULA range `fd7a:115c:a1e0::/48`, filling the low 80 bits from the key. No address assignment, no DHCP — the peer's IP *is* a hash of its identity, which also makes the SSH server's `peerKeyForSession` trivially reverse the peer's key from its source address.

- **Userspace TCP teardown, done obsessively.** Because the entire TCP stack runs in-process (gVisor), exiting right after `Close()` can discard a FIN or ACK that never got transmitted. `DrainTCP`, `closeProxyConnTimeout`, and `ProxyConns`'s `CloseWrite` half-close dance (tailcat.go:2191+) all exist to flush FINs before process exit. Two helpers reach gVisor internals via `reflect` + `unsafe` (`tcpipStackOf`, `gonetTCPConnInternals`) with TODOs to add upstream accessors — a senior engineer will wince at the cheat, but it's the price of correct close semantics without a kernel.

- **`os.Root` confinement for SFTP.** `tailcat_sftp.go` roots every client path in `os.OpenRoot`, so neither `..` nor symlinks escape. The write-only drop-box modes (`FileServeWO`, `FileServeWOPlus`) go further: uploads get server-chosen unique names (`stem.UTC-timestamp.random.ext`), and the `stat` policy (tailcat_sftp.go:374) reports `NoSuchFile` for anything the session didn't itself create, so a sender can't enumerate or even probe the existence of existing files.

- **Exit-node IPv4 rides a NAT64 prefix.** The tunnel is IPv6-only, so `DialTCP`/`DialUDP` map IPv4 destinations into `64:ff9b::/96` (tailcat.go:2113) and the server's forward handlers unmap them — a clean way to carry IPv4 through a v6-only overlay.

- **DERP-map caching with a freshness policy.** `DERPMapCache` is an interface the CLI implements on disk (`~/.cache/tailcat`); younger-than-an-hour maps are used cold, older ones revalidated with `If-None-Match`, and any stored map serves as fallback if the fetch fails (10s timeout). The Tailcat-Mode header tells the map server whether a server or client is asking.

- **Computed build tags, not a hand-edited list.** `internal/buildtags` derives the `ts_omit_*` allowlist from `tailscale.com/feature/featuretags` — keep a handful of features (`netstack`, `ssh`, `gro`, `bakedroots`), expand their dependencies, omit everything else — shrinking binaries ~16% (the wasm build drops ~18%). A sync test fails if `build-tags.txt` or `.goreleaser.yaml` drift from the computed source of truth.

## Design decisions

**Optimized for ephemeral, zero-trust use.** The default is a fresh key generated in memory, an address nobody has ever seen, dead the moment the process exits. Saved keys (`genkey`) are the opt-in; the magic name `default` silently becomes the persistent identity once it exists, and the startup line tells you which happened.

**The address is the whole security model.** No usernames, no ACLs beyond an optional `--allow` allowlist of node keys. That's both the elegant part and the limitation: it's a point-to-point primitive, not a mesh — one server, N clients, no client-to-client routing, no revocation beyond never sharing the address again.

**Short vs. self-contained is an explicit user choice.** `--full-address` / `resolve` embed DERP metadata so clients skip a map fetch; the default integer region ID keeps addresses short but costs a round-trip. `--fixed-region` exists so a DNS-published address stays valid across restarts.

**No stability promises, loudly.** Go API, CLI flags/output, and wire format may all change; the public DERP relays have no SLA. The `readme` subcommand embeds the README into the binary (via `readme.go`) specifically so people — and AI agents — with only a binary can learn to use it without web access.

## Comparison notes

The obvious frame is **[[Tailscale]]** itself: tailcat is the data plane (magicsock + WireGuard + netstack + DERP) with the control plane deleted, and it uses Tailscale as a *library* rather than a fork.

Against **[[Headscale]]**, the contrast is sharp: Headscale *replaces* the coordination server with a self-hosted one so official clients keep working; tailcat *removes* the coordination layer entirely and moves identity into the bearer-capability address. Headscale keeps the tailnet's multi-node, ACL'd, named-identity model; tailcat gives up all of that for a single encrypted pipe with no server at all.

**[[Tunnet]]** is the closest modern sibling — it bundles mesh, serve, tunnel, send, and SSH under one identity system and policy engine, with an optional control plane. Tailcat is the opposite pole on the same axis: no identity system, no policy engine, no membership database, just an address that *is* the credential. Where Tunnet's Direct mode still needs CRDT membership and a pre-shared-key challenge-response to join a network, tailcat's join protocol is one ping and one ack.

**[[RustDesk]]** shares the NAT-traversal-plus-relay-fallback shape (race direct paths, fall back to an encrypted relay), but applies it to remote desktop; tailcat is the same trick generalized into a netcat — stdin/stdout, ports, UDP, files, SSH.

**[[Agentcookie]]** and **[[Domenic Denicola's Agentic Coding Setup]]** are consumers of the *full* Tailscale stack (tailnet, accounts, coordination). Tailcat is the underlying primitive beneath such tools — the "encrypted pipe to a machine" without the account ceremony — which is why it deliberately notes its appeal to AI agents in the `readme` embedding.

#tool #project #networking #security #vpn #p2p

---
*Sources: [[raw/tailcat]], [[summary/tailcat]]*
*Last updated: 2026-09-04*
