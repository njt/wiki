# Cloudflare OS

Cloudflare's open-source AI productivity platform that reimagines the OS
abstraction for the agentic era: per-user sandboxed applications (Gadgets)
instead of centralized SaaS, capability-based security (Gatekeepers) instead
of ambient permissions, and Code Mode agents that write and execute code
rather than call predefined tools. Built entirely on Cloudflare Workers and
Durable Objects by the team that built Workers itself.

---

## Architecture

### The OS Metaphor, Made Concrete

Cloudflare OS maps traditional OS concepts onto its Workers-based
infrastructure directly and convincingly:

| Traditional OS | Cloudflare OS |
|---|---|
| Kernel | `workshop-backend` (Durable Object managing workspace state) |
| Device drivers | `gatekeeper-*` packages (per-service security wrappers) |
| Shell | `workshop-frontend` (web-based agent chat and gadget UI) |
| Processes | Gadgets (per-user, sandboxed application instances) |
| Executables | Blueprints (shareable `.gadget` templates) |
| ACLs | Shared permissions (lazily-evaluated permission graph) |
| Agents | First-class managed entities with their own restricted permissions |

The "kernel" (`workshop-backend/src/server.ts`, ~865 lines + `overseer.ts`,
~9,500 lines) genuinely performs OS-kernel-like duties: connecting users to
programs and devices, sandboxing applications, and enforcing access control.
Gatekeepers — which connect users and agents to external services — are
analogous to device drivers: they wrap diverse external APIs behind a
uniform capability-based interface.

The key insight is that agents are *not* treated as users. They're first-class
entities accountable to a human but with their own restricted, revocable
permissions. This is what traditional OSes lack — and what Cloudflare OS
argues they should provide.

### Workspaces as Durable Objects

Every workspace is a single Durable Object (DO), giving each user's
environment strong consistency and persistent state without managing
databases. The DO stores:
- **Code** — a Yjs document synced between client and agent
- **Gatekeepers** — installed DO Facets wrapping external services
- **Gadgets** — per-user application instances
- **Chats** — agent conversation histories with compaction checkpoints
- **Blueprints** — shareable gadget code templates
- **Observers** — non-owner collaborators verified to see historical state

Gadgets run in Dynamic Worker Facets — a Workers Runtime feature added
specifically to support Cloudflare OS. Each gadget's server is a facet
isolated from the internet, accessible only through explicit bindings
configured by the user. Client code runs in a sandboxed iframe with
Content-Security-Policy and iframe sandbox restrictions.

### Cap'n Web RPC: The Universal Interface

All communication — client↔server, agent↔gadget, gatekeeper↔workspace —
uses Cap'n Web RPC over WebSocket, supporting promise pipelining. This
has a profound dual benefit:

1. **Low boilerplate for agents**: Define a method on the server, call it
   from the client as if local. Agents can discover and invoke any gadget's
   API without per-app MCP integration.
2. **Automatic agent API**: Every gadget exposes a well-structured API
   that the Code Mode agent can call directly, without the developer
   building any agent-specific integration layer.

### Yjs for Real-Time Sync

Yjs (V2 encoding) syncs code changes between the browser client, the
Durable Object storage, and the AI agent. This means:
- Humans and agents can edit code simultaneously
- Changes are replicated in real time across collaborators
- Edit history is preserved for replay and branching
- The agent sees code changes as they happen, not just at invocation time

This is the same CRDT-based approach that powers collaborative editing in
Google Docs — applied to the code that defines the application itself.

---

## Key Techniques

### Gatekeepers: Simulated Actions for Async Approval

Gatekeepers are the most innovative security contribution. Each external
service (GitHub, Google, Slack, etc.) gets a Gatekeeper — a Worker that:

- Wraps the service's native API behind a clean Cap'n Web interface
- Handles OAuth authorization
- Enforces narrow access to only the specific resource the user intended
- Logs every action for review
- **Simulates outcomes** for side-effecting actions to avoid blocking the agent

