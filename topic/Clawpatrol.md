# Clawpatrol

A transparent network-layer security firewall for AI coding agents. Clawpatrol sits between agents (Claude, Codex) and upstream services, intercepting traffic at the IP layer via WireGuard or Tailscale tunnels. It decodes wire protocols (Postgres, ClickHouse, Kubernetes, SSH, HTTPS) to extract typed facts, runs CEL rules against them (deny `DROP TABLE`, gate `kubectl delete pod` behind human approval), injects real credentials replacing placeholder tokens, and logs everything. Written in Go, single binary, SQLite-backed, HCL-configured, MIT licensed.

---

## Architecture

### Transport: L3 tunneling, not HTTP proxy

Unlike traditional API gateways, Clawpatrol operates at the network layer. Two transport modes:

- **WireGuard**: Embedded `wireguard-go` + gVisor userspace netstack. Gateway listens on UDP :51820. gVisor netstack accepts SYNs to ANY destination IP/port — the dispatcher sees the original 4-tuple. No `iptables` on the gateway host.
- **Tailscale**: Embedded `tsnet` node joins the tailnet in-process. Mints OAuth-derived auth keys for onboarded clients. Funnel exposes join bootstrap on :443.

On Linux, `clawpatrol run -- <cmd>` creates a fresh user+net+mnt namespace per process. The auth key is hidden from agent processes by overlaying an empty tmpfs on `~/.clawpatrol/` before exec.

### Dispatch: port-driven protocol routing

When a TCP flow arrives at the gateway, the destination port drives routing:

| dst port | Handler |
|----------|---------|
| :443 | TLS SNI peek (parse ClientHello bytes) → HostEndpoint lookup → HTTPS family dispatch |
| :5432 | Postgres wire-protocol MITM (SCRAM auth offload + sql rule matching) |
| :53 | DNS VIP responder (return allocated VIP for known hostnames; forward everything else) |
| VIP address | VIP table lookup → endpoint runtime for ssh, clickhouse_native |
| else | direct-IP ConnIndex lookup; falls through to transparent relay |

**Transparent relay** means any traffic no plugin claims is spliced to the real upstream byte-for-byte. The `unknown_host` policy setting (passthrough vs. deny) only applies to unmatched HTTPS SNI.

### Plugin architecture: Terraform-style registry

The config system uses a plugin registry (`config/plugin.go`) where each `(Kind, Type)` pair has a factory, validator, builder, runtime interface, and HCL emitter. Plugins register at `init()` time. Six kinds: endpoint, credential, rule, approver, profile, tunnel.

The loader strips **framework-level attributes** (endpoint binding, placeholder) from block bodies before plugin decode, so plugins don't implement cross-cutting concerns. Adding a new cross-cutting attribute is a one-line entry in `frameworkAttrsByKind`.

### Endpoint plugins: protocol-level MITM

Each endpoint plugin owns wire-protocol decode for one upstream type:

