---
url: https://github.com/cipherstash/cipherstash
title: CipherStash
author: CipherStash
date_fetched: 2026-07-21
date_published: unknown
---

# CipherStash — Full Architectural Analysis

CipherStash is a data security platform providing searchable field-level encryption with identity-bound key management. The `cipherstash/cipherstash` repo is a "front door" meta-repository that serves as the entry point to a multi-repo product ecosystem. It contains only a README and a discovery skill for AI agents — the actual implementation lives across four product repositories.

## Products & Architecture

### Overview

The platform has five products arranged in a layered architecture:

```
Your application (client-side encryption)
                │
    ┌───────────┴───────────┐
@cipherstash/stack (TS/JS)  CipherStash Proxy (Rust)
Protect.go (Go)             transparent, in front of Postgres
    │                        │
    └───────────┬────────────┘
                │
       EQL (searchable ciphertext + indexes in PostgreSQL)
                │
       ZeroKMS (unique key per value, derived on demand)
```

### 1. Stack (`@cipherstash/stack`, TypeScript)

The primary SDK for TypeScript/JavaScript applications. A pnpm workspace monorepo managed with Turborepo. The language breakdown (83.7% TypeScript, 15.9% PLpgSQL) indicates it ships with PostgreSQL-native components for query execution.

**Schema definition** uses builder-pattern helpers:
```typescript
encryptedTable("users", {
  email: encryptedColumn("email").equality().freeTextSearch(),
  salary: encryptedColumn("salary").orderAndRange()
})
```

**Client-side encryption**: `client.encrypt(plaintext, { column, table })` returns a discriminated union (`.failure` / `.data`) — a functional error-handling pattern rather than throwing. Model-level operations (`encryptModel`, `bulkEncryptModels`) handle entire objects or batches.

**Identity binding**: Uses `OidcFederationStrategy` to authenticate as the end user, with `.withLockContext({ identityClaim })` binding decryption to a specific identity claim. Data encrypted with a lock context can only be decrypted with the same context.

**Integrations**: Drizzle ORM, Supabase, Prisma, Amazon DynamoDB.

### 2. Proxy (Rust)

A transparent PostgreSQL wire-protocol proxy written in Rust. Sits between the application and PostgreSQL, intercepting SQL traffic and encrypting/decrypting on the fly with zero SQL changes.

**Dual-stream architecture**: Each client connection spawns two concurrent handlers:
- `frontend.rs`: Intercepts client-to-server messages, runs type inference, encrypts, forwards transformed SQL
- `backend.rs`: Intercepts server-to-client messages, identifies encrypted columns in result rows, batch-decrypts, returns plaintext

Both share a `Context` tracking active statements, portals, column metadata, and timing.

**Extended query protocol**: Tracks state across Parse → Bind → Describe → Execute. At Parse time, SQL is intercepted, type inference runs, literals are encrypted, the AST is transformed, and metadata is stored. At Bind time, parameters are encrypted using Parse-phase metadata.

**Five Rust crates** under `packages/`:
- `cipherstash-proxy/`: Main binary (wire protocol, encryption service, schema management, TLS, CLI)
- `eql-mapper/`: SQL type inference engine (unifier, type definitions, trait bounds, operator/function signatures, transformation rules)
- `eql-mapper-macros/`: Procedural macros for declaring operator/function type signatures
- `showcase/`: Example healthcare data model

### 3. Encrypt Query Language (EQL, PostgreSQL extension)

A PostgreSQL extension that stores encrypted data as `jsonb` and provides index term extractors. Written primarily in PLpgSQL with code generation from a Rust catalog.

**Three index term types**, each optimized for a different query pattern:

| Index Type | Mechanism | PostgreSQL Index | Operations |
|---|---|---|---|
| `unique` | HMAC-SHA256 (`eql_v3.hmac_256`) | Hash index on `eq_term()` | Equality (`=`) |
| `match` | Bloom filter over n-grams | GIN index on `match_term()` | `LIKE`, `ILIKE` |
| `ore` | Order-Revealing Encryption blocks | B-tree on `ord_term()` | `>`, `<`, `BETWEEN`, `ORDER BY` |

