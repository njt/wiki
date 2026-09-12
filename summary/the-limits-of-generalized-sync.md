---
url: https://aaltodoc.aalto.fi/server/api/core/bitstreams/d485ca46-ef01-41bc-ae4c-d468afb209a8/content
title: "The Limits of Generalized Sync: A Taxonomy of Architectures, Trade-offs, and Decision Factors"
author: Mikael Siidorow
date_fetched: 2026-07-03
date_published: 2026-05-25
topics:
  - distributed-systems
  - databases-and-data
---

# The Limits of Generalized Sync: A Taxonomy of Architectures, Trade-offs, and Decision Factors

Master's thesis by Mikael Siidorow, Aalto University School of Science, Master's Programme in Computer, Communication and Information Sciences. Supervised by Prof. Petri Vuorimaa, advised by Juho Vepsäläinen (DSc) and Ian Tuomi (MSc). Collaborative partner: Teamspective Oy. 131 pages. Licensed CC BY-NC-SA 4.0.

## Research Question

**Main RQ**: To what extent can client-side data synchronization be generalized across web applications?

Three sub-questions:
1. **ARQ1 (Taxonomy)**: What architectural patterns exist across the synchronization spectrum from server-authoritative to local-first?
2. **ARQ2 (Trade-offs/Limits)**: What trade-offs and limits do these architectural patterns introduce?
3. **ARQ3 (Fitness)**: How do application requirements determine sync engine fitness?

## Method

- Literature review (distributed systems, CRDTs, optimistic replication)
- Practitioner content analysis of 69 sources (conference talks, podcasts, blog posts, 2023–2026)
- Semi-structured interviews with 13 practitioners (vendors, adopters, custom builders)
- Case study: adopting Zero sync engine at Teamspective (B2B SaaS, managed Postgres on Render)
- AI-assisted source processing (Gemini 3.1 Pro) with manual validation
- Claude Code and Codex used extensively during case study implementation

## Taxonomy: Eight Architectural Dimensions

1. **Offline capability** — most discriminating dimension; constrains viable authority model
2. **Integration depth** — pluggable (integrate with existing backend) vs bundled (replace backend)
3. **Authority model** — server-authoritative vs decentralized/multi-master vs hybrid
4. **Partial replication** — query-driven dynamic vs static sync rules
5. **Conflict resolution strategy** — CRDT, LWW, custom handlers, revision trees
6. **Read/write path architecture** — engine-owned vs engine-transported vs no write path
7. **Sync unit** — state transfer vs operations vs events vs delta-state
8. **Network topology** — star (client-server) vs mesh (peer-to-peer)

Dimensions 1–6 are what practitioners actively reason about; 7–8 are treated as platform defaults.

## Four Architectural Clusters (NOT a spectrum)

### 1. Database-pluggable
Zero, ElectricSQL, PowerSync. Integrate with existing Postgres backends. Server-authoritative. Incremental adoption path.

### 2. Bundled platforms
Convex, InstantDB, Jazz v2. Replace the backend entirely. Server-authoritative. Greenfield-oriented.

### 3. CRDT-decentralized
Classic Jazz, Ditto, Automerge. Decentralized authority via CRDTs. Full local-first operation. Cost: data model constrained to CRDT-compatible structures.

### 4. Singular
LiveStore, RxDB, Turso Sync, CouchDB, TinyBase. Each occupies a unique architectural position, not clustering with others.

## Seven Technical Trade-offs

1. **Consistency vs responsiveness** — local writes feel instant but weaken consistency; server writes guarantee consistency at latency cost
2. **Read/write path asymmetry** — read path commoditizes across applications; write path resists generalization (ElectricSQL pivoted to read-only based on this finding)
3. **Integration depth vs incremental adoption** — pluggable engines enable incremental adoption; bundled engines require greenfield
4. **Authorization** — three vendors independently converged on server-enforced authorization; 26 of 69 sources identify it as a barrier
5. **Partial replication** — static rules are rigid; dynamic queries break on complex joins; lowest practitioner coverage of any dimension
6. **Conflict resolution** — receives extensive academic attention but rarely surfaces in practice; LWW dominates in production
7. **Hidden integration costs** — schema evolution across client versions, client-generated IDs, browser maturity gaps, deployment requirements surface only after adoption

## Five Fundamental Limits

1. **Authorization as hard limit** — permission changes must propagate across replicas and reconcile offline-mutated state against revoked permissions. Vendor consensus: separate authorization from sync mechanism.
2. **Global invariants and data volume** — uniqueness constraints require consensus (mathematical limit); ~50K objects is a practical scaling boundary (Linear, Zero users)
3. **Write-path resistance to generalization** — business logic, conflict resolution, and failure recovery are application-specific. Notion couldn't use standard CRDTs; Figma maintains two separate sync engines.
4. **Browser platform immaturity** — WASM SQLite + OPFS hit Chrome incognito limits (~100MB) and Safari doesn't support OPFS in incognito. Mobile native SQLite is mature; browser offline is bleeding edge.
5. **Partial sync and networking as unsolved layers** — no approach has converged on a scalable solution without engineering workarounds

## Application Fitness

| Type | Fitness | Key constraint |
|------|---------|----------------|
| Offline field apps | Strong | Few viable engines (PowerSync, CouchDB) |
| B2B productivity tools | Strong | Moderate data volumes |
| Collaborative editing | Partial | Requires CRDT or custom conflict resolution |
| Transactional | Poor | Needs global invariants incompatible with local writes |
| Data-intensive / analytics | Poor | Sync replicates rows, not computed results |
| "Sync Light" (read-only reactivity) | Alternative | Component assembly, not a sync engine |

## Decision Factors

- **Speed and DX drive adoption** — engineers adopt sync engines as "state management replacements" and "API simplifiers," not for synchronization itself
- **Postgres compatibility as binary filter** — existing Postgres apps eliminate bundled platforms immediately
- **Offline requirement as binary filter** — immediately narrows viable engine set
- **No production standard yet** — ecosystem immaturity; team size mediates impact
- **Build vs buy** — sync engines' primary competitor is engineers building custom, not another engine

## Case Study: Zero at Teamspective

- Deployed Zero for user profile settings and workspace feature flags; rest of app remains on SWR caching
- Managed Postgres (Render) surfaced provider-specific friction: no logical replication privileges, no event triggers → custom WAL workaround, 14-minute resync on slot loss
- Client-generated IDs required for Zero inserts → kept row creation on REST API (update-only mutations)
- Authorization boundary: ZQL cannot express per-viewer column projections
- Zero integration was straightforward at code level; hidden costs lived in surrounding ecosystem (monorepo tooling, Docker, managed Postgres)

## Key Theoretical Lens

The thesis uses three lenses from software engineering classics:
- **Brooks (essential vs accidental complexity)**: sync engines remove network plumbing (accidental) but not domain-specific consistency requirements (essential)
- **Spolsky (leaky abstractions)**: sync engines leak at the database layer — engineers still provision logical replication, publications, event triggers
- **Saltzer et al. (end-to-end argument)**: authorization is application-specific by the end-to-end argument; sync engines honor this by not trying to generalize it

## Keywords

CRDTs, data synchronization, eventual consistency, local-first software, replicated databases, sync engines, web applications
