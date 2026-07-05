# Locker

Open-source, self-hostable file storage and knowledge platform — a Dropbox/Google Drive alternative where you bring your own storage backend (local disk, S3, R2, Vercel Blob). Built by zmeyer44 in TypeScript on Next.js 16, tRPC, PostgreSQL, and BetterAuth. Docker Compose gets you running in minutes.

---

## Key Quotes

> "Your files, Your cloud, Your rules."

The pitch in six words. The self-hosting movement keeps producing tools that say: the cloud is just someone else's computer, and if you can `docker compose up`, that computer can be yours.

> "We moved 4TB of team files off Google Drive in a weekend."

Testimonial from an engineering lead at a 50-person startup. The key detail: they used an existing S3 bucket. Locker's multi-store architecture means you don't need to migrate data — you point it at storage you already have and it layers organization, sharing, and search on top.

> "Attach multiple storage backends per workspace with primary, replica, and read-only configurations."

This is the architectural insight that separates Locker from a CRUD app with a file upload field. Storage is abstracted behind provider adapters in a dedicated `packages/storage/` package, and switching providers is a single env var. The replication model (primary/replica/read-only) means you can tier storage by cost and durability without changing application logic.

> Locker auto-detects which platform it's running on and adjusts its capabilities accordingly. Persistent runtimes support all features. Serverless runtimes disable long-running operations.

Smart platform detection that acknowledges a real constraint: not all hosting is equal, and pretending otherwise creates broken features. The honesty about what doesn't work on Vercel (store sync, bulk KB ingestion) is refreshing compared to products that silently degrade.

## Key Themes

#tool #self-hosting #open-source #knowledge-base

**Multi-store abstraction.** The storage layer is the hardest part of any file-hosting app, and Locker treats it as a first-class plugin interface rather than bolting on S3 support after the fact. This is the right call for a self-hosted tool — your storage topology shouldn't constrain your file organization.

**AI as a feature layer, not the product.** The knowledge base, document transcription, and QMD-powered search sit *on top* of storage, not instead of it. This is the correct layering: storage is the durable primitive, AI is the discovery layer. Compare to tools that start with "AI knowledge base" and treat file storage as an afterthought.

**Plugin system from day one.** Built-in plugins for search (QMD, FTS), transcription, Google Drive sync, and knowledge base suggest the author understands that a self-hosted platform lives or dies by extensibility. Users will always want integrations the core maintainer didn't anticipate.

**Virtual bash filesystem.** The `ls`, `cd`, `cat`, `grep` shell via `just-bash` is a clever interface choice — it gives power users a familiar interaction model and makes the tool agent-friendly (agents already know how to use shell commands).

## Critical Analysis

Locker is ambitious in a way that's either a strength or a liability depending on your tolerance for scope. The feature list reads like what you'd get if you asked Claude to spec a file storage platform and said "yes" to every suggestion: file explorer, previews, tags, share links, upload links, command palette, knowledge base, plugins, transcription, notifications, workspaces, multi-store, virtual bash, quotas. This is a solo developer project with zero GitHub stars (as of fetch). The gap between ambition and adoption is the story here.

The tech stack choices are solid and modern — Next.js 16, tRPC, Drizzle, BetterAuth, Turborepo — but they also mean Locker requires Node.js 20+, pnpm 9+, Docker, and PostgreSQL 16+. That's a meaningful operational commitment for "just store some files." The counterargument is that anyone self-hosting already has Docker and Postgres, and the `docker compose up` path genuinely works in one command. But the dependency surface means Locker isn't competing with `rsync` or a NAS — it's a full application platform.

The knowledge base feature is the most interesting and the most suspect. AI-powered wikis with interactive graph views sound great in a README; in practice, they need aggressive curation to avoid becoming a second junk drawer alongside the file storage. The plugin architecture could save this — if the knowledge base is a plugin, it can be disabled by people who just want file storage. But the README lists it as a core feature, not a plugin, which suggests it's deeply coupled.

The virtual bash shell is genuinely clever. It maps filesystem operations to a CLI metaphor that both humans and agents understand, making Locker potentially useful as a shared file interface between people and coding agents. This is the one feature that feels like it came from using the tool rather than ideating about it.

For the self-hosting enthusiast, Locker is worth watching. For anyone else, the question is whether the complexity budget is justified by the feature set, or whether a simpler tool ([[Graft]] + a web UI, or just S3 with signed URLs) gets you 80% of the way with 20% of the operational overhead.

Cross-links: [[Self-Hosted LLMs]] (self-hosting ecosystem), [[Graft]] (object storage architecture), [[Dolt]] (open-source infrastructure), [[Headscale]] (self-hosted networking), [[QMD]] (used for semantic search plugin), [[PiClaw]] (self-hosted Docker platform), [[n8n]] (self-hosted workflow automation), [[Mist]] (collaboration without accounts), [[AI Killing B2B SaaS]] (open-source vs SaaS), [[Simplicity in the Age of AI-Assisted]] (the complexity budget question), [[Dev Containers]] (Docker-based infrastructure)

---
*Sources: [[summary/locker-dev]]*
*Last updated: 2026-05-15*
