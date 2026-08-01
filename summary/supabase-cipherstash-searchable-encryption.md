---
url: https://supabase.com/blog/searchable-field-level-encryption-with-cipherstash
title: "Searchable field-level encryption on Supabase with CipherStash"
author: bilharmer
date_fetched: 2026-07-21
date_published: 2026-07-09
---

Announces the CipherStash integration for Supabase — a Data Level Access
Control (DLAC) platform that extends access controls down to individual
encrypted values, enforced at decryption time rather than at the query layer.

The core problem: regulated-data teams choose between field-level encryption
(which breaks `WHERE`, indexes, and joins — forcing in-app filtering) or
skipping encryption entirely (which leaves plaintext exposed). CipherStash
offers a third path: each value is stored as a JSON payload containing
ciphertext alongside Searchable Encrypted Metadata (SEM). Postgres can
filter, sort, and join on the SEM without recovering the original value. Keys
are per-value, derived on demand via ZeroKMS, and never leave the customer's
control — neither CipherStash nor Supabase can access plaintext data.

The TypeScript SDK works transparently with Supabase.js, Drizzle, and Prisma
Next. A CipherStash Proxy speaks the Postgres wire protocol for scenarios
where the SDK doesn't fit (analytics, admin tools, other languages).

Targets HIPAA, GDPR, SOC 2, FedRAMP, and IL4 environments, claiming shorter
compliance reviews and reduced breach surface while keeping Postgres and
search intact.