The simulation technique is the breakthrough. Traditional human-in-the-loop
setups require synchronous approval — the agent stops and waits. This is
so annoying that users disable it. Gatekeepers instead:

1. When the agent performs an action requiring approval, the Gatekeeper
   *simulates* the outcome locally
2. The agent believes the action completed and proceeds to queue up more work
3. If the agent tries to read back results, the Gatekeeper provides
   simulated data consistent with the simulated action
4. Once the agent finishes, the human approves or rejects actions in bulk,
   at their convenience

This decouples agent throughput from human attention — the agent never
stalls waiting for approval, but the human retains final authority over
every side-effecting action.

Gatekeepers implement a typed interface (`Gatekeeper<Session>`) with
methods for `applyAction()`, `rejectAction()`, `revertAction()`,
`addObserver()`, `removeObserver()`, and `getAutoApprovableActions()`.
An `ApprovalQueue` manages `authorizeObservation()`, `submitAction()`,
and hook binding.

### Code Mode Agents

Instead of a predefined tool set, Cloudflare OS agents use **Code Mode**:
they write and immediately execute JavaScript snippets via Dynamic Workers.

The agent loop (powered by `@earendil-works/pi-agent-core`) generates
code, which runs in a sandboxed Dynamic Worker with:
- Bindings to the gadget's server API (so the agent can call gadget methods)
- Bindings to Gatekeepers (so the agent can interact with external services)
- No internet access beyond explicit bindings
- Callback resolvers for async operations

The `CODE_MODE_HARNESS` is an inline string in `overseer.ts` that defines
the Dynamic Worker code running agent scripts. This harness provides the
bridge between the agent's JavaScript and the workspace's capabilities.

This approach has several advantages over tool-based agents:
- **Expressiveness**: arbitrary code can express any task, not just those
  anticipated by tool designers
- **Composability**: multiple operations can be chained in a single execution
- **Efficiency**: fewer round trips than tool-calling loops
- **Sandboxing**: the Dynamic Worker is isolated, so even buggy agent code
  can't escape its bindings

### Lazy Permission Graph Revocation

Sharing uses a directed permission graph with two roles:
- **build**: full read/write access (code, gatekeepers, chat, sharing)
- **use**: UI-only access (can interact with the gadget but not modify it)

Edges come from user assignments or share links (128-bit random keys,
HMAC-SHA-256 stored — raw keys never persisted server-side). An effective
role is computed at every `open()` call as the maximum role reachable
from the owner through valid edges.

The design is **lazy revocation**: to revoke access, you only need to sever
edges in the graph. The next `open()` recomputes reachability and the
former collaborator loses access. No credential rotation, no cache
invalidation — just graph topology.

A `keepUsers` option allows re-rooting the graph when removing collaborators,
preserving the subgraph of users who should keep access.

### Compaction with Immutable Checkpoints

Chat history compaction creates immutable checkpoints that bound message
replay. Each `CompactionCheckpoint` stores:
- The compacted-to message index
- A summary of the compacted portion
- Chat bindings at compaction time
- Observed code version
- Accepted and proposed changes

This means replaying a chat only needs to start from the most recent
checkpoint rather than from message zero — a practical necessity for
long-running agent conversations in a Durable Object with memory limits.

### Observer Tracking for Shared Gadgets

When a gadget is shared, every viewer must be able to verify they can
see everything the gadget has historically observed. The observer system
tracks per-gatekeeper, per-observer verification:

- Each observer gets a random opaque `observerId`
- Per-gatekeeper `accountChoices` map observers to their own connected
  accounts
- `authorizeObservation()` checks `excludeObservers` against the sharing
  graph before allowing new observations
- Four per-gatekeeper verification strategies: private-only (A), ACL
  check (B), data-set tracking (C), low-stakes no-op (D), not-applicable (N)

This is security infrastructure most collaborative agent platforms don't
even acknowledge as a problem.

---

## Design Decisions

### Per-User Instances Over Centralized SaaS

The defining architectural bet: every user runs their own copy of every
app. This inverts 25 years of cloud architecture for two reasons:

