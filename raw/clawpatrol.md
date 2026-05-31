---
url: https://github.com/denoland/clawpatrol
title: clawpatrol — The security firewall for agents
author: Deno Land Inc.
date_fetched: 2026-05-31
date_published: 2026-05-31
---

# clawpatrol — Deep Architectural Analysis

## Summary

Clawpatrol is a transparent network-layer security firewall for AI coding agents. It sits between agents (Claude, Codex, etc.) and upstream services (APIs, databases, Kubernetes), intercepting traffic at the IP/TCP level via WireGuard or Tailscale tunnels. The agent has no awareness of the proxy — there's no `HTTPS_PROXY` env var, no CA bundle to configure.

The gateway:
1. Terminates the tunnel, inspects destination port/IP to pick protocol handlers
2. Decodes wire protocols (Postgres, ClickHouse native, Kubernetes API, SSH, HTTPS) to extract typed facts
3. Runs CEL rules against those facts (deny `DROP TABLE`, gate `kubectl delete pod` behind human approval)
4. Injects real credentials at the wire, replacing placeholder tokens the agent holds
5. Logs every request, verdict, and latency to SQLite, exposed via dashboard SSE

Written in Go (~49K lines), single binary. HCL configuration. MIT licensed.

## Architecture Deep Dive

### 1. Gateway struct (main.go:337-391)

The `Gateway` struct is the central coordinator. It holds:
- `cfg *config.Gateway` — parsed HCL config
- `policy atomic.Pointer[config.CompiledPolicy]` — compiled rule set, atomically swapped on reload
- `connIdx atomic.Pointer[runtime.ConnIndex]` — dstIP→endpoint lookup for non-SNI protocols
- `dnsvip *dnsvip.Allocator` — hostname→VIP mapping for protocols without SNI
- `certs *CertCache` — on-the-fly TLS leaf certs signed by gateway CA
- `oauth *OAuthRegistry` — live OAuth token refresh
- `agents *AgentRegistry` — per-device agent state for dashboard
- `hitl *HITLRegistry` — pending approval pool for human-in-the-loop
- `secrets runtime.SecretStore` — stacked secret store (DB slots > OAuth tokens > env vars)
- `tunnels *TunnelManager` — lifecycle for persistent tunnels (k8s port-forward, Tailscale, SSH)
- `blobs runtime.BlobStore` — SQLite-backed persistent blob store for plugins (SSH host keys, JWT keys)
- `transports sync.Map` — per-endpoint `http.Transport` cache to avoid per-request allocation

### 2. Transport Layer

Two transport modes, both supported simultaneously:

**WireGuard mode** (`wireguard.go`, `run_linux.go`, `relay_linux.go`):
- Embedded `wireguard-go` + gVisor userspace netstack
- Gateway listens on UDP :51820, allocates /32 per peer from `subnet_cidr`
- gVisor netstack accepts SYNs to ANY destination IP/port — no `iptables` on gateway
- On Linux, `clawpatrol run` creates a fresh netns per process, hides auth key by overlaying tmpfs on config dir before exec

**Tailscale mode** (`tailscale.go`, `setup_tailnet_login.go`, `run_tsnet_common.go`):
- Embedded `tsnet` node joins the tailnet in-process
- Mints OAuth-derived auth keys for onboarded clients
- Funnel for internet-reachable join bootstrap on :443
- Per-process: ephemeral tailnet nodes on Linux (netns + tsnet.Server with Ephemeral:true); macOS NetworkExtension hosts tsnet stack and PPID-filters flows
- Whole-machine: installs system Tailscale with `--authkey`, sets gateway as exit node

### 3. Dispatch System (runtime/dispatch.go)

When a TCP flow arrives at the gateway:

| dst port | Handler |
|----------|---------|
| :443 | TLS SNI peek → HostEndpoint lookup → HTTPS family dispatch (https/k8s) or passthrough |
| :5432 | Postgres wire-protocol MITM (auth offload + sql-family rule matching) |
| :53 | DNS VIP responder (returns allocated VIP for known hostnames; forwards everything else) |
| any, dst is VIP | VIP-bound endpoint runtime (ssh, clickhouse_native reached by hostname) |
| else | direct-IP ConnIndex lookup; transparent relay as fallback |

Key dispatch concepts:

