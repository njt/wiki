---
url: https://gist.github.com/4213ae0471f35801e28267cbf058c76f
title: "Beyond the Warehouse: Data Stacks That Actually Work"
author: Thomas in 't Veld (Tasman Analytics)
date_fetched: 2026-09-13
date_published: unknown
topics:
  - databases-and-data
  - software-engineering-craft
---

A conference talk (AI & Data Summit, captured as a ytx YouTube transcription gist; undated, internal evidence points to ~2026) by Thomas in 't Veld, founder of Tasman Analytics, which builds data stacks for fast-growing startups and scale-ups — ~60–70 client engagements over six years. Opening with VentureBeat's "85% of data projects fail" as a hook, he diagnoses three recurring failure modes: over-designing the stack, underestimating data modeling, and failing at activation.

The technical core: an analytics stack is not a production engineering stack (duplicates that production can tolerate would double-count revenue; event streams are not logs, so tracking plans and event governance matter); ingestion should be prioritized by ranked business insights (revenue reporting first — RevenueCat data before anything else); data problems should be fixed at source, not repaired in the model. The centerpiece is a "narrow waist" domain model — entities (users, transactions, subscriptions, campaigns) modeled once in the middle of the stack, combining Inmon-style entity modeling in the warehouse with Kimball-style dimensional marts — as a static target that survives growth and vendor swaps (HubSpot→Salesforce changes only extraction logic). Data engineering is "25, 30% actually building the pipelines and it's 70, 75% building the testing suite," run through CI/CD, with infrastructure-as-code all the way down.

Three takeaways frame the talk: prioritize stack choices by business value; don't underestimate data modeling; and build semantic layers — defined once (entities, dimensions, join rules, canonical metrics), version-controlled, enforced in CI/CD — which he calls "critical to do anything with AI in analytics." Three build paths: homegrown text files, dbt's semantic layer, or Omni (built by the old Looker team). In Q&A he blesses Databricks' native metrics as pragmatically fine, but holds one zero-tolerance line: "If you end up with two different types of semantic layers, you've done it wrong."

Activation is named the #1 problem even in successful builds — companies spend heavily and still see marketing teams export raw data into Excel. His four pillars: presentational models linked to decisions, very short feedback loops, treating every insight as a data product with scope and acceptance criteria, and internal analytics measuring whether anyone actually uses the dashboards. On security, his sharpest agent-relevant claim: never let agents query raw data — PII exposure, prompt injection and SQL injection live at the raw layer, and the well-designed data model is the security control. The gist includes the ytx summary, tools/practices lists, an unusually self-critical "Unanswered Questions" section, and the full transcript with Q&A (data mesh deflected, dbt alternatives teased but never named, burn-it-all advice for legacy event data, "keep receipts" for stakeholder definition politics).
