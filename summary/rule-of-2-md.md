---
url: https://chromium.googlesource.com/chromium/src/+/main/docs/security/rule-of-2.md
title: "The Rule of 2"
author: Chrome Security Team
date_fetched: 2026-10-11
date_published: 2019-03 (last revised)
topics:
  - security-and-sandboxing
  - software-engineering-craft
---

Chromium's canonical security policy: when handling untrustworthy input, code may pick at most two of three properties — untrustworthy inputs, unsafe (memory-unsafe) implementation language, and high privilege. A CL involving all three will not be approved.

The document grounds the rule in data: a Microsoft study found ~70% of yearly patched vulnerabilities are memory safety issues, and as of March 2019 only ~5 of 130 public Chromium Critical-severity bugs were not obviously memory corruption. It defines the three axes carefully: "untrustworthy input" is any input whose grammar is too complex to parse safely regardless of provenance (so provenance matters almost always); memory-unsafe languages are C, C++, and assembly, with Rust's `unsafe` treated as an unsafe subset requiring expert human review; "high privilege" is relative, running from firmware down to the browser process.

For each way to shed one of the three properties it gives concrete techniques: use an approved memory-safe language (Java, Rust — e.g. the QR code generator and the Rust JSON parser); sandbox the parsing (Site Isolation renderers, sandboxed utility processes like Safe Browsing's ZIP analyzer and the Data Decoder Service); assert provenance cryptographically (pinned TLS, content hashes over untrusted channels); or normalize input into a simpler grammar in a low-privilege process before parsing at high privilege (PNG → `SkBitmap` over Mojo). Structured IPC types with simple grammars (`GURL`, `base::Token`) are audited exceptions, as are RE2 and Protobuf with caveats.

The doc closes honestly: Chromium contains known violations (the unsandboxed Windows network process), the top priority is no *new* violations, and existing ones are mitigated over time — e.g. swapping the browser process's C++ JSON parser for a Rust one.
