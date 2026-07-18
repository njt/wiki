---
url: https://github.com/tunnetio/Tunnet
title: Tunnet — Open-source mesh VPN, serve, tunnel, send, and SSH platform
author: tunnetio
date_fetched: 2026-07-18
date_published: 2025
---

# Tunnet — Architectural Analysis

Tunnet is an open-source platform (AGPL-3.0) that connects machines into a private mesh network with an integrated suite of networking primitives: mesh VPN, internal service exposure (Serve), public tunneling (Tunnel), P2P file transfer (Send), and identity-based SSH. It operates in two modes — Managed (control plane with dashboard, SSO, policies) and Direct (pure P2P with CRDT membership, no server needed) — and supports migration between them.

Written in Rust with a TypeScript management UI, the project is ~26K lines of Rust across six crates plus TypeScript packages for the dashboard, management API, and SDK.

## Architecture

### Crate tree

```
crates/
  tunnet-common/     — shared types: policy engine, wire protocol, IPv6 helpers, relay control, recording format
  tunnet-core/       — core library: node bootstrap, routing table, ACL engine, connection pool, tunnel/serve/send managers, Direct mode membership, stream protocol
  tunnet-agent/      — CLI + system service: TUN data plane, SSH server, DNS, firewall, service management
  tunnet-control/    — control plane server: HTTP API (axum), PostgreSQL, policy store, SSH auth, HA, enrollment
  tunnet-relay/      — public edge relay for tunnels: HTTPS termination, ACME, reverse-tunnel agent acceptor
  tunnet-node-napi/  — Node.js native addon (napi-rs) for the @tunnet/sdk npm package
```

```
apps/
  dashboard/   — React admin dashboard (Vite)
  management/  — management API server (Bun)
  docs/        — documentation site (VitePress)
```

### Network stack