**Performance characteristics** (from EQL documentation):
- Equality (hash): 0.43–0.46 ms, near-constant across dataset sizes
- ORE range: 4.1 ms at 10K rows → 8.1 ms at 10M rows
- Bloom match: 1 ms at 10K → 216 ms at 10M (degrades with dataset size)
- GROUP BY on encrypted data: ~3× overhead vs plaintext

**Code generation**: The `eql_v3` scalar encrypted-domain types are generated from a single Rust constant (`CATALOG` in `crates/eql-domains/src/lib.rs`). Each `DomainFamily` row declares domain name, ScalarKind, and which index terms apply. `mise run build` invokes `cargo run -p eql-codegen` to regenerate SQL. Generated files are deterministic and byte-identical for an unchanged catalog, committed in place under `src/v3/scalars/<T>/`.

**Rust crate structure**: `eql-domains` (catalog), `eql-codegen` (SQL generator), `eql-bindings` (shared types + generated TS/JSON Schema), `eql-tests-macros` (SQLx test matrix).

### 4. Protect.go (Go SDK)

A Go encryption SDK that wraps precompiled Rust static libraries (six platform-specific `.a` files: macOS ARM64/Intel, Linux ARM64/x64, glibc and musl variants). No Rust toolchain needed by consumers.

**Two schema definition approaches**:
- Struct tags: `cs:"unique"`, `cs:"match"`, `cs:"ore"`, `cs:"ste_vec(prefix=t/c)"` with Go type inference (`string`→`text`, `int`→`number`, etc.)
- Programmatic builder: `protect.NewSchema("users").Column("email", protect.CastAsString).Equality().FreeTextSearch().Done()`

**Bulk operations**: `BulkEncryptModels` makes "a single KMS call for all fields across all models." Query values are encrypted via `EncryptQuery(ctx, col, queryType, plaintext)` returning index-specific fields (`UniqueIndex`, `MatchIndex`, `OreIndex`) for use in SQL WHERE clauses.

### 5. ZeroKMS

A key management service that inverts the traditional KMS model: keys are derived on demand based on user identity and access policies, never persisted. Benchmarked at up to ~14× faster than AWS KMS for high-frequency operations. Keyset isolation provides "provable cryptographic separation, not policy enforcement" for multi-tenant deployments.

## Key Techniques & Non-Obvious Implementation Choices

### 1. Non-deterministic encryption with deterministic search terms

CipherStash uses non-deterministic AES-256-GCM for the ciphertext (repeated encryptions of the same plaintext produce different ciphertexts), but accompanies each ciphertext with deterministic search term indexes (HMAC-SHA256 for equality, Bloom filter for text search, ORE for range). This gives the best of both worlds: ciphertexts resist comparison attacks, but the deterministic HMAC term enables equality lookups. The HMAC is keyed per `(value, encryption-key)` pair, so even the deterministic term reveals nothing to an attacker without the key.

### 2. Robinson-style type inference in a SQL proxy

The Proxy's `eql-mapper` crate implements a constraint-based type inference engine operating at parse time without executing SQL. Types are unified using a Robinson-style unification algorithm adapted for SQL semantics. Every AST node gets a type (Native, EQL, Projection, Var, or Associated). EQL types carry trait bounds (`Eq`, `Ord`, `TokenMatch`, `JsonLike`, `Contain`) that form a hierarchy (`Ord` implies `Eq`). When two `Partial` EQL types for the same column meet, their bounds merge (union); when `Partial` meets `Full`, the result promotes to `Full`. The system automatically infers the minimum encryption payload needed for each value — a smart optimization that avoids encrypting unnecessary index terms.

### 3. Three independent visitors in a single AST traversal

The Proxy's type inference runs three visitors simultaneously during a single AST walk:
- **ScopeTracker**: lexical scopes, table visibility, column resolution, wildcard expansion
- **Importer**: brings schema into scope, creates typed projections
- **TypeInferencer**: actual inference via the unifier, with per-node implementations

