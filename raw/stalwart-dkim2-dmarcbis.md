---
url: https://stalw.art/blog/dkim2-dmarcbis/
title: DKIM2 and DMARCbis Have Landed, and Stalwart Speaks Them First
author: Mauro D. (Project Maintainer)
date_fetched: 2026-07-11
date_published: 2026-07-06
---

# DKIM2 and DMARCbis Have Landed, and Stalwart Speaks Them First

**Author:** Mauro D. (Project Maintainer)
**Publication Date:** Jul 6, 2026
**Reading Time:** 27 min

---

## Overview

The post announces that two major email authentication protocol updates — **DKIM2** (draft -04) and **DMARCbis** (RFCs 9989, 9990, 9991, published May 2026) — are now fully implemented in **Stalwart v0.16.12**, making Stalwart the first mail server to support either protocol. A browser-based [mail-auth playground](https://mail-auth.stalw.art) lets anyone test them without installing anything.

---

## DKIM2 (draft-ietf-dkim-dkim2-spec)

### What DKIM1 Did

DKIM lets a domain sign a message with a private key; the public key lives in DNS at `selector._domainkey.domain`. A valid signature means "this domain vouches for this content."

### Issues with DKIM1

Six key problems drove the redesign:

1. **Replay attacks** — A captured signed message can be replayed to millions of recipients; every copy still verifies on the originator's reputation.
2. **Forwarding breaks signatures** — Mailing list modifications (subject tags, footers) cause signature failure, indistinguishable from malicious tampering.
3. **No chain of custody** — "Received" and "Return-Path" headers are unauthenticated and trivially forgeable.
4. **ARC as a flawed patch** — It required a separate trust fabric that never materialized at internet scale.
5. **Inconsistent header signing** — Variable canonicalization modes left exploitable gaps.
6. **Backscatter from untrusted bounces** — DSNs could land on innocent third-party domains.

### How DKIM2 Works

DKIM2 splits the old single header into two:

- **`Message-Instance`** — Carries a revision number, cryptographic fingerprints of headers and body, and (when content changes) a reversible "recipe."
- **`DKIM2-Signature`** — Records the SMTP envelope (`MAIL FROM`, `RCPT TO`), a sequence number, and flags. The envelope inside the signature is what creates a verifiable chain.

Two counters drive the system: one for signatures (per hop), one for message revisions (only when content changes).

### Key Mechanisms

**Replay prevention** — Each signature captures `mf=` (MAIL FROM) and `rt=` (RCPT TO). Consecutive hops must line up: one hop's sending domain must match the prior hop's recipient domain. A `donotexplode` flag forbids fan-out; `exploded` marks legitimate fan-out.

**Recipes (forwarding survivability)** — A base64-encoded JSON recipe (`r=` tag) tells the verifier how to reverse changes and reconstruct the prior message state. It has optional `"h"` (header) and `"b"` (body) parts. Header instances are counted **bottom-up**; body lines **top-down**. An empty step list means "remove every instance." `{"b":null}` marks content as genuinely unreconstructable (e.g., contractual redaction).

**The example given:** Alice sends a message → a mailing list adds `[list]` to the subject and appends a footer → the list records a recipe reversing both changes → Bob's server applies the recipe, reconstructs Alice's original, and verifies her signature passes.

> "A footer tweak and a malicious body swap no longer look the same."

**Bounce trust** — DKIM2 ignores `Received` and `Return-Path`. The authoritative return address comes from the signed `mf=` tags. A DSN must be addressed to the highest-numbered hop's `mf=`. A DKIM2 bounce is itself a DKIM2-signed `multipart/report` carrying the original message, enabling full re-verification.

> "Pass all three and the bounce is provably about a message you really sent."

This is what makes deferred bounces safe for the first time — a provider can accept a message, later decide it's spam, and send a signed bounce back along the recorded path with no risk of backscatter.

**ARC absorbed** — DKIM2 eliminates the need for ARC's separate trust fabric. Recipes let verifiers undo changes and recheck the originator's own signature. The IETF's DMARC working group is moving to reclassify RFC 8617 as Historic.

---

## DMARCbis (RFCs 9989, 9990, 9991)

### What DMARC Does

DMARC ties authentication to the visible `From:` domain via **identifier alignment** — a DKIM or SPF result only counts if it aligns with the `From:` domain. Domain owners publish a policy (`none`, `quarantine`, `reject`) in DNS.

### Issues with DMARC1

1. **Public Suffix List dependency** — A static volunteer-maintained file sat on the critical path of global mail authentication, with no mandated refresh cadence.
2. **`pct` tag dysfunction** — Only 0 and 100 ever worked reliably despite being designed as a percentage dial.
3. **Phantom subdomain spoofing** — Attackers could forge mail from unregistered subdomains.
4. **Monolithic specification** — Core protocol, aggregate reporting, and failure reporting couldn't evolve independently.

### What Changed

**One RFC becomes three:** RFC 9989 (core protocol), RFC 9990 (aggregate reporting), RFC 9991 (failure reporting) — all moving from Informational to Standards Track.

**DNS Tree Walk replaces the Public Suffix List.** The receiver queries `_dmarc.<author-domain>`, then drops labels leftward toward the root until it finds applicable records. The algorithm is capped at **eight queries** — for names longer than 8 labels, it strips multiple labels at once so only 7 remain, then proceeds one label at a time.

The Organizational Domain is selected by a deterministic rule set:
1. A record with `psd=n` marks its own domain as the Organizational Domain.
2. A record with `psd=y` (not at the starting point) means the OD sits one label below.
3. Otherwise, the record with the fewest labels wins.

> "The Organizational Domain is now defined by DNS records that the domain owner controls."

### Tag Changes

| Tag | Status | Purpose |
|-----|--------|---------|
| `np` | **New** | Policy for non-existent subdomains (NXDOMAIN per RFC 8020) |
| `psd` | **New** | Public suffix declaration (`y`/`n`/`u`) |
| `t` | **New** | Testing flag — softens enforcement without disabling reporting |
| `pct` | Removed | Percentage sampling (only 0 and 100 worked) |
| `rf` | Removed | Failure report format (only one format deployed) |
| `ri` | Removed | Aggregate report interval (effectively fixed at ~1 day) |
| `v, p, sp, adkim, aspf, fo, rua, ruf` | Unchanged | All legacy tags remain valid |

The **`t` flag** steps policy down one level: `reject` → `quarantine`, `quarantine` → `none`. Reports still flow normally.

### p=reject Nuance

> "The policy became a strong signal to weigh rather than a trigger to pull blindly."

Three rules: (1) Sending domains SHOULD NOT publish `p=reject` if users might post to mailing lists; (2) any domain using `p=reject` MUST apply valid DKIM signatures (not rely on SPF alone); (3) receivers MUST NOT reject solely because a policy says `reject`.

### Reporting Upgrades

**RFC 9990** — New XML namespace `urn:ietf:params:xml:ns:dmarc-2.0`, extensibility slot, `discovery_method` field distinguishing `psl` from `treewalk`, DKIM results now require naming the selector. Destinations still prove consent via `_report._dmarc`.

**RFC 9991** — Updated from RFC 6591, adds a dedicated `dmarc` failure type and required `Identity-Alignment` field. Public suffix domains must not act on `ruf` without a specific agreement.

### What Stayed the Same

Alignment (relaxed/strict via `adkim`/`aspf`), the three policies (`none`/`quarantine`/`reject`), evaluation of only the `From:` header, and backward compatibility — "Records written for RFC 7489 remain valid."

---

## Availability in Stalwart

Both protocols ship in **Stalwart v0.16.12**. The underlying Rust library, [`mail-auth`](https://github.com/stalwartlabs/mail-auth), is open source and covers DKIM1, DKIM2, SPF, DMARC, DMARCbis, and ARC. The browser playground compiles the same code to WebAssembly.

> "Being early to a standard is how the standard gets tested against real mail before it hardens."

---

## Tags

dkim, dmarc, dkim2, dmarcbis, email, security, rust, server
