# CRDTs Are Not Enough (Loro)

Zixuan Chen's field report from building Lody, a local-first sync engine and agent control plane on top of the Loro CRDT library: convergence of individual documents is only the start, and the hard problems are document discovery, cross-document visibility, local search under encryption, and making the cloud a dumb relay.

---

Chen and his collaborators have worked on Loro since 2022, but the argument here comes from a product, not a library: Lody, a local-first control plane for team AI agents, with thousands of documents to sync. The essay works backwards from UX needs to infrastructure, and the punchline is the title — the CRDT core (eventual consistency, incremental sync, dedup of out-of-order updates) is the *solved* part.

The opening anecdote is the sharpest thing in the piece: a user gets a new-message notification, opens the conversation, and the message isn't there. The conversation index and the message body are separate sync streams, delivered independently, and each converges perfectly — while the user-visible operation is broken. "Convergence within each document does not provide that visibility guarantee across documents." Even an atomic server-side commit wouldn't fix it; the relationship between updates must be tracked end to end.

Other unsolved layers he catalogues:

- **Document discovery and lazy loading** — solved with Flock, a new CRDT for syncing a directory (titles/metadata first, content on click), iterated from JSON → over-engineered binary V1 → columnar V2 inspired by database design.
- **Local storage and search** — SQLite-in-the-browser failed (one writer vs many tabs forces leader election); they fell back to IndexedDB. Full-text and semantic search under end-to-end encryption is explicitly open.
- **The backend** — Durable Objects couldn't carry the workload: connection-time billing and SQLite read/write costs inside hibernating objects made version diffs expensive. The answer is a *dumber* cloud: ElectricSQL's Durable Streams, an append-only log where clients store offsets, publish snapshots for compaction, and use an ephemeral channel plus append-CAS for presence and E2E crypto. The counterintuitive note: CRDTs should let the server do *less* (unlike OT), yet naive CRDT hosting still costs heavy compute — so make the server a relay over Raft groups with cold data in object storage.
- **Authentication** — still centralized; decentralized access control (Keyhive et al.) not integrated.

The Git/GitHub framing is doing real work: local-first doesn't mean no cloud, it means the cloud is a swappable backup and helper — GitHub succeeded *because* Git is local-first and GitHub is effortless. Local-first sync engines want the same split.

## Themes

#concept #tool #pattern

## Analysis

This is the most honest piece of local-first engineering writing I've read in a while, precisely because it refuses the genre's usual triumphalism. The movement's pitch is usually "local-first is obviously better and here's why the world is wrong" — Chen instead documents seven thousand commits of sync rework, an abandoned binary format, a failed SQLite-on-the-web architecture, and features (search, E2E, decentralized auth) still unresolved. That vulnerability is what makes the central claim credible: the CRDT library was never the product, and the gap between "documents converge" and "users see a coherent system" is where the actual engineering lives.

The cross-document visibility problem deserves to be a named concept in sync-engine design. It's a distributed-systems problem hiding inside a UX problem: the requirement is "no notification until the message is locally present," which is a read-visibility guarantee across two independent streams — the kind of thing transactions solved in databases, here smeared across the client, the transport, and the UI.

It's also a fitting coda that Lody is an *agent control plane*. The piece never dwells on the agent angle, but the workload — thousands of independent conversations, metadata-first browsing, E2E encryption of agent data — is a preview of what sync infrastructure looks like when agents are the primary producers of documents.

- Complicates [[The Limits of Generalized Sync]]: Chen's evidence that even within one stack (Loro), "one unified sync engine" remains aspirational — visibility, search, and auth are application problems the engine hasn't absorbed.
- Strengthens [[The Log — Unifying Abstraction for Real-Time Data]]: an industrial-strength vindication — Durable Streams is exactly the append-only log with offsets, here serving CRDT catch-up instead of stream processing, with the server reduced to a dumb relay.
- Complicates [[Durable Actors — Open-Source Durable Objects]]: a concrete negative result on the Durable Objects cost model — per-connection billing and hibernation-forced SQLite writes make single-object CRDT repos uneconomic, pushing toward object-storage-backed logs instead.
- Nuances [[Stash — Conflict-Free Folder Sync]]: both treat the directory (not the document) as the sync unit, but Lody's Flock shows how much policy — prefetch, search, visibility — lives above that layer even once conflict-free sync is free.

---
*Sources: [[raw/crdt-is-not-enough]], [[summary/crdt-is-not-enough]]*
*Last updated: 2026-11-13*
