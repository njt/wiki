---
url: https://www.anthropic.com/research/redesigning-source-control
title: "Redesigning source control for agent scale"
author: Anthropic
date_fetched: 2026-10-24
date_published: 2026-10
topics:
  - developer-tools
  - agent-coding-workflow
---

Anthropic reports that coding is no longer its bottleneck — developer infrastructure is. Daily merges in its monorepo grew from ~2,500 (May 2026) to over 30,000 (September 2026), driven almost entirely by agents. Git's distributed, offline-first design, built for humans on laptops, became the constraint: ~100 GB of main-branch history, 25 million checkouts a day, and at peak several hundred internal git mirrors straining to keep up.

The answer is Scion, an in-house version control system built and fully migrated to in five months by a seven-person team (who also shipped the Antique code review tool in the same window). Scion inverts git's assumptions on six principles: clients are always online; client-server asymmetry (the server computes, the client caches); every source control query exposed as an API so tools need no local repo; fully horizontal scalability with no packfiles and no fully materialized copy anywhere; cheap, lock-free, server-side commits whose cost scales with the change, not the repo; and optional, sparse-by-design working copies. Changesets — write-ahead-log entries recording only changed files — replace commits; derived data (blame, path history, skip lists) is computed recursively and transparently. The cache-tiered fleet (scion-daemon → scion-cache → scion-server over Spanner) runs at 99%+ hit rates and is 200x smaller than the git mirroring infrastructure it replaced; CI is two orders of magnitude faster and realized a 25% CPU capacity win.

The lessons cut both ways. Claude rewrote the hand-written git ingestion pipeline 20x more memory-efficient and 4x faster with no instructions; engineers supplied design tradeoffs and, crucially, the word "no" when agents violated system constraints or over-engineered in response to adversarial review. Merge queues "don't make much sense at agent scale," so Anthropic leans on post-merge and pre-deploy testing instead. Claude Tag-powered "superfans" — agents obsessively monitoring single components in Slack channels — detect issues traditional alerts miss. Scion is explicitly not offered as a git replacement; Anthropic's contribution is the principles, offered to GitHub, GitLab and the OSS community for scaling git server-side.