1. **Security through isolation**: a bug in the slide deck app can't leak
   anyone else's slides — each instance is in its own sandbox
2. **AI-driven customization**: users can ask their agent to modify any
   app's code to add features they need, without affecting other users

This only works because AI makes per-user customization economically
viable. In a pre-AI world, the overhead of maintaining per-user instances
would be absurd. In an AI world, the agent does the customization work,
and the sandboxing prevents the customization from becoming a security
risk.

### Capability-Based Access Over Ambient Permissions

Agents and gadgets start with access to *nothing*. Even if the workspace
has configured external accounts, agents don't automatically get to use
them. The user must explicitly *introduce* each agent to each resource.

This contrasts with most agent harnesses where MCP servers are configured
upfront and all tools are ambiently available in every chat. The capability
model keeps each agent restricted to only what it needs for the current
task.

Introductions happen via URL pasting, UI selection, or agent request
(with human approval). The agent can't self-escalate — it can only *ask*
for access it thinks it needs.

### Async Approval Over Synchronous Gates

The Gatekeeper simulation approach is a direct response to a known
failure mode: synchronous approval drives users to disable safety
mechanisms. By decoupling agent progress from human attention, Gatekeepers
make the secure path the convenient path.

This is the same design principle behind `sudo`'s timeout, SSH agent
forwarding, and password managers — security that fights user behavior
loses. Security that works with user behavior wins.

### Workers-Native Architecture

Cloudflare OS is built by the Workers team on Workers, using features
(Dynamic Workers, Facets) that were added to the runtime specifically
to support it. This isn't a generic web app that happens to run on
Workers — it's designed to exploit Workers-specific primitives:

- **Durable Objects** for strongly-consistent per-workspace state
- **Dynamic Worker Facets** for per-gadget isolation
- **Workers Bindings** for capability-based service connections
- **workerd** compatibility for self-hosted deployment

The tradeoff is platform coupling, but the benefit is that security
isolation comes from the runtime rather than application-level
enforcement — a genuinely stronger security posture.

---

## Comparison Notes

### vs. Traditional Coding Agents (Claude Code, Codex, Cursor)

Cloudflare OS is not a coding agent — it's a *platform* that *includes*
a coding agent. The agent is one component in a larger system that
manages workspaces, gadgets, sharing, and security. Where Claude Code
runs on a developer's laptop with filesystem access, Cloudflare OS
agents run in the cloud with capability-scoped access to specific
resources.

The Code Mode approach also contrasts with tool-based agents: instead
of a fixed set of tools (read, write, bash, etc.), the agent writes
arbitrary JavaScript that executes in a sandboxed Worker. This is more
flexible but also more dependent on the sandbox for safety.

### vs. Cloud Software Factories

Both [[Cloud Software Factories]] and Cloudflare OS centralize agent
execution and add governance layers. But the philosophies diverge:

- Factories optimize for **pipeline throughput** — triage→spec→implement
  →review→verify→ship→monitor as an automated assembly line
- Cloudflare OS optimizes for **individual empowerment** — every user
  gets their own apps, can modify them with AI, and shares them like
  documents

The factory is a replacement for the development organization. Cloudflare
OS is a replacement for the SaaS suite. Both are "operating systems" in
different senses — the factory is an OS for the SDLC; Cloudflare OS is
an OS for knowledge work.

### vs. MCP-Based Agent Architectures

Gatekeepers are "supercharged MCP servers" — they provide the same
service-wrapping function but add capability-based introductions,
simulated actions for async approval, and observer tracking for shared
context. An MCP server says "here are my tools"; a Gatekeeper says
"here are my tools, I'll simulate their effects until you approve,
and I'll make sure everyone who sees the results is authorized."

The [[Bringing MCP 2026-07-28 to Claude]] spec adds stateless HTTP and
OAuth 2.0, moving MCP closer to the Gatekeeper model in terms of
authentication — but the simulation and observer patterns remain
unique to Cloudflare OS.

---

*Sources: [[raw/cloudflare-os]]*
*Last updated: 2026-08-06*
