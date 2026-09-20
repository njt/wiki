---
url: https://lethain.com/migrations/
title: "Migrations: the sole scalable fix to tech debt"
author: Will Larson
date_fetched: 2026-09-20
topics:
  - software-engineering-craft
---

Will Larson's essay argues that migrations — not local cleanups — are the only scalable mechanism for paying down technical debt, because individual engineers and teams exhaust all the low-hanging fruit and everything left requires many teams moving together. Drawing on his experience running Uber's migration from Puppet-managed services to self-service provisioning, he frames migrations as a way of life at growing companies: most tools support only about one order of magnitude of growth before failing, and the ability to migrate becomes a defining constraint on overall velocity.

The bulk of the piece is a three-phase playbook. **Derisk**: write a design document, shop it with the hardest and most atypical teams first (never the easiest), and embed with them — because every team that signs on is betting the migration will actually finish. **Enable**: build tooling that programmatically migrates the easy ninety percent, then invest in self-service tooling and documentation treated as products, with reversibility as a core property. **Finish**: stop the bleeding by installing ratchets so new code uses the new approach, generate tracking tickets with management visibility, personally finish the long tail, and reserve recognition for completion rather than kickoff — since abandoned migrations poison future ones.

The essay closes on the stakes: companies that don't get good at migrations languish in debt and end up doing a full rewrite anyway.
