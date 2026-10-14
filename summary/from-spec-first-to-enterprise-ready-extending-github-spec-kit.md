---
url: https://developer.microsoft.com/blog/from-spec-first-to-enterprise-ready-extending-github-spec-kit/
title: "From Spec-First to Enterprise-Ready: Extending GitHub Spec Kit"
author: Microsoft Developer Blog (GitHub Spec Kit team)
date_fetched: 2026-10-14
date_published: 2026 (exact date not stated in source)
topics:
  - specifications-as-the-product
  - coding-agents-and-frameworks
---

This is the follow-up to Microsoft's foundational Spec-Driven Development article, aimed at the enterprise adoption problem: different teams have different stacks, domains, and compliance obligations, and forking the toolkit per team would fragment the ecosystem. The answer is a stable core with four explicit extension layers.

The four layers: **presets** are stackable, priority-ordered overrides of templates, commands, prompts, and terminology (a security preset strengthens threat-modeling prompts; a domain preset injects terminology and mandatory acceptance criteria); **extensions** add wholly new capabilities beyond the built-in lifecycle, such as a security-review command or an accessibility quality gate; **bundles** package presets and extensions into one versioned, installable unit with pinned versions and provenance tracking; **workflows** chain commands, prompts, scripts, and human checkpoints into repeatable processes for things like bug remediation and release readiness — with the stated goal of making the expected sequence explicit, not removing human judgment.

Distribution is governed through a two-tier catalog model: an organization-controlled catalog with review, provenance, and prompt-injection safeguards gets highest resolution priority, while the community catalog is discovery-only by default — "a listing confirms format and metadata — not a security review or endorsement." The article applies the same layering to the project constitution (organization baseline → domain overlay → team customization) and closes with adoption advice: keep customization modular and the core untouched, so local improvements don't become long-lived forks.

A notable example: token-usage visibility as an extension — collecting usage per run, surfacing trends, and triggering optimization guidance at thresholds, framed as one of several cross-cutting concerns (accessibility, secure development, dependency governance) that become adoptable without touching the SDD lifecycle.
