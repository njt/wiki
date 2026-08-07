# Cloudflare OS

Cloudflare's open-source platform for bringing AI agents into every corner of an organization — not just engineering. It combines agent workspaces grounded in company context, a novel security model where policy follows what the agent has observed, and a platform where every "file" can be a full-stack app that agents and humans both modify. Deployed internally across all of Cloudflare's functions, now available for any organization to run on their own Cloudflare account.

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

### vs. SmolForge — Platform Primitives in Production

[[SmolForge]] is the strongest independent proof that Cloudflare's platform primitives compose into full applications. It's a GitHub clone — Git hosting, issues, PRs, CI/CD, gists, wikis — built entirely on Workers, D1, R2, and Durable Objects, the same stack Cloudflare OS uses. Where Cloudflare OS is a general-purpose organizational platform, SmolForge is a domain-specific application of the same primitives to the code-hosting problem. SmolForge's repository agents (every repo gets its own durable agent authority) and Forge Deploy (immutable previews + SHA-gated activation) demonstrate the kind of agent-native features that become possible when the platform itself is built on serverless primitives.

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

## Key Quotes

> "Every organization has a mission, a reason for being. Organizations pass that mission — along with their terminology, procedures, systems, standards, and ways of working — to their people."

The article opens by framing the problem as organizational context transfer. This matters because it positions Cloudflare OS not as a coding tool but as an *organizational* tool — the thesis is that what makes agents useful at work isn't model capability, it's access to institutional knowledge and internal systems.

> "Agents start with no access. Inside, every agent and app starts with access to nothing. An agent can ask for access to a specific resource, which you can grant or deny."

This is the security model's foundation and its sharpest departure from the status quo. Most enterprise AI deployments start by handing agents API keys and hoping for the best. Cloudflare OS inverts that: default-deny, capability-by-capability, with every resource access mediated through a Gatekeeper that holds the credential and enforces policy.

> "Controlling the initial read is not enough. Take, for example, the case where an agent reads a sensitive table in a data warehouse and uses it to produce a live dashboard. Sharing the dashboard must not become a way to share the table with people who could not access it directly."

The observation-tracking model is the most interesting security idea in the post. It solves the chained-access problem that [[Zero Trust for AI Agents]] identifies — an agent with read-only database access and a file-sharing tool can exfiltrate data using only legitimate operations. Cloudflare OS's answer is to attach resource observations to every artifact the agent produces, and re-check permissions when anyone tries to view or interact with that artifact. This is Least Agency operationalized at the platform level.

> "So if you can build a tool to do a job yourself, agents can use your tool to do the job when you're not there."

The Cap'n Web RPC layer means the same server methods an app exposes to its browser client are callable by agents. This closes the loop: apps aren't just things agents build for humans, they're tools agents can use. The platform becomes a compounding library of agent-callable capabilities.

> "Not every task needs the most expensive model. You may not want to run the most expensive frontier model to summarize your unread emails every morning."

Cloudflare OS is model-agnostic by design, routing every inference call through AI Gateway. This gives organizations a single place to decide model routing policy — which echoes the model-tiering patterns in [[The Advisor Strategy]] and [[Thrifty (Tiered Delegation for Claude Code)]], but applied at the organizational rather than individual-developer level.

## Key Themes

- **#platform** — Cloudflare OS as organizational agent infrastructure: workspaces, Gatekeepers, apps, blueprints
- **#security** — Observation-tracking policy model: permissions follow data through the entire artifact graph
- **#pattern** — Capability-based access for agents: typed bindings like `env.PROJECT.listIssues()` instead of raw API keys
- **#concept** — Blueprint sharing: apps shared as modifiable templates, not static artifacts. "File a feature request" becomes "fork and modify with AI"
- **#tool** — Gatekeepers as service-specific security middleware: OAuth, policy enforcement, observation logging, rate limiting
- **#pattern** — Dynamic Workers + Durable Object Facets as the serverless app substrate: every app gets its own SQLite database and isolated V8 runtime

## Critical Analysis

**What's genuinely new:** The observation-tracking security model is the standout contribution. Most agent platforms focus on *granting* access — who can do what. Cloudflare OS additionally tracks *what was observed* and propagates those constraints downstream. This solves a real problem that almost nobody else is addressing: the data-exfiltration-via-legitimate-operations path that [[Zero Trust for AI Agents]] identifies as beyond the reach of credential-level controls.

