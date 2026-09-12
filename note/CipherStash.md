# CipherStash

A data security platform that makes encrypted data searchable without decryption. Field-level encryption with unique keys per value, identity-bound key derivation, and a PostgreSQL extension that supports equality, range, ordering, and free-text queries directly over ciphertext. The platform spans a TypeScript SDK, Go SDK, Rust PostgreSQL proxy, and a managed key management service — all sharing a common encrypted query language (EQL) foundation.

---

## Architecture

CipherStash uses a **layered architecture** with two integration paths converging on shared lower layers:

```
Application code (client-side encryption)
        │
    ┌───┴──────────────┐
Stack (TS/JS SDK)      Proxy (Rust, transparent)
Protect.go (Go SDK)    sits in front of Postgres
    │                   │
    └────────┬──────────┘
             │
    EQL (searchable ciphertext as jsonb + indexes in PostgreSQL)
             │
    ZeroKMS (unique key per value, derived on demand, never stored)
```

**Two integration strategies serve different adoption paths**:
- **SDK path** (Stack or Protect.go): Encryption in application code — fine-grained control, works with any database, requires code changes. Stack offers first-class ORM integrations (Drizzle, Supabase, Prisma, DynamoDB).
- **Proxy path**: A Rust binary speaking the PostgreSQL wire protocol. Sits between app and Postgres, encrypts/decrypts transparently with **zero SQL changes**. Ideal for legacy systems.

### Stack (TypeScript SDK, `@cipherstash/stack`)

A pnpm workspace monorepo managed with Turborepo. Schema definition uses a builder pattern: `encryptedTable("users", { email: encryptedColumn("email").equality().freeTextSearch() })`. All encryption/decryption operations return discriminated unions (`.failure` / `.data`) rather than throwing. Identity-bound encryption uses `OidcFederationStrategy` + `.withLockContext({ identityClaim })` to cryptographically bind data to a specific end user.

### Proxy (Rust)

Five crates under `packages/` in the `cipherstash/proxy` repo. The core innovation is **`eql-mapper`**, a constraint-based SQL type inference engine operating at parse time without executing SQL.

**Dual-stream per connection**: `frontend.rs` handles client→server (type inference, encryption, AST transformation), `backend.rs` handles server→client (result decryption, batch processing). Both share a `Context` tracking statement/portal state. The extended query protocol (Parse → Bind → Describe → Execute) is fully tracked, allowing parameter encryption to use metadata computed during Parse.

**Type inference** uses Robinson-style unification adapted for SQL. Every AST node gets a type (Native, EQL, Projection, Var, Associated). EQL types carry trait bounds (`Eq`, `Ord`, `TokenMatch`, `JsonLike`, `Contain`) in a hierarchy where `Ord` implies `Eq`. Three visitors run in a single traversal: ScopeTracker (lexical scopes), Importer (schema import), TypeInferencer (unification). Partial EQL types merge their bounds when they meet; Partial meeting Full promotes to Full. The system automatically infers the **minimum encryption payload** needed.

**Schema management**: Uses `ArcSwap` for lock-free reads — query processing never blocks on schema reload. DDL statements during transactions create a `SchemaWithEdits` overlay masking the base schema. On commit, a full reload is triggered.

**Transformation pipeline**: Rules implement `TransformationRule` and compose via Rust tuple chains (1–16 rules). Each rule has `would_edit` for dry-run optimization — passthrough queries skip AST reconstruction entirely.

### Encrypt Query Language (EQL, PostgreSQL extension)

The shared foundation. Encrypted data stored as `jsonb` with the payload:

| Field | Name | Purpose |
|-------|------|---------|
| `c` | Ciphertext | AES-256-GCM encrypted value |
| `u` | Unique index | HMAC-SHA256 for equality |
| `m` | Match index | Bloom filter for text search |
| `o` | ORE index | Order-revealing blocks for range |
| `sv` | STE vector | Searchable tree encoding for JSON |

**Term extractor functions** (`eq_term()`, `ord_term()`, `match_term()`) pull individual index terms from the `jsonb`, enabling functional indexes (hash for equality, B-tree for ORE, GIN for bloom). This is the mechanism that makes PostgreSQL's native index types work over encrypted data.

All EQL domain types are **code-generated** from a single Rust catalog constant (`CATALOG` in `crates/eql-domains/src/lib.rs`). CI gates prevent drift between generated and committed SQL. The SQL dependency graph uses `-- REQUIRE:` comments resolved by `tsort` into a deterministic concatenation order.

### ZeroKMS

Key derivation, not key storage. A unique key is derived per value on each encryption/decryption call, bound to identity and policy. Never persisted. Benchmarked at ~14× faster than AWS KMS for high-frequency operations. Multi-tenant isolation via **keysets** — one keyset per tenant for provable cryptographic separation.