- **SNI peek** (`main.go:229-303`): For TLS :443, parse ClientHello bytes to extract SNI hostname without completing the handshake. Look up endpoint by hostname. Then MITM with dynamically-minted leaf certs.
- **ConnIndex** (`conn_route.go`): Endpoint plugins declare `ConnRouteHosts() []string`. At policy load, hosts are DNS-resolved and indexed as dstIP→endpoint. Postgres connections on :5432 are dispatched by IP.
- **DNS VIP** (`dnsvip/dnsvip.go`): For protocols without SNI/Host headers (SSH, ClickHouse native), each hostname gets a stable virtual IP from a private range. DNS queries for those hostnames return VIPs; connections to VIPs look up the hostname in the VIP table.
- **Transparent relay**: If no plugin claims the destination, bytes are spliced to the real upstream unchanged.

### 4. Plugin Architecture (config/plugin.go)

The config system uses a plugin registry pattern borrowed from Terraform:

```go
type Plugin struct {
    Kind     Kind       // endpoint | credential | approver | rule | profile | tunnel
    Type     string
    New      func() any // factory for gohcl decode target
    DecodeBody func(body hcl.Body, ctx *hcl.EvalContext, target any) hcl.Diagnostics
    Validate func(decoded any, name string, ctx *BuildCtx) hcl.Diagnostics
    Build    func(decoded any, name string, ctx *BuildCtx) (any, hcl.Diagnostics)
    Runtime  any // protocol-specific interface, type-asserted at dispatch time
    Emit     func(body any, name string, hb *hclwrite.Body)
    Family   string // for endpoint plugins
    Disambiguators []string // for credential dispatch
}
```

Plugins register at `init()` time. The blank import chain at `config/plugins/all/all.go` pulls in every built-in plugin.

**Framework-level attribute peeling** (`plugin.go:158-203`): The loader strips framework-owned attributes (endpoint binding, placeholder) from block bodies before plugin decode. New cross-cutting features just add a one-line entry to `frameworkAttrsByKind` — no plugin churn.

### 5. Credential System (internal/config/plugins/credentials/)

Each credential plugin implements one or more of the runtime interfaces:

- `HTTPCredentialRuntime.InjectHTTP` — stamp Authorization header / cookie
- `PostgresAuthCredential.PostgresAuth` — return (user, password) for SCRAM offload
- `ClickhouseAuthCredential.ClickhouseAuth` — return (user, password) for Hello packet injection
- `TLSCredentialRuntime.ConfigureUpstreamTLS` — add client cert for mTLS
- `HITLNotifier.NotifyHITL` — post approval prompt to Slack/Discord/Telegram
- `HTTPRequestSigner.SignHTTPRequest` — AWS SigV4 signing (reads endpoint service+region)
- `EKSBearerMinter.MintEKSBearer` — mint k8s-aws-v1 bearer for EKS

Credentials are bound to endpoints via framework-level attrs (`endpoint = X` or `endpoints = [...]`). Multi-credential dispatch uses disambiguators:

- **placeholder** (HTTP): gateway detects the placeholder string the agent sent (e.g., `ghp_clawpatrol_placeholder_do_not_use`) and picks the matching credential
- **user** (Postgres/SSH): matches the wire-protocol username
- **database** (ClickHouse): matches the Hello packet's default_database

### 6. Rule Engine + CEL (match/cel.go)

Rules are written in HCL with CEL conditions:

```hcl
rule "no-secrets" {
  endpoint  = k8s-prod
  condition = "k8s.resource == 'secrets'"
  verdict   = "deny"
  reason    = "Secret values must not leave the cluster via the agent"
}
```

The CEL compiler (`CompileCondition`) does several smart things at compile time:

1. **Case-insensitive normalization**: Walks the AST and lowercases string literals compared against lowercased-path fields (e.g., `sql.verb`), so `sql.verb == "SELECT"` matches lowercase input without runtime cost.

2. **Truncation-aware compilation**: Pre-computes whether a condition reads a truncatable field (body, SQL statement). If so, the dispatcher auto-denies on truncated requests without evaluating the CEL program — a fail-closed gate.

3. **Unparseable-aware compilation**: Pre-computes whether a condition reads a parser-derived field (SQL verb, tables, functions). If the parser refused the input and a rule reads one of these, auto-deny without evaluation. Raw `sql.statement` is exempt — it's populated regardless of parse success.

4. **Variable usage tracking**: `collectReferencedVars` walks the AST to determine which top-level variables a condition references. The gateway uses this to decide whether to buffer the HTTP body before evaluation.

### 7. Facet System (facet/facet.go)

Protocol families are registered as "facets" — each facet owns:
- Its CEL environment (which variables and types are available)
- Its `NewMatcher` (compiles conditions against its env)
- Its `PrepareRequest` / `Report` hooks (derive metadata, export dashboard fields)
- Its `Transport()` (how the gateway dispatches flows for this family)

Built-in facets: `http`, `sql`, `k8s`. Adding a new protocol = registering a new facet + endpoint plugin(s) — no changes to the rule engine or dispatcher.

