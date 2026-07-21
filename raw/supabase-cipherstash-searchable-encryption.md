---
url: https://supabase.com/blog/searchable-field-level-encryption-with-cipherstash
title: Searchable field-level encryption on Supabase with CipherStash
author: bilharmer
date_fetched: 2026-07-21
date_published: 2026-07-09
site: Supabase Blog
---

# Searchable field-level encryption on Supabase with CipherStash

The post announces the availability of a CipherStash integration for Supabase. CipherStash is described as a Data Level Access Control (DLAC) platform for apps running on Postgres. DLAC extends traditional access controls—which normally operate at row or table granularity—down to individual encrypted values, enforcing policy "at decryption rather than at the query layer."

## How CipherStash Works

Teams can encrypt sensitive fields at the application layer with a unique key per value, enabling searches and joins on encrypted data without decryption. Key management is handled through ZeroKMS, described as "CipherStash's zero-knowledge key management service." The article emphasizes that "because keys never leave your control, neither CipherStash nor Supabase can access your plaintext data."

Setup requires a single CLI command: `npx stash init --supabase`

## The Problem It Solves for Regulated Workloads

The author presents a binary that regulated-data teams typically face:

1. **Traditional field-level encryption** — encrypted values appear as random bytes to Postgres, so `WHERE email = ?` returns nothing, indexes break, and joins fail. Teams must pull all rows, decrypt in-app, and filter in memory.
2. **Skipping encryption** — performance stays good, but breaches expose plaintext sensitive data.

CipherStash offers a third path. Encryption occurs inside the application before data reaches the database. Each value is stored as a "single JSON payload containing the ciphertext alongside Searchable Encrypted Metadata (SEM)." This metadata allows Postgres to filter, sort, and join without enabling recovery of the original value. Queries are converted to SEM inbound; Postgres matches against stored SEM, and only authorized ciphertexts are returned. "Every encrypted value carries a policy stating who can read it and under what condition," enforced at decryption time.

For HIPAA, GDPR, or SOC 2 environments, the claim is that teams keep Postgres, Supabase, and search functionality while benefiting from shorter compliance reviews and reduced breach surface.

## Integration Details

CipherStash Stack works with TypeScript apps using Supabase.js, Drizzle, and Prisma Next. Encryption and decryption happen transparently in the application. Marked columns flow through the SDK while all other columns behave normally — WHERE clauses, fuzzy text matching, ORDER BY, and JSON queries continue functioning over encrypted data.

Keys are managed via ZeroKMS. "Each encrypted value gets its own key, derived on demand," and keys can be split across regions for data residency requirements including FedRAMP and IL4.

For scenarios where the SDK doesn't fit — analytics jobs, admin tooling, background workers in other languages, or any direct database access outside the application — **CipherStash Proxy** provides a secure alternative. It speaks the Postgres wire protocol and handles encryption/decryption transparently, so existing SQL clients and tools work without code changes.

## Call to Action

The post links to the integration page for adding CipherStash to a Supabase project.
