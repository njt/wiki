---
url: https://blog.cloudflare.com/engineering-standards-enforcement/
title: "How Cloudflare enforces engineering standards using AI"
author: Cloudflare Blog
date_fetched: 2026-08-06
date_published: 2026-08-05
site: Cloudflare Blog
---

Cloudflare built the **Cloudflare Codex**, a governed set of engineering standards expressed as RFCs (using SHOULD/MUST from RFC 2119), with a dedicated governance model that divides the corpus into domains (architecture, security, reliability, TypeScript, Rust, etc.) each led by a domain owner. RFCs go through rounds of review, and enforcement is gated behind a separate "enforced" lifecycle state that gives teams time to absorb new requirements before they become blocking.

The Codex feeds multiple AI agents. The **AI code reviewer** has flagged nearly 230,000 violations and blocked 16,000 merges since inception earlier this year. It retrieves relevant RFC statements (compacted into JSON with stable slug identifiers for tracking) and applies them based on SHOULD vs. MUST severity and the RFC's lifecycle state. Cloudflare also provides custom linter packages (TypeScript+oxlint first, Rust in development, Go planned) for millisecond-level enforcement of language-specific rules, and a local CLI that runs the same AI reviewer without the CI round-trip.

The **spec reviewer** evaluates technical design documents against Codex requirements before implementation begins. Running on Cloudflare's Developer Platform (Workers, D1, AI Gateway, Cron Triggers), it has reviewed nearly 600 unique specs since May 2026 across 3,200+ invocations. The **incident report reviewer** applies the same approach to postmortems, with 200+ reports assessed and mandatory review for high-severity incidents.

The architecture uses progressive disclosure: a purpose-built agent extracts and compacts SHOULD/MUST statements into JSON metadata, so consuming agents retrieve only the statements — not full RFC bodies — unless additional context is needed. This avoids context-window bloat across 60+ RFCs. Future plans include expanding beyond engineering to product, security, and compliance domains.