## Key Techniques

### Non-deterministic ciphertext + deterministic search indexes

Each value gets AES-256-GCM ciphertext (non-deterministic: same plaintext → different ciphertext) alongside deterministic index terms keyed per `(value, encryption-key)`. This prevents ciphertext comparison attacks while enabling equality lookups via the HMAC term. The ciphertext reveals nothing; each index term reveals a specific, bounded property. This is a deliberate and explicit trade-off, not a weakness.

### Parse-time SQL type inference (Proxy)

The eql-mapper crate implements a full constraint-based type system for SQL, using Robinson-style unification. This is not something you'd expect to find in a database proxy. The three-visitor-in-one-pass design (ScopeTracker + Importer + TypeInferencer) is efficient and elegant. Associated types (`<T as JsonLike>::Accessor`) resolve polymorphic operator signatures differently for encrypted vs. native columns, enabling a single signature to serve both code paths.

### Auto-inferred minimum encryption payload

The type inference system determines which index terms a query actually needs and only encrypts those terms — a `Partial(Eq)` payload for `WHERE email = ?` vs. a `Full` payload for `INSERT`. This minimizes KMS calls and ciphertext size. The decision is made at parse time by the unification algorithm, not by explicit configuration.

### Sparse batch encryption

Non-null values are collected with position tracking, sent to ZeroKMS in one batch, then reconstructed with ciphertexts at their original positions. This handles nullable columns efficiently.

### Code-generated SQL from a single Rust catalog

`DomainFamily` rows in a Rust constant declare all encrypted-domain types, their scalar kinds, and their index terms. `cargo run -p eql-codegen` regenerates SQL surfaces deterministically. This eliminates inconsistency between domain types — the catalog IS the source of truth.

### Identity-bound encryption via lock contexts

Data keys are cryptographically bound to identity claims (from JWT tokens). A value encrypted with `identityClaim: "user_123"` can only be decrypted by that user. Access is enforced cryptographically, not by policy evaluation that could be bypassed by a bug.

### Lock-free schema reads with ArcSwap

The Proxy's schema state uses `ArcSwap` — readers never block, schema updates are atomic. Combined with SchemaWithEdits overlay for in-transaction DDL, this gives correctness + performance.

## Design Decisions

### Honest about information leakage

The documentation explicitly names what each index type reveals: HMAC reveals value equality (to key holders), Bloom reveals n-gram overlap, ORE reveals relative ordering, OPE reveals more ordering than ORE. The ORE vs. OPE choice is presented as a security/performance spectrum with clear guidance. This transparency is rare in security products.

### Encryption client owns the schema, not the database

In CipherStash, the encryption client decides which index terms a value carries. The database has no configuration state for which operations are allowed on a column — the query surface is fixed by the domain variant. This keeps security policy in the application layer.

### PostgreSQL, not a custom database

CipherStash extends PostgreSQL rather than building a custom encrypted database. This preserves existing tooling, backups, and operational knowledge. The trade-off: PostgreSQL edge cases (untyped literals silently falling back to native `jsonb` operators, operator mismatch errors) can be surprising.

### Keys are derived, never stored

Traditional KMS stores keys; ZeroKMS derives them on demand from identity and policy. This eliminates the key storage attack vector entirely. Enabling this requires key derivation fast enough for per-query use — the claimed 14× performance advantage over AWS KMS.

### Agent-native by design

The `cipherstash/cipherstash` repo explicitly targets AI agents as first-class consumers: a deterministic product→repo→docs map, a discovery skill for task routing, and `llms.txt` documentation. This is forward-thinking at a time when agents are increasingly the first interface to developer tools.

## Comparison Notes

- **vs. AWS KMS / cloud KMS**: Complementary — ZeroKMS is backed by AWS KMS at the root key level but inverts the model (derive vs. store). The performance delta (14×) is the enabler.
- **vs. SQL Server Always Encrypted**: CipherStash supports a richer query surface (range, text search, JSON queries vs. equality-only deterministic encryption). Trade-off: larger ciphertext (multiple index terms per value).
- **vs. application-level encryption**: Custom app encryption gives strongest security but kills query capability. CipherStash is the middle ground.
- **vs. MongoDB Field-Level Encryption**: Similar client-side encryption model but CipherStash preserves searchability via EQL indexes at the cost of information leakage through those indexes.
- **Platform comparison (Stack vs. Proxy)**: Stack = code changes, any DB, fine-grained control. Proxy = zero code changes, PostgreSQL-only, transparent. The dual approach is a deliberate strategy to cover greenfield and brownfield.

Tags: #tool #database #security #encryption

---
*Sources: [[raw/cipherstash]]*
*Last updated: 2026-07-21*