**The Gatekeeper pattern** is also significant. Rather than giving agents raw API access with broad permissions, Gatekeepers expose a narrow TypeScript API with resource-scoped, policy-enforced operations. This is the "group tools around intent" principle from [[Building Agents for Production Systems with MCP]] applied to the entire internal systems surface area, not just MCP tool definitions.

**The Blueprint model** is clever but underspecified. Sharing an app's *code* without its *state* or *resources* means each blueprint instantiation starts fresh — no data leakage, no credential sharing. But it also means every team that forks a dashboard blueprint has to reconnect their own data sources. The post doesn't address how blueprints handle the resource-binding problem at fork time.

**What's underspecified:** The post is an announcement, not a technical deep-dive. Key questions left open: How do Gatekeepers handle rate limiting at the observation level (if 1,000 agents observe the same resource, does every downstream artifact carry 1,000 observations)? What's the latency overhead of re-checking observation policies on every view? How does the platform handle the case where an observed resource's permissions change *after* the agent produces an artifact — does the artifact become inaccessible, or is access snapshotted at creation time?

**The build-vs-buy tension:** [[The Case Against Building Your Own Agent Platform]] argues that building agent platforms is a trap for most organizations. Cloudflare OS is a hybrid answer: the *platform* is open source and deployable, but the *context, skills, workflows, and integrations* are what each organization builds for itself. This is Johnson's "buy the category stuff, build the business logic" heuristic, with Cloudflare providing the category stuff as deployable infrastructure rather than a SaaS product. The managed-product roadmap (Cloudflare dashboard, containers, Slack integration) suggests the eventual SaaS offering may make the build-vs-buy calculus even cleaner.

**Compared to personal agent frameworks:** [[Personal Agents]] documents the "my own AI assistant" movement where individuals wire up agents to their own tools. Cloudflare OS is the organizational version of the same impulse — but with security, governance, and sharing built in from the start rather than bolted on. The observation-tracking policy model in particular is something no personal agent framework attempts.

**The Cloudflare portfolio play:** Cloudflare OS draws on an unusual number of Cloudflare-internal infrastructure pieces: Access (auth), Workers (serverless compute), Durable Objects (stateful serverless), AI Gateway (model routing), Dynamic Workers (on-demand isolates), Durable Object Facets (newly built for this project). It's both a product announcement and a demonstration that Cloudflare's platform primitives compose into something larger than any individual service. The open-source release is smart — it lets organizations customize without forking, and creates an ecosystem of Gatekeepers and blueprints that Cloudflare doesn't have to build itself.

**The partnership strategy** (Presidio and Happy Cog) is a tell: Cloudflare OS is not just infrastructure, it's a *services* play. The source code is open, but the context curation, skill building, interface customization, and rollout are where the value sits. This is the same dynamic as [[Cloud Software Factories]] — the factory code matters less than the organizational knowledge embedded in it.

## Related Pages

- [[Security and Sandboxing]] — The Gatekeeper + observation-tracking model is a new entry in the credential-management and data-exfiltration landscape
- [[Zero Trust for AI Agents]] — Cloudflare OS operationalizes "Least Agency" and the "impossible vs. tedious" test at the platform level
- [[Cloudflare Temporary Accounts for Agents]] — The temporary-accounts feature serves the same platform as Cloudflare OS; the agent-as-user thesis spans both
- [[The Case Against Building Your Own Agent Platform]] — Cloudflare OS as the "buy the category stuff" half of Johnson's heuristic
- [[Building Agents for Production Systems with MCP]] — MCP Server Portals bridge Cloudflare OS to the existing MCP ecosystem; the Gatekeeper pattern extends "group tools around intent" to internal systems
- [[Personal Agents]] — The organizational-scale version of the personal agent impulse, with security and governance built in
- [[Cloud Software Factories]] — Shares the centralization thesis: move agents out of individual laptops into governed infrastructure
- [[Who Does What — Team Topologies for the Agentic Platform]] — Wulveryck's "agentic platform" is exactly what Cloudflare OS aims to be; the observation-tracking model is the kind of systemic guardrail his framework calls for
- [[Orchestrating AI Code Review at Scale]] — Cloudflare's existing production AI infrastructure; Cloudflare OS extends the same platform thinking beyond engineering
- [[Project Glasswing — Mythos at Cloudflare]] — Cloudflare's security-focused agent harness; Cloudflare OS applies the platform-security mindset to general organizational work

---
*Sources: [[raw/cloudflare-os]], [[summary/cloudflare-os]] (announcement post); [[raw/cloudflare-os-repo]], [[summary/cloudflare-os-repo]] (codebase analysis)*
*Last updated: 2026-08-06*
