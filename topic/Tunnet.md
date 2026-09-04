# Tunnet

An open-source platform that connects machines into a private mesh network, then layers serve, tunnel, file transfer, and identity-based SSH on top — all governed by one identity system and one policy engine. Operates in two modes: Managed (control plane, dashboard, SSO, centralized policies) and Direct (pure P2P with CRDT membership, no server needed). Written in Rust (~26K lines) with a TypeScript management UI. AGPL-3.0.

---

## Architecture

Tunnet is a Rust workspace of six crates (`tunnet-common`, `tunnet-core`, `tunnet-agent`, `tunnet-control`, `tunnet-relay`, `tunnet-node-napi`) plus TypeScript apps for the dashboard, management API, and documentation.

### Network layer

Everything runs over QUIC via the [iroh](https://github.com/n0-computer/iroh) library. Each node has an Ed25519 keypair — the endpoint ID *is* the hex-encoded public key, making identities self-certifying with no CA required. Features negotiate separate ALPN protocol lanes over a single QUIC connection: `tunnet/stream/1` for stream multiplexing, `tunnet/dgram/1` for mesh IP packets, `tunnet/relay/1` for tunnel control, and several more for auth, recording, and file transfer.

### Core abstractions

**`CoreNode`** (`crates/tunnet-core/src/node.rs`) is the central god object — it holds references to the routing table, ACL engine, connection pool, and all feature managers (serve, tunnel, send). It bootstraps in one of two modes:

- **Managed mode**: registers with the control plane, receives a full snapshot of peers/routes/policies, then syncs via a persistent WebSocket with a polling fallback. The connection pool keeps connections alive.

- **Direct mode**: bootstraps an iroh-docs CRDT document per network as the membership database. Peers authenticate with an HMAC challenge-response proving knowledge of a pre-shared key (with network_id bound into the proof to avoid try-all-secrets). Connections are on-demand — idle peers disconnect and reconnect when traffic flows. Discovery uses invite coordinator dial + Mainline DHT announce.

**`RoutingTable`** (`crates/tunnet-core/src/routing.rs`) uses `Arc<ArcSwap<Tables>>` for lock-free reads on the packet-forwarding hot path. Lookup chain: direct peer IP → subnet longest-prefix-match → exit node. Supports multi-network routing with first-joined-wins IP collision resolution and FNV-1a-derived synthetic IPs for hostname route DNS.

**`ConnPool`** (`crates/tunnet-core/src/iroh_pool.rs`) wraps iroh connections with on-demand suspend/resume: when a Direct-mode peer is idle, its connection closes; when traffic arrives, packets are buffered (up to 64 packets / 1MB) while the pool reconnects, then flushed. Managed mode keeps connections alive.

**`AclEngine`** (`crates/tunnet-core/src/acl.rs`) evaluates policy rules (selectors for endpoints, tags, networks, CIDRs with priorities and port ranges) against a context of self and peer identity. Default deny; fails open when the policy bundle is stale and empty — a deliberate connectivity-over-security trade-off.

### Feature stack

- **Mesh**: TUN device (`tun-rs`) + system DNS pointing at `100.100.100.53` (PeerDNS magic IP) + system route installation. Outbound packets traverse routing table lookup → ACL check → iroh datagram send. Inbound packets are validated against the routing table (anti-spoof: source IP must match the peer's assigned mesh IP).

- **Serve**: Binds a TLS or TCP listener on the mesh IP, forwards to localhost. ACL supports `all_peers`, `machines` (specific endpoint IDs), and `tags` modes. Peer join/leave events with byte counters reported to the control plane.

- **Tunnel**: Connects to a relay over iroh, registers with auth token, then proxies relay-forwarded streams to localhost. HTTP path-based redirect rules for multi-port routing. TCP tunnel support with `Forward` headers.

- **Send**: P2P file transfer via `iroh-blobs` with BLAKE3 verification. Consent model: auto-accept, prompt, or deny. Targets specific peers or tagged groups.

- **SSH**: Identity-based via `russh` — no key distribution. Session recording, re-auth enforcement, control-plane-initiated session kill. Port NAT rewrites inbound connections from the external SSH port.

- **Relay**: Self-hosted HTTPS edge with ACME. Accepts agent reverse-tunnel connections, forwards by subdomain/SNI. Reports traffic to the control plane.

### Secrets and sealing

Agent secrets (identity key, network PSKs, doc tickets) are stored encrypted in `state.enc` with tiered protection: platform keychain on macOS, TPM on Linux, plain file otherwise. Plaintext state in `state.json` never contains secrets — those fields are `#[serde(skip)]`.

## Key techniques

- **Lock-free routing**: `ArcSwap` on the routing table means packet forwarding never acquires a lock
- **CRDT membership**: iroh-docs provides free eventual consistency and offline operation in Direct mode
- **On-demand connections**: idle Direct-mode peers suspend; packet buffer + reconnect makes it transparent to applications
- **Dual-path sync**: WebSocket for real-time updates + polling fallback for resilience; ACL fails open when the control plane is unreachable
- **Stateful firewall with conntrack**: outbound connections auto-allow return traffic; distinguish deny (silent drop) from reject (TCP RST)
- **Self-certifying identities**: Ed25519 public key = node ID, no PKI needed
- **Signed policy bundles**: agents verify the control plane's Ed25519 signature before applying any policy change

## Design decisions

**Rich feature set in one codebase**: Everything shares one identity, one policy engine, one routing table. The cost is `CoreNode` as a god object and cross-crate coupling.

**iroh dependency**: QUIC transport, NAT traversal, content-addressed blobs, and CRDT membership come for free. The trade-off is tight coupling to iroh's API and a substantial dependency footprint.

**Two modes, not two products**: Managed and Direct share the same agent binary (`tunnet`), same routing table, same feature managers. Every subsystem has dual code paths. A `tunnet upgrade-to-managed` command migrates networks without losing connectivity.

**Connectivity over security when stale**: If the ACL policy bundle can't be refreshed from the control plane, the agent fails open rather than breaking the network.

## Comparison notes

Unlike [[Tailscale]] (which uses WireGuard + a closed-source coordination server), Tunnet uses QUIC/iroh for transport and is fully open-source including the control plane and relay. Its Direct mode — CRDT membership with no server — has no Tailscale equivalent.

Unlike [[Headscale]] (an open-source control server for Tailscale clients), Tunnet is a ground-up implementation with its own agent and protocol, not a compatibility layer.

Unlike ngrok or Cloudflare Tunnel (SaaS-only), Tunnet's relay is self-hosted. Tunneling is bundled with mesh networking — a tunnel endpoint can also be a mesh peer.

Unlike [[ZeroTier]] (custom P2P protocol with root servers), Tunnet uses standard QUIC and a CRDT document for membership. Tunnet also bundles SSH, file transfer, and serve features that ZeroTier lacks.

Unlike [[Tailcat]] (Tailscale's data plane with the control plane deleted, identity collapsed into a single bearer-capability address), Tunnet keeps a real identity system and policy engine; tailcat has no membership or ACL beyond a node-key allowlist, at the cost of losing mesh routing and multi-party semantics entirely.

The key architectural difference: Tunnet is a **single identity system with a unified policy engine** where mesh, serve, tunnel, send, and SSH all share the same routing table, ACL, and connection pool. Most competitors treat these as separate products or bolt them on.

[[Tailport]] is the minimal counterpoint: instead of building its own mesh it delegates entirely to Tailscale — shelling out to the `tailscale` CLI for serve/funnel and layering only a TUI, a port registry, and a Caddy-edge publish path on top.

#tool #project #networking #vpn #p2p #security

---

*Sources: [[raw/tunnet]]*
*Last updated: 2026-07-18*
