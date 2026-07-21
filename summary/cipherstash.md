# CipherStash — Summary

CipherStash is a data security platform providing **searchable field-level encryption** for PostgreSQL-backed applications. Its core innovation is making encrypted data queryable: you can run equality, range, ordering, and free-text search over ciphertext without decrypting it first. Encryption happens client-side, with unique keys per value derived on demand by ZeroKMS (a key management service that never stores keys). Identity-bound encryption ties decryption to end-user JWT claims, enforced cryptographically rather than by policy.

**Architecture**: A layered stack. At the top, developers choose between the **Stack** TypeScript SDK (encryption in application code, with ORM integrations for Drizzle/Supabase/Prisma/DynamoDB), the **Go SDK** (Protect.go, precompiled Rust static libs), or the **Proxy** (a Rust PostgreSQL wire-protocol proxy that encrypts/decrypts transparently with zero SQL changes). Both paths converge on **EQL** (Encrypt Query Language), a PostgreSQL extension that stores encrypted values as `jsonb` with three index term types: HMAC-SHA256 for equality, Bloom filters for text search, and Order-Revealing Encryption (ORE) for range/ordering. **ZeroKMS** derives unique keys per value on demand, never persisting them.

**Key techniques**: Non-deterministic AES-256-GCM ciphertext paired with deterministic search indexes; a Robinson-style constraint-based SQL type inference engine in the Proxy (three visitors in one AST traversal); ArcSwap lock-free schema reads; sparse batch encryption; polymorphic operator signatures via Rust procedural macros; code generation of all EQL SQL surfaces from a single Rust catalog constant; transformation rule composition via Rust tuple chains with dry-run optimization.

**Trade-offs**: Each index type leaks some information (HMAC reveals equality, Bloom reveals overlap, ORE reveals order) — deliberate, documented, and bounded. ORE vs. OPE offers a security/performance spectrum. The dual SDK/Proxy approach covers both new development and legacy migration, at the cost of maintaining two integration paths. ZeroKMS's key derivation model eliminates key storage attacks but requires fast enough derivation for every query.

**Meta-design**: The `cipherstash/cipherstash` repo is a "front door" designed for both human and AI agent consumption, with a deterministic product → repo → docs map, a discovery skill for agents, and AI-ready documentation (`llms.txt`).

---
*Source: [[raw/cipherstash]]*
*Last updated: 2026-07-21*
