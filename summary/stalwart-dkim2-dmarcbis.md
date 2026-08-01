---
url: https://stalw.art/blog/dkim2-dmarcbis/
title: "DKIM2 and DMARCbis Have Landed, and Stalwart Speaks Them First"
author: Mauro D. (Project Maintainer)
date_fetched: 2026-07-11
date_published: 2026-07-06
---

Stalwart v0.16.12 is the first mail server to ship both DKIM2 (draft-ietf-dkim-dkim2-spec) and DMARCbis (RFCs 9989, 9990, 9991, published May 2026). The post explains both protocols in detail, why they were needed, and how they work — accompanied by a browser-based playground that compiles the same Rust library to WebAssembly.

**DKIM2** fixes six DKIM1 failures: replay attacks, forwarding breakage, missing chain of custody, ARC's flawed patchwork approach, inconsistent header canonicalization, and untrusted bounce backscatter. It splits signing into two headers: `Message-Instance` (revision counter, hashes, and reversible "recipes") and `DKIM2-Signature` (SMTP envelope, sequence number, flags). Replay is blocked because each signature records `MAIL FROM` and `RCPT TO`, and consecutive hops must line up. Forwarding survivability comes from base64-JSON recipes that tell a verifier how to undo changes and reconstruct the prior state — a mailing list adding a subject tag and footer can be reversed so the originator's signature still verifies. Bounces become provably safe: a DSN must target the signed return path, be signed itself, and carry the original message for re-verification.

**DMARCbis** splits the monolithic DMARC1 spec into three Standards Track RFCs (core, aggregate reporting, failure reporting). Its headline change is DNS Tree Walk, which replaces the volunteer-maintained Public Suffix List with a deterministic DNS query capped at eight hops. Three new tags arrive: `np` (policy for non-existent subdomains), `psd` (public suffix declaration), and `t` (a testing flag that softens enforcement). The `pct`, `rf`, and `ri` tags are removed as unused. The `p=reject` policy is recast as a strong signal rather than a blind trigger — receivers must not reject on policy alone, and sending domains on `p=reject` must DKIM-sign. Reporting adds a new XML namespace, a `discovery_method` field, and requires DKIM results to name the selector.

Both protocols are implemented in the open-source Rust crate [`mail-auth`](https://github.com/stalwartlabs/mail-auth).