### 8. HITL (Human-in-the-Loop) System

Two modes:

**Synchronous HITL** (`hitl_operation_store.go`, `hitl_async_runtime.go`):
- When a rule matches with `approve = [...]`, the gateway holds the client connection
- Routes through approvers: `dashboard` (web UI button), `human_approver` (Slack/Discord/Telegram interactive message), `llm_approver` (LLM judge against an inline `policy` prompt)
- Each approver implements `ApproverRuntime.Approve(ctx, req) → (ApproveVerdict, error)`
- The `HITLPool` interface lets any approver publish a pending entry and block on a decision channel

**Async HITL with retry grants** (`hitl_async_runtime.go`):
- When sync approval times out, the system creates a durable operation record in SQLite
- Client receives a 202 with a `status_token` for polling
- Operator approves/denies via dashboard or messenger
- On approval: a one-shot retry grant is created. The agent can re-send the EXACT same request once
- Retry is validated via HMAC fingerprint of the original request (profile, principal, auth binding, request body)
- Lifecycle: `sync_waiting` → `pending_approval` → `approved_waiting_for_retry` → `executing_upstream` → `upstream_succeeded`/`upstream_failed`

### 9. Dashboard & Web API (web.go)

The dashboard runs inside the gateway binary (React SPA embedded via `embed.FS`). Routes include:

- `/api/state` — TTL-cached gateway state for dashboard polling (every 5s)
- `/api/requests` — recent request log with facets
- `/api/fixtures/export` — export request history for regression tests
- `/api/hitl/pending` — pending human approval queue
- `/api/hitl/decide` — operator approves/denies a pending request
- `/api/onboard/start` + `/api/onboard/poll` — device-flow onboarding
- `/api/cred/<name>/interactive` — Slack interactive message webhook endpoint
- `/api/peer/tsnet/register` — tsnet daemon registration

Authentication: dashboard password OR tailnet identity (when in Tailscale control mode). Operators list in HCL controls who can bypass the password.

### 10. Tests & Quality

The test suite is significant:
- E2E tests for HITL async flow (`hitl_async_e2e_test.go`)
- Config validation tests with expected error messages (`config/testdata/error_*.hcl` + `.errors.txt`)
- Golden file tests for config compilation (`feature_*.hcl` + `.want.json`)
- WireGuard watchdog tests, response sanitization tests, dashboard auth tests
- Action fixture tests: JSON fixtures simulate requests for regression testing rules (`clawpatrol test` command)

## Innovation Points

1. **Transparent L3 interception via userspace netstack**: Unlike most API gateways which are HTTP proxies, Clawpatrol operates at the IP layer. The agent sees real DNS, real IPs, real TLS certs (minted on the fly by the gateway CA). This means ANY TCP protocol can be intercepted without agent code changes — the Postgres, ClickHouse, and SSH support proves this.

2. **Protocol-aware credential injection**: Not just HTTP Bearer tokens. The gateway offloads Postgres SCRAM authentication (acting as one peer), injects ClickHouse Hello packet credentials, and replays SSH authentication using the credential's key/password. The agent never sees real database credentials or SSH keys.

3. **CEL truncation fail-close via compile-time AST analysis**: Rather than trusting rules to handle truncated input, the CEL compiler pre-computes which fields are truncatable and auto-denies at dispatch time. This is a rare example of a security property enforced through the compiler, not through runtime checks.

4. **DNS VIP as protocol-routing primitive**: For protocols without SNI, the gateway runs its own DNS server to route traffic by hostname. This is clever because it leverages existing agent behavior (DNS resolution) as an implicit routing signal — no configuration needed on the agent side.

5. **Polling-based async HITL with one-shot retry grants**: Instead of holding connections open indefinitely, the system creates a durable retry grant that only matches the exact original request. This avoids connection timeout issues while maintaining security.

## Design Trade-offs

| Choice | For | Against |
|--------|-----|---------|
| L3 interception (WireGuard) | Transparent to agents, works for any TCP protocol | Complex setup, requires kernel/userspace netstack |
| CEL expressions | Sandboxed, well-documented, familiar to GCP users | Limited expressiveness compared to Rego/OPA |
| SQLite for state | Single binary, no external deps | No horizontal scaling, single-writer bottleneck |
| Go plugin architecture | Type-safe, compile-time checked | Adding protocols requires Go code, not just config |
| Per-process netns (Linux) | True per-process isolation, no side effects | Linux-only; macOS uses NetworkExtension (different code paths) |
| Placeholder token injection | Agent code unchanged, no SDK integration needed | Requires operators to set placeholder env vars |