- **https** — Terminates TLS, parses HTTP requests, exposes `http.method`, `http.path`, `http.headers`, `http.body_json` to rules
- **kubernetes** — Parses k8s API URLs into `k8s.verb`, `k8s.resource`, `k8s.namespace`, `k8s.name`. Applies cluster CA cert via `EndpointTLSConfigurer`
- **postgres** — Parses Postgres wire protocol messages (Query/Parse), offloads SCRAM-SHA-256 authentication (gateway IS one peer; agent receives synthesized AuthenticationOk), exposes `sql.*` facts
- **clickhouse_native** — Parses ClickHouse native protocol Hello/Query packets, injects credentials into username/password slots, VIP-routed because no SNI
- **ssh** — Acts as SSH server to agent (accepts any auth; WireGuard is trust boundary) and SSH client to upstream (replays credential's key/password). Lazy-generated per-endpoint ed25519 host keys persisted in BlobStore

### Credential plugins: injection at the wire

Each credential plugin implements protocol-specific injection interfaces:

- `HTTPCredentialRuntime.InjectHTTP` — stamp Authorization/cookie headers
- `PostgresAuthCredential.PostgresAuth` — return (user, password) for SCRAM offload
- `ClickhouseAuthCredential.ClickhouseAuth` — return (user, password) for Hello packet injection
- `TLSCredentialRuntime.ConfigureUpstreamTLS` — add client cert for mTLS
- `HITLNotifier.NotifyHITL` — post approval prompt to Slack/Discord/Telegram
- `HTTPRequestSigner.SignHTTPRequest` — AWS SigV4 signing (reads endpoint service+region)
- `EKSBearerMinter.MintEKSBearer` — mint k8s-aws-v1 bearer for EKS

Multi-credential dispatch uses **disambiguators**: placeholder value (HTTP), user (Postgres/SSH), database (ClickHouse). The agent sends a placeholder token; the gateway detects it and swaps in the real secret.

### Rule engine: compiled CEL with fail-closed safety

Rules compile CEL conditions at policy load time. The compiler (`match/cel.go`) pre-computes safety properties:

- **Case-insensitive normalization**: AST walk lowercases string literals compared against lowercased-path fields. `sql.verb == "SELECT"` works with lowercase input at zero runtime cost.
- **Truncation fail-close**: If a rule reads a truncatable field (body, SQL statement) and the wire frontend truncated the bytes, the rule auto-denies WITHOUT evaluating CEL. No partial-match risk.
- **Unparseable fail-close**: If the parser refused the input and a rule reads a parser-derived field, auto-deny. Raw `sql.statement` is exempt — it's populated regardless of parse success.
- **Variable usage tracking**: AST walk determines which top-level variables a condition references. The gateway buffers the HTTP body only when needed.

### Facets: composable protocol families

Protocol families are registered as "facets" (`facet/facet.go`) — each owns its CEL environment, matcher factory, request preparation, and dashboard report fields. Three built-in: `http`, `sql`, `k8s`. Facets compose: a k8s rule can reference `http.method` because the k8s facet adds the HTTP facet's CEL env. Adding a new protocol = registering a new facet + endpoint plugin — no dispatcher changes.

### HITL: human + LLM approval chains

Rules can specify `approve = [...]` chains. Two modes:

**Synchronous**: Gateway holds the client connection. Approvers (`dashboard`, `human_approver` via Slack/Discord/Telegram, `llm_approver` with inline policy prompt) get called via `ApproverRuntime.Approve()`. The `HITLPool` publishes pending entries and blocks on a decision channel.

**Async with retry grants**: When sync times out, a durable operation record is created in SQLite. Client receives 202 with `status_token`. Operator approves/denies later. On approval: a one-shot retry grant. The agent can re-send the EXACT same request once — validated via HMAC fingerprint of the original request (profile, principal, auth binding, body).

## Key Techniques

- **WireGuard userspace netstack as universal proxy**: Instead of per-protocol proxy configuration, ALL TCP traffic enters through one interface. The dispatcher classifies by port, SNI, DNS VIP, or ConnIndex. Any protocol works without agent changes.

- **DNS VIP routing**: For protocols without SNI (SSH, ClickHouse native), the gateway runs an in-process DNS server on :53. It returns virtual IPs for known hostnames. When the agent dials the VIP, the VIP table recovers the hostname and routes to the right endpoint. This leverages existing agent DNS behavior as an implicit routing signal.

- **CEL AST rewrite for case normalization**: Rather than normalizing at runtime or requiring operators to use lowercase, the CEL compiler walks the AST at compile time and rewrites string literals. Zero runtime cost, invisible to operators.

- **Placeholder-driven credential selection**: The agent never holds real secrets. It sends placeholder strings (e.g., `ghp_clawpatrol_placeholder_do_not_use`). The gateway's `PlaceholderDetector` interface (one per endpoint family) scans the request for candidate placeholders, and the dispatcher picks the matching credential. Different agent profiles can use different placeholders to get different credentials.

- **Postgres SCRAM offload**: The gateway terminates SCRAM-SHA-256 on both sides — it IS the server to the agent and IS the client to upstream. The agent receives a synthesized `AuthenticationOk` without ever participating in the SCRAM exchange. This is only possible because the gateway controls both sides of the connection.

- **Per-process network namespace isolation**: On Linux, `clawpatrol run` creates a new netns using `unshare(CLONE_NEWUSER|CLONE_NEWNET|CLONE_NEWNS)`. The agent process runs inside, sees only the WireGuard interface. The auth key is hidden by mounting tmpfs over `~/.clawpatrol/` in the child namespace. macOS uses NetworkExtension with PPID filtering instead.

## Design Decisions

**L3 interception over HTTP proxy**: More complex but completely transparent. The agent sees real DNS, real IPs, real TLS certs (minted by the gateway CA). Any TCP protocol works — Postgres, ClickHouse, SSH. Trade: requires kernel-level networking (netns on Linux, NE on macOS).

**CEL over Rego/OPA**: CEL is sandboxed and well-documented for GCP users (Deno's ecosystem). But it's less expressive than Rego — no partial evaluation, no policy composition. The CEL compiler compensates with smart AST analysis for truncation/unparseable detection.

**Single binary with SQLite**: Everything in one Go binary — gateway, dashboard, migrations, tunnel termination. No external database, no Redis, no cloud dependency. Trade: single-writer bottleneck, no horizontal scaling. Suitable for small-team deployments (the target use case).

**Plugin registry requiring Go code**: Unlike configuration-driven proxies, adding a new protocol means writing Go code and registering a plugin. This is type-safe and compile-time checked but raises the bar for extensibility. External plugins via gRPC (`config/extplugin/`) exist but are less mature.

**One-shot retry grants for async HITL**: Rather than holding connections open indefinitely, the system creates a durable retry grant. This avoids connection timeout issues but requires the agent to implement polling/retry logic. The HMAC fingerprint ensures the retry grant can only be used for the exact original request.

## Comparison Notes

**vs. [[OneCLI]]**: OneCLI is a credential vault with transparent proxy injection. Clawpatrol does credential injection too, but adds protocol-aware rule enforcement, human approval workflows, and full audit logging. OneCLI focuses on the "agent never holds secrets" problem; Clawpatrol solves the broader "gate everything the agent does" problem.

**vs. [[agentsh]]**: agentsh is an execution-layer security gateway using FUSE+eBPF+seccomp for filesystem and syscall-level enforcement. Clawpatrol operates at the network layer — they're complementary. agentsh controls what the agent can DO locally; Clawpatrol controls what it can REACH remotely.

**vs. [[yolo-cage]]**: yolo-cage uses Vagrant + egress proxy to sandbox agents. Clawpatrol's per-process netns approach is lighter-weight but Linux-specific. yolo-cage's VM approach works cross-platform but is heavier.

**vs. [[PgDog]]**: PgDog is a PostgreSQL proxy for connection pooling and load balancing. Clawpatrol's Postgres MITM does auth offload and SQL-aware rule enforcement — different problem space. PgDog optimizes database connections; Clawpatrol gates what queries can execute.

**vs. [[StrongDM Factory Techniques]]**: StrongDM's dark factory patterns (DTU, Gene Transfusion, Filesystem-as-memory) are about the agent development workflow. Clawpatrol is about the security boundary AROUND that workflow. They could work together: Clawpatrol gates the dark factory's outbound traffic.

**vs. [[LLM Guard]]**: LLM Guard scans for prompt injection and data leakage. Clawpatrol gates actions at the network layer, not the prompt layer. Complementary: LLM Guard at the input, Clawpatrol at the output.

## Tags

#tool #project #security #agents #gateway #proxy #golang

---
*Sources: [[summary/clawpatrol.md]]*
*Last updated: 2026-05-31*
