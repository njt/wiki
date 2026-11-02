---
url: https://www.youtube.com/watch?v=L1OBRtfPS34
title: "Stanford CS146S: How always-on agents work"
author: Lee Robinson (leerob)
date_fetched: 2026-11-02
date_published: 2026 (recent)
topics:
  - agent-architecture
  - agent-memory-and-context
---

A 45-minute Stanford CS146S lecture by Lee Robinson (building agent infrastructure at SpaceX, formerly Cursor/Vercel) on how "always-on" agents — bots that live on servers rather than in your terminal — are architected.

He frames four eras of developer-AI interaction (copy-paste ChatGPT → terminal harnesses → dedicated apps → always-on bots) and then walks the building blocks: a thin-client/thick-server split where the client has exactly one tool (`send_to_user`) and everything heavy lives server-side; a Firecracker VM "computer" the agent can boot, snapshot, and sleep; durable workflows via Temporal so crashed work resumes mid-task rather than replaying; and a router with a "do I need to bug the user?" heuristic so the bot behaves like a colleague who knows when to be quiet.

The second half is context engineering: prompt caching as the dominant cost lever (cache misses treated as bugs), compaction done while the cache is still warm, subagents as fresh-context helpers that give the illusion of infinite context, dynamic tool discovery (names only in context, schemas on demand), and layered memory (persistent facts, dated logs, ephemeral scratchpads, everything else in greppable files). He closes with a training flywheel (inner eval loop, outer model-training loop), a "delete the product" philosophy, and open problems: evaluating month-long agents when models ship monthly, memory representations, security, and the proactivity-vs-annoyance dial.