This is more efficient than three separate passes and avoids consistency issues between visitor ordering.

### 4. Polymorphic operator signatures via procedural macros

EQL operators and functions are declared with generic type parameters and trait bounds using Rust procedural macros. Associated types (e.g., `<T as JsonLike>::Accessor`) resolve differently depending on whether T is EQL or Native, allowing the same signature to work for both encrypted and unencrypted columns. Unknown functions fall back to assuming all arguments are native — a safe default since native types satisfy all trait bounds.

### 5. Sparse batch encryption

The Proxy collects only non-null encrypted values while tracking original positions, sends them to ZeroKMS in a single batch, then reconstructs the result vector with ciphertexts placed back at their original positions. This minimizes API calls while correctly handling nullable columns.

### 6. ArcSwap for lock-free schema reads

The Proxy's schema state uses `ArcSwap`, providing lock-free reads with atomic updates. Query processing never blocks on schema reload. Schema reloads happen on startup (exponential backoff, up to 10 attempts), periodically (configurable interval), and on-demand (DDL detection triggers reload on transaction commit).

### 7. In-transaction DDL tracking via SchemaWithEdits overlay

DDL statements (`CREATE TABLE`, `ALTER TABLE`, `DROP TABLE`) are captured in a `SchemaWithEdits` overlay that masks the loaded schema so subsequent statements in the same transaction see updated structure. On commit, the proxy triggers a full schema reload.

### 8. Transformation rule composition via Rust tuples

SQL AST transformation rules implement a `TransformationRule` trait and compose via tuple implementation (chains of 1 to 16 rules). Each rule has a `would_edit` method enabling dry-run optimization: the system first checks if any rule would modify the AST, and only rebuilds if necessary — for passthrough queries (no encrypted columns), this avoids AST reconstruction entirely.

### 9. SQL dependency graph with `tsort`

EQL SQL sources use `-- REQUIRE:` comments for dependency edges. The build collects edges, resolves them with `tsort`, and concatenates files in dependency order into a single installer. The `eql_v3` surface is self-contained — no `eql_v2.<symbol>` reference appears anywhere, enforced in CI.

### 10. Identity-bound encryption keys

Rather than storing keys and checking access via policy, CipherStash binds encryption keys to identity claims cryptographically. A user's JWT token is tied to their data keys — only that user can decrypt their data. This means access is enforced cryptographically, not just by policy evaluation.

## Design Decisions & Trade-offs

### Performance vs. security in index types

The choice between ORE (Order-Revealing Encryption) and OPE (Order-Preserving Encryption) is a deliberate security/performance trade-off. OPE produces ciphertexts that sort under PostgreSQL's native byte ordering (no operator class needed), making range scans cheaper. But OPE reveals more about relative value order. ORE is recommended by default, with the docs noting the choice depends on "performance and threat-model requirements."

### Information leakage as a deliberate, bounded trade-off

Each index type reveals something about the plaintext: HMAC reveals when two values are equal (but only to someone without the HMAC key), Bloom filters reveal overlap in n-gram sets, ORE reveals relative ordering. This is not a bug — it's a deliberate design choice to make encryption usable. The platform documents this honestly rather than pretending to be perfect.

### Encryption client determines index terms, not the database

In most database security products, the database defines which columns are encrypted and how. In CipherStash, the encryption client decides which index terms a value carries. The database has no configuration state for which operations are allowed on a column — the query surface is fixed by the domain variant at creation time. This keeps security policy in the application layer where developers already manage access control.

### SDK vs. Proxy: two integration strategies

The dual approach (SDK and Proxy) lets teams choose based on their constraints. The SDK requires code changes but gives fine-grained control over the encryption schema. The Proxy requires zero SQL changes but adds a network hop and is PostgreSQL-specific. Both share the same EQL foundation.

### ZeroKMS: derived, not stored

Traditional KMS stores keys and serves them on demand. ZeroKMS derives keys from identity and policy on each request, never persisting them. This eliminates the storage attack vector entirely but means that key derivation must be fast enough for every encryption/decryption operation. The claimed 14× speed advantage over AWS KMS is the enabler for this architectural choice.

