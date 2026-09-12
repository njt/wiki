---
url: https://weli.dev/blog/the-valley-of-webhooks/
title: The Valley of Webhooks
author: weli.dev
date_fetched: 2026-08-06
topics:
  - software-engineering-craft
---

# The Valley of Webhooks

A practitioner's account of building the same webhook-based data replication system three times across three companies, and the belated realization that webhooks — notifications designed to trigger side effects — are structurally the wrong primitive for keeping a local copy of a provider's data. The author traces how an "afternoon of work" reliably expands into a multi-week stack of signatures, dedup tables, ordering buffers, bootstrap importers, and a reconciliation cron that exists as a written confession of lost trust. The piece frames webhooks-for-replication as a local optimum in the fitness landscape: a design that's better than its immediate alternatives, so the industry has spent fifteen years paving the valley floor with excellent tooling (Svix, Hookdeck, EventBridge, Fivetran, Airbyte) rather than climbing toward the higher peak — a consumer-initiated, cursor-addressed change log. The author proposes SCROLL, a draft protocol that flips the arrow from provider-push to consumer-pull, and asks whether twenty lines of code could replace the entire stack.
