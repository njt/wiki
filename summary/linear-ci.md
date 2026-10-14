---
url: https://starsling.dev/blog/linear-ci
title: "Linear's CI Speedup, Remixed for Agents"
author: StarSling team
date_fetched: 2026-10-14
date_published: 2026-10
topics:
  - agent-coding-workflow
  - developer-tools
---

StarSling's post is a derivative work with a twist: it takes Linear's well-known write-up of how they made CI 15% faster while the test suite grew 4x, and repackages each of the 15 optimizations as a copyable prompt for a coding agent, backed by StarSling's `ci-speedup` skill (70+ catalogued CI optimizations). "Just send this post to your agent" — the artifact is designed to be consumed by a model, not a human.

Linear's actual results: PR wait time down from 6+ minutes to just over 5, runner time per test roughly halved, against a test suite that quadrupled. The gains came in four buckets — infrastructure and tooling (third-party runners 34% faster, tsgo cutting tsc 73%, lint rules rewritten to drop type information for a 68% lint speedup), critical-path gating (fetch-depth caps taking a 94s gate to 20s, moving cache-marker writes off the merge path), repeated setup (a CI base image preinstalling Postgres, pnpm filtered installs down from 44–73s to 16–18s, deleting a node_modules cache that was slower than rebuilding, consolidating seven jobs into two for ~87,000 runner-minutes/month), and test execution (8 shards, plus an opt-in `isolate: false` Vitest project worth ~17% of monthly savings — also the highest correctness risk).

The most agent-relevant detail: Linear notes that "agents now write the majority of our tests", so they updated their agent skills to encode the performance opt-in constraints — the optimization loop feeds back into the prompts that generate the code. StarSling's framing is that CI speed is never finished: re-run the skill against your latest runs and it names the next bottleneck with numbers from your own history.