### PostgreSQL as the encrypted database, not a custom engine

Rather than building a custom encrypted database, CipherStash extends PostgreSQL with searchable encryption via extensions. This means users keep their existing PostgreSQL tooling, backups, and operational knowledge. The trade-off is that PostgreSQL wasn't designed for encrypted query execution, so edge cases (untyped literals falling back to native jsonb operators, operator mismatch on wrong column types) are sharp.

### Code generation from a single Rust catalog

All EQL domain types, operators, functions, and aggregates are generated from a single Rust constant. This ensures consistency — the catalog is the "source of truth" and all SQL surfaces derive from it deterministically. No hand-editing of generated files is allowed (CI gate). The constraint that generated code must never use `STRICT` and must be `LANGUAGE plpgsql` for blockers (to prevent the planner from skipping RAISE) shows deep PostgreSQL expertise.

### Multi-tenant isolation via keysets

Cryptographic separation (one keyset per tenant) rather than application-level policy enforcement. This is "provable" isolation — even if the application has a bug, data from one tenant can't be decrypted with another tenant's keys. The trade-off is operational complexity: keyset management becomes a new operational concern.

## Innovation Points

1. **Searchable encryption on standard PostgreSQL** — not a custom database engine, but extensions on the database you already run. This is the "boring technology" approach to a hard problem.

2. **The Proxy's type inference engine** — a full constraint-based type system for SQL implemented at parse time with Robinson unification. Not something you'd expect to find in a database proxy. The three-visitor-in-one-pass design is elegant.

3. **Zero-storage KMS** — keys derived on demand and never persisted. This inverts the entire model of how key management works and eliminates a whole class of attacks.

4. **Honest about information leakage** — the documentation explicitly names what each index type reveals, rather than hiding it behind "military-grade encryption" marketing.

5. **Agent-native design** — the `cipherstash/cipherstash` meta-repo is explicitly designed for AI agents as first-class consumers, with a `SKILL.md` discovery skill, `llms.txt` documentation, and a deterministic product → repo → docs map. This is forward-thinking in an era where agents are increasingly the first users of developer tools.

## Comparison to Related Concepts

### vs. AWS KMS / cloud KMS
ZeroKMS is complementary (backed by AWS KMS at the root) but inverts the model: KMS stores keys, ZeroKMS derives them. The performance claim (14× faster) matters because it enables per-value key derivation at query time.

### vs. MongoDB Field-Level Encryption / Cassandra TDE
These provide client-side encryption but lose searchability. CipherStash preserves querying via EQL index terms, at the cost of information leakage through the indexes.

### vs. Always Encrypted (SQL Server) / pg_tde
Always Encrypted supports equality on deterministic encryption but not range, text search, or JSON queries. CipherStash's multi-index approach provides a richer query surface at the cost of larger ciphertext (each value carries HMAC + Bloom + ORE terms).

### vs. application-level encryption
Custom app-level encryption (encrypt in app code, decrypt in app code) provides the strongest security but kills all database query capability. CipherStash is the middle ground — encrypted in transit and at rest, searchable via EQL indexes.

### vs. CipherStash Stack vs. CipherStash Proxy
Stack requires code changes (define schemas, call encrypt/decrypt) but works with any database. Proxy requires zero code changes but is PostgreSQL-specific. Both share EQL and ZeroKMS. This is a well-considered two-pronged strategy for different adoption paths.

## Repository Structure Notes

The `cipherstash/cipherstash` repo itself is intentionally minimal — a README plus a `skills/` directory containing a single discovery skill for AI agents. The implementation is distributed across:
- `cipherstash/stack` (TypeScript monorepo, 2,068+ commits, 199 releases, latest `stash@0.17.1`)
- `cipherstash/proxy` (Rust, Docker image on Docker Hub)
- `cipherstash/encrypt-query-language` (PLpgSQL + Rust codegen, published on database.dev)
- `cipherstash/protectgo` (Go + precompiled Rust static libs)