Everything runs over QUIC via the [iroh](https://github.com/n0-computer/iroh) library. Each node has an Ed25519 keypair; the endpoint ID is the hex-encoded verifying key (`AgentIdentity::endpoint_id_hex()` in `crates/tunnet-core/src/identity.rs`). The endpoint binds with per-feature ALPNs:

- `tunnet/stream/1` — general-purpose stream multiplexing (proxy, SSH proxy, SDK connections)
- `tunnet/dgram/1` — datagram tunnel for mesh IP packets
- `tunnet/relay/1` — relay control channel
- `tunnet/direct-auth/2` — Direct mode PSK challenge-response
- `tunnet/recording/1` — SSH session recording cast
- `tunnet/send-blobs/1` — P2P file transfer offers
- `iroh-docs` and `iroh-gossip` — Direct mode membership CRDT

### Two-mode bootstrap (`crates/tunnet-core/src/node.rs`)

`CoreNode::bootstrap()` dispatches to `bootstrap_managed()` or `bootstrap_direct()` based on `PersistedState`:

**Managed mode** (`bootstrap_managed`):
1. Signs and sends a register request to the control plane with hostname, version, and metadata
2. Receives an `EndpointSnapshot` containing all memberships, policies, and routes for the org
3. Builds the routing table from the snapshot via `apply_membership()`
4. Opens a WebSocket to the control plane for real-time updates (snapshots, deltas, policy changes, open-tunnel commands)
5. Falls back to a cached snapshot on register failure (offline bootstrap)
6. Also runs a polling fallback loop as a backup sync mechanism

**Direct mode** (`bootstrap_direct`):
1. Loads all joined Direct networks from persisted state
2. For each network, derives an IPv4 address from the endpoint ID (FNV-1a hash in `100.64.0.0/10` CGNAT space) with a collision counter for birthday collisions
3. Bootstraps an iroh-docs document per network — the membership CRDT
4. Each doc has keys like `peers/<endpoint_id>/hostname`, `peers/<endpoint_id>/ip`, `peers/<endpoint_id>/tags`
5. Changes to the docs document replicate via iroh-gossip; a Direct sync loop watches for membership changes and rebuilds the routing table
6. PSK transport auth: before any app ALPN is accepted, peers must complete an HMAC challenge-response over `tunnet/direct-auth/2`, proving knowledge of that specific network's secret
7. Discovery: peers are seeded from invite coordinator + doc membership, with optional Mainline DHT announce for topic liveness

### Routing table (`crates/tunnet-core/src/routing.rs`)

The `RoutingTable` is the central lookup structure. It uses `Arc<ArcSwap<Tables>>` for lock-free reads — the entire table is atomically swapped on updates. The `Tables` struct holds:

- `by_ip: HashMap<Ipv4Addr, Arc<PeerInfo>>` — primary mesh IP → peer lookup
- `by_network_ip: HashMap<(Uuid, Ipv4Addr), Arc<PeerInfo>>` — per-network IP lookup (for multi-network Direct mode)
- `by_endpoint: HashMap<String, Arc<PeerInfo>>` — endpoint hex → peer
- `by_hostname: HashMap<String, Arc<PeerInfo>>` — hostname → peer (for PeerDNS)
- `subnets: Vec<(Ipv4Net, Arc<PeerInfo>)>` — subnet routes, sorted by prefix length descending (longest-prefix-match)
- `hostname_exact/hostname_wildcards` — hostname route resolution (Serve-like features)
- `exit_node: Option<Arc<PeerInfo>>` — selected exit node

The lookup chain for `lookup_ip()` is: direct peer → subnet LPM → exit node. For each Direct network, `replace_network()` sorts by `join_index` and first-joined wins on IP collisions. The `synthetic_ip_for()` function uses FNV-1a hash of the hostname to produce a stable IP in `100.100.0.0/16` for wildcard hostname route DNS resolution.

### Connection pool with on-demand suspend (`crates/tunnet-core/src/iroh_pool.rs`)

`ConnPool` wraps iroh connection management with an on-demand policy:

- **Managed mode**: `keep_alive = true` — connections stay open
- **Direct mode**: `keep_alive = false` — idle connections auto-close after 120s and reconnect when traffic resumes

Each peer has a `PeerSlot` with states `Connected → Suspended → Reconnecting`. When a packet arrives for a suspended peer, the pool buffers it (up to 64 packets / 1MB), initiates reconnect, and flushes on success. Metrics track reconnect attempts, success/failure, latency, and dropped packets. An idle sweeper runs every 10s to close connections idle past the timeout.

### Data plane (`crates/tunnet-agent/src/dataplane.rs`, `tun_io.rs`)

The data plane is hot-swappable via an IPC command channel (`DataPlaneHandle`). `bring_up()`:
1. Creates a TUN device via `tun-rs`
2. Assigns the mesh IPv4 address
3. Configures system DNS to point at the PeerDNS magic IP (`100.100.100.53`)
4. Installs system routes for remote subnets
5. Spawns the outbound loop: reads packets from TUN, looks up destination in the routing table, applies ACL checks, and sends via iroh datagrams
6. For Direct mode, applies per-network firewall rules

`bring_down()` tears everything down: aborts the outbound task, unapplies system routes, restores DNS, drops the TUN device.

### ACL and policy engine (`crates/tunnet-core/src/acl.rs`, `tunnet-common/src/policy.rs`)

The `AclEngine` evaluates policy rules against a connection context (`EvalCtx`):
- `self_endpoint_hex`, `self_ip`, `self_tags`, `self_network`
- `peer_endpoint_hex`, `peer_ip`, `peer_tags`, `peer_network`
- `dst_port`, `protocol` (TCP/UDP/ICMP/Any)

Rules use selectors (`Any`, `Endpoint`, `Tag`, `Network`, `Cidr`) with priorities and port ranges. The default is deny; when the ACL bundle is stale (poll failed) and has no explicit rules, it falls back to allow — a deliberate fail-open for connectivity.

For Managed mode, the policy bundle is signed by the control plane's Ed25519 key and delivered via WebSocket. For Direct mode, the firewall is user-configured and synced through the iroh-docs membership document. The `FirewallEngine` (`crates/tunnet-core/src/direct/firewall.rs`) adds stateful connection tracking: outbound connections automatically allow return traffic via a conntrack table with protocol-specific timeouts (TCP 300s active, 10s TIME_WAIT; UDP 30s; ICMP 10s).

### Tunnel manager (`crates/tunnet-core/src/tunnel.rs`)

The tunnel feature connects to a relay over iroh via `RELAY_ALPN`, sends a `RelayCtrl::Register` message with tunnel ID/subdomain/auth token, then maintains a control channel with keepalive pings every 20s. Incoming relay-forwarded streams are proxied to localhost. For HTTPS tunnels, the manager parses the HTTP request path to match against redirect rules (path-based routing to different local ports). For TCP tunnels, it reads a `Forward` header with target port and optional target IP.

### Serve manager (`crates/tunnet-core/src/serve.rs`)

Serve binds a TLS or TCP listener on the node's mesh IP. From `bootstrap_managed()`, the Serve manager receives start/stop commands via WebSocket from the control plane. It supports three ACL modes: `all_peers` (open), `machines` (specific endpoint IDs), and `tags` (tag-based). Peer identity is reverse-resolved from the connecting IP through the routing table, and join/leave events with byte counters are reported to the control plane via the WebSocket channel.

### SSH subsystem (`crates/tunnet-agent/src/ssh/`)

The SSH server runs on the mesh IP using the `russh` crate. Key features:
- **Identity-based**: no key distribution — auth is tied to Tunnet identity and organization SSH policies
- **Port NAT**: `ssh_nat.rs` rewrites inbound connections from the external SSH port (typically 22 on mesh) to an internal listen port, and rewrites outbound replies in the TUN loop
- **Session recording**: via `tee.rs`, sessions are recorded to cast files and uploaded to the control plane
- **Session registry**: `SshSessionRegistry` tracks active sessions with kill capability — the control plane can request session termination via `KillSshSession` WebSocket message
- **Re-auth**: SSH policies support `action: check` with configurable re-auth periods
- **SFTP**: built-in via `russh-sftp`

### Send — P2P file transfer (`crates/tunnet-core/src/send.rs`)

Uses `iroh-blobs` for content-addressed file transfer. Files are chunked and verified with BLAKE3. Transfer offers are sent over a dedicated ALPN (`SEND_ALPN`). The consent model supports `auto_accept`, `prompt`, and `deny` modes. Files arrive in a configurable inbox directory. Transfers can target specific peers or tagged groups.

### Control plane (`crates/tunnet-control/`)

The control plane is an axum HTTP server with PostgreSQL:
- `/v1/enroll`, `/v1/register`, `/v1/poll` — device lifecycle
- `/v1/ws` — WebSocket for real-time sync (snapshots, deltas, commands)
- `/v1/tunnels` — tunnel creation and lifecycle
- `/v1/ssh-auth/*` — SSH auth evaluation and verification
- `/v1/ssh-recordings` — session recording upload and playback
- `/v1/relay/*` — relay registration and heartbeat
- Admin API on separate port for internal management

Background tasks: Postgres LISTEN/NOTIFY for cross-instance state sync, stale device eviction (60s), presence sweeping (30s), tunnel TTL expiry (30s), device auto-cleanup (60s). High-availability is supported via `ha.rs` with cross-instance notification.

### Relay (`crates/tunnet-relay/`)

The relay is a standalone binary that:
1. Binds an iroh endpoint with `RELAY_ALPN`
2. Runs an HTTPS server (rustls) on `0.0.0.0:443` with optional ACME
3. Accepts agent reverse-tunnel connections and registers them in `TunnelRegistry`
4. Forwards incoming HTTPS connections to the correct agent by subdomain/sni lookup
5. Reports traffic stats and heartbeats to the control plane
6. Supports TCP port mapping for non-HTTP tunnels

### Policy and signing

The control plane holds a policy signing key (Ed25519). All `PolicyBundle` objects carry a signature. Agents verify this signature before applying policy changes, ensuring agents only accept policy from their own control plane. The `ServiceAuth` mechanism uses a shared secret for internal service-to-service authentication.

## Non-obvious implementation choices

### ArcSwap for lock-free routing table reads

The routing table uses `Arc<ArcSwap<Tables>>` — on the hot path (packet forwarding), reads are lock-free atomic pointer swaps. Rebuilds lock a mutex-protected `BTreeMap<Uuid, NetworkSlice>`, construct a new `Tables` struct, then atomically swap. DNS resolution creates synthetic IPs in a concurrent `DashMap` to avoid blocking the main table rebuild.

### Secret sealing with tiered encryption

Agent secrets (identity key, network PSKs, doc tickets) are stored in `state.enc` with an accompanying `state.enc.meta` file. The `SealTier` enum (`None`, `Machine`, `User`) determines the protection level. On macOS, `Machine` tier uses the Keychain; on Linux it can use the TPM or a machine key. The `AgentSecrets` struct holds the decrypted material in memory; `PersistedState` has `#[serde(skip)]` on secret fields so they never serialize to the plaintext `state.json`.

### iroh-docs as membership database

Direct mode uses iroh-docs (a CRDT document store) as its membership database — not a custom protocol. Each network is one document with keyspaced entries (`peers/<endpoint_id>/hostname`, `meta/name`, etc.). Changes replicate via iroh-gossip, and a sync loop watches `LiveEvent` streams from the docs engine to detect membership changes. This means Direct mode gets eventual consistency and offline operation for free from the CRDT layer.

### HMAC challenge-response with network_id binding

Direct mode PSK auth (`crates/tunnet-core/src/direct/auth.rs`) doesn't just verify knowledge of the secret — it binds the claimed `network_id` into the HMAC proof so the server only checks against that specific network's secret, avoiding a try-all-secrets vulnerability. The `AuthCache` tracks authenticated peers per network, and the `DirectAuthHook` (iroh endpoint hook) blocks non-auth ALPNs until the peer is authenticated for at least one joined network.

### Conntrack firewall with reject vs. drop

The Direct mode firewall (`crates/tunnet-core/src/direct/firewall.rs`) distinguishes between `Deny` (silent drop) and `Reject` (TCP RST / ICMP unreachable). Silent drops cause timeouts; rejects give immediate feedback to applications. The conntrack table uses a `DashMap` for concurrent access with a garbage collection sweep every 10s.

### WebSocket as the primary sync channel with poll fallback

Managed mode uses a persistent WebSocket for real-time updates, but also runs a polling loop (`spawn_poll_fallback`) at a configurable interval (default 30s) as a backup. This dual-path sync means the agent stays current even if the WebSocket disconnects and reconnects. The poll fallback marks the ACL as "stale" on failure, switching to fail-open to preserve connectivity.

### Packet buffering during reconnect

When a Direct mode connection is suspended and a packet arrives, `ConnPool::send_or_buffer()` queues the packet (up to 64 packets / 1MB per peer), initiates reconnection, and flushes on success. If the buffer fills, packets are dropped (with metrics). This means Direct mode peers can appear "always connected" to applications even though connections are on-demand underneath.

### SSH port NAT in the TUN hot path

`ssh_nat::rewrite_outbound()` runs on every outbound packet in the TUN loop, rewriting source ports for SSH connections. This is a deliberately minimal check — it only touches packets from the internal SSH port — so the overhead on non-SSH traffic is a single comparison.

## Design trade-offs

### Rich feature set vs. codebase complexity

Tunnet bundles six major features (mesh, serve, tunnel, send, SSH, relay) into a single codebase. The upside: everything shares one identity system, one policy engine, and one routing table. The downside: ~26K lines of Rust with deep cross-crate coupling. `CoreNode` holds references to everything (`ServeManager`, `TunnelManager`, `SendManager`, `ConnPool`, `RoutingTable`, `AclEngine`), making it the god object of the system.

### iroh dependency vs. bespoke protocol

The project is heavily dependent on the iroh ecosystem (iroh, iroh-gossip, iroh-blobs, iroh-docs). This gives QUIC transport, NAT traversal, content-addressed blobs, and CRDT membership for free. The cost is tight coupling to iroh's API and release cycle, and the complexity of managing iroh endpoint hooks and ALPN negotiation correctly.

### Two modes vs. single architecture

Supporting both Managed and Direct mode means every subsystem (routing, ACL, connection management, membership) has two code paths. The `CoreNode` fields express this: `signed: Option<SignedClient>` (Managed only), `direct_auth: Option<AuthCache>` (Direct only), `direct: HashMap<Uuid, DirectNetworkRuntime>` (empty in Managed). This is cleaner than a trait-based abstraction but means every consumer must handle both cases.

### Fail-open on stale ACL

When the control plane is unreachable and the ACL bundle has no explicit rules, the agent fails open (allows all traffic). This prioritizes connectivity over security — a deliberate choice that mirrors how Tailscale and similar tools handle control plane outages. The `stale` flag in `AclEngine` tracks this state explicitly.

### On-demand connections vs. always-on mesh

Direct mode defaults to on-demand connections (idle peers are disconnected and reconnected when traffic flows). This saves resources for personal/small-group use but means the first packet to a sleeping peer incurs reconnection latency (~5s timeout). Managed mode defaults to keep-alive, appropriate for organizational deployments where connectivity is critical.

## Key techniques summary

| Technique | Where | Why |
|---|---|---|
| `Arc<ArcSwap<T>>` for routing | `routing.rs` | Lock-free reads on the hot packet-forwarding path |
| iroh-docs CRDT for membership | `direct/membership.rs` | Free eventual consistency, offline operation, gossip replication |
| HMAC challenge-response with network_id binding | `direct/auth.rs` | PSK auth without try-all-secrets vulnerability |
| Conntrack firewall with reject/drop distinction | `direct/firewall.rs` | Stateful L4 filtering with immediate feedback on blocked connections |
| WebSocket + poll fallback dual sync | `sync.rs`, `ws_client.rs` | Real-time updates with offline resilience |
| Packet buffering during reconnect | `iroh_pool.rs` | On-demand connections that feel always-on to applications |
| Tiered secret sealing (Keychain/TPM/plaintext) | `secret_store.rs` | Defense in depth for key material, platform-appropriate |
| FNV-1a hash for synthetic DNS IPs | `routing.rs` | Stable, deterministic IP assignment for hostname routes without coordination |
| ALPN-based feature negotiation | `node.rs` | Each feature (stream, datagram, relay, auth) gets its own protocol lane over a single QUIC connection |
| Signed policy bundles | `policy.rs`, `signing_key.rs` | Agents only accept policy from their own control plane |
| Ed25519 identity = endpoint ID | `identity.rs` | Self-certifying: the public key IS the node identifier, no CA needed |

## Comparison to related systems

**vs. Tailscale**: Tailscale uses WireGuard for the data plane with a coordination server for key distribution and ACLs. Tunnet uses QUIC (iroh) instead of WireGuard, bundles more features (tunnel, serve, send, SSH) into one agent, and offers a pure P2P mode with no server dependency. Tailscale's control plane is closed-source; Tunnet's is fully open.

**vs. NetBird**: NetBird also uses WireGuard + a management server. Tunnet's Direct mode (CRDT membership via iroh-docs) has no equivalent in NetBird. Tunnet's relay/tunnel system is more feature-rich (path-based redirects, TCP port mapping, ACME).

**vs. ngrok/Cloudflare Tunnel**: These are SaaS-only. Tunnet's relay is self-hosted and open-source. Tunnet bundles tunneling with mesh networking — a tunnel endpoint can also be a mesh peer.

**vs. ZeroTier**: ZeroTier uses a custom P2P protocol with a root server network. Tunnet's Direct mode is conceptually similar (CRDT-governed membership) but uses iroh's QUIC transport instead of a custom protocol, and Tunnet bundles SSH, file transfer, and serve features that ZeroTier lacks.

**vs. Headscale**: Headscale is an open-source control server for Tailscale clients. Tunnet is a ground-up reimplementation with its own agent, control plane, and relay — not a compatibility layer.

The key architectural difference from all these: Tunnet is a **single identity system with a unified policy engine**, where mesh networking, service exposure, tunneling, file transfer, and SSH all share the same routing table, ACL engine, and connection pool. Most competitors treat these as separate products or bolt them on.

## Version

0.6.0 (Rust workspace, edition 2024, MSRV 1.96). AGPL-3.0 with commercial licensing available.

*Last updated: 2026-07-18*
