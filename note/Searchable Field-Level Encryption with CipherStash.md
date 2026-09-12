# Searchable Field-Level Encryption with CipherStash

CipherStash brings Data Level Access Control (DLAC) to Supabase/Postgres: encrypt individual field values at the application layer, store searchable encrypted metadata alongside the ciphertext, and enforce decryption policy per-value — all while keeping WHERE clauses, joins, sorting, and fuzzy text matching working. The zero-knowledge key management means neither CipherStash nor Supabase can access your plaintext.

---

## Key Quotes

> "Every encrypted value carries a policy stating who can read it and under what condition."

This is the architectural headline. DLAC moves enforcement from the query layer (RLS, application middleware, WHERE clauses) to decryption time. A row-level rule says "this user can see this row." A field-level policy says "this user can read *this specific value* under *these conditions*." The difference matters when different fields in the same row have different sensitivity levels — a user profile where the email is PII but the display name is public.

> "Because keys never leave your control, neither CipherStash nor Supabase can access your plaintext data."

The zero-knowledge claim: key derivation happens client-side through ZeroKMS, and the service never holds keys. This is the standard zero-knowledge architecture pattern applied to database field encryption. The trust boundary sits at the application, not the database or the key management service.

> "Each encrypted value gets its own key, derived on demand."

Per-value key derivation is the scaling trick. If every value gets a unique key, a single key compromise exposes one value, not the entire column. Combined with the ability to split keys across regions for data residency, this targets the compliance stack directly — FedRAMP, IL4, GDPR data residency, HIPAA minimum-necessary.

> "Encrypted values appear as random bytes to Postgres, so `WHERE email = ?` returns nothing, indexes break, and joins fail."

The author nails the structural reason field-level encryption has been impractical: encryption destroys the queryability that databases exist to provide. CipherStash's answer — Searchable Encrypted Metadata (SEM) as JSON alongside the ciphertext — is the same shape as every searchable-encryption scheme (blind indexes, deterministic encryption, order-preserving encryption) but packaged as an SDK integration rather than a cryptographic research project.

## Key Themes

- **#concept** — Data Level Access Control (DLAC): policy enforcement at decryption, not at query time. A granularity shift from row/table to individual values.
- **#tool** — CipherStash: DLAC platform integrating with Supabase via SDK (TypeScript: Supabase.js, Drizzle, Prisma Next) and wire-protocol proxy (CipherStash Proxy) for non-SDK access.
- **#tool** — ZeroKMS: zero-knowledge key management service that derives per-value keys on demand without ever holding them.
- **#pattern** — Searchable Encrypted Metadata (SEM): storing encrypted values as JSON payloads with non-reversible metadata that Postgres can filter, sort, and join against.
- **#pattern** — Application-layer encryption with transparent SDK integration: marked columns flow through the SDK; unmarked columns behave normally.

## Critical Analysis

**The real innovation is packaging, not cryptography.** Searchable encryption has existed in academic literature for decades — deterministic encryption, order-preserving encryption, blind indexes, structured encryption. What CipherStash ships is the integration engineering: an SDK that transparently encrypts marked columns, a proxy for non-SDK access, and key management that doesn't require a team of cryptographers. The innovation is making it `npx stash init --supabase` rather than a research paper.

**The SEM approach inevitably leaks metadata.** If you can search on `email = 'alice@example.com'`, the database learns *something* about the plaintext — at minimum, that two rows share the same email value. CipherStash is upfront about this (it's "searchable encrypted metadata," not homomorphic encryption), but the post doesn't quantify what information leaks through the metadata layer. For regulated workloads, this is the question a security review will ask: "what exactly can a database administrator see, and what would an attacker with a pg_dump learn?" The answer matters more than the encryption claim.

**The proxy is the sleeper feature.** CipherStash Proxy — a Postgres wire-protocol proxy that handles encryption/decryption transparently — solves the SDK-only problem. Analytics jobs, admin tooling, background workers in Go or Python, BI tools connecting directly: these are the things that make field-level encryption break in practice. A proxy that speaks the Postgres wire protocol means existing SQL clients work without code changes. This is the right architecture for a problem where "just use the SDK" is a non-answer for half your infrastructure.

**The threat model is compliance, not nation-states.** The post mentions HIPAA, GDPR, SOC 2, FedRAMP, IL4. These are regulatory frameworks, not adversarial threat models. CipherStash protects against database-level access (breached database, subpoenaed cloud provider, curious DBA) but not against application-level compromise (if your app is pwned, the attacker calls the SDK with valid credentials). This is the right threat model for the target market — healthcare, finance, government — but it's worth being explicit: this is defense-in-depth against database compromise, not end-to-end encryption against application compromise.

**The Supabase integration is a distribution wedge.** Supabase has momentum as the Postgres platform for AI-era applications (see [[InsForge]], which positions as "Supabase for agents"). CipherStash picking Supabase as its first integration target is smart: a platform with developer growth, a Postgres foundation, and a customer base that cares about managed infrastructure. The `npx stash init --supabase` one-command setup is the kind of developer experience that turns a security product into a default.

## Related Pages

- [[InsForge]] — Open-source BaaS for coding agents: Postgres+RLS, auth, S3, Supabase-like architecture exposed as MCP tools
- [[Nubase]] — AI-native backend with Supabase-compatible auth + PostgREST, database-per-tenant isolation
- [[Xano]] — No-code backend with AI-generated Postgres and APIs at enterprise scale
- [[Databases and Data]] — Hub page for storage architecture, Postgres patterns, and data engineering
- [[Security and Sandboxing]] — Hub page for encryption, isolation, and agent safety
- [[Interdict]] — Runtime safety layer between AI agents and PostgreSQL: parses SQL through real Postgres AST, measures blast radius on risky writes
- [[HTTP API Design Guide (Heroku)]] — The ur-text of modern REST API design; relevant to any API that exposes encrypted data over HTTP

---
*Sources: [[raw/supabase-cipherstash-searchable-encryption]]*
*Last updated: 2026-07-21*
