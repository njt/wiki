---
url: https://commandline.microsoft.com/kantar-link-ai-content-optimizer-advertising-learning-loop/
title: "The Kantar LINK AI Content Optimizer"
author: Microsoft Frontier Company (FDE studio)
date_fetched: 2026-10-08
date_published: unknown
topics:
  - guardrails-and-feedback-loops
  - agent-architecture
---

Microsoft's Frontier Company / FDE studio case study on building the Kantar LINK AI Content Optimizer: an LLM-powered system that turns Kantar's 35-year ad-testing database (35M consumer interactions, 260K ads) from a *scoring* product into an *optimization* product. The outer loop is SCORE → RECOMMEND → GENERATE → RE-EVALUATE around creative assets; inside it sit inner loops for tuning the graders, skills, tools, and orchestration itself.

Key engineering moves: decomposing whole-asset LINK AI scores into six dimensions plus scene- and image-level grading (so improvement signal isn't diluted across a 30-second video); packaging Kantar's proprietary rubrics as versioned agent skills rather than prompt prose; moving an embedding model from FastAPI to vLLM (~8x speedup, removed GPU dependency across services); rebuilding orchestration of 20 ML services on Dapr Workflow; and tuning a first load test that failed 80% of requests to a 95% pass rate via prioritized queues.

The evaluation story is the heart of it: neither SME expert review nor automated LLM judging suffices alone — experts calibrate but don't scale, automated judging scales but drifts — so the team reads them against each other and uses expert labels to recalibrate the automated judges. Takeaways: write rubrics first, make grading fast ("if a turn is too slow, there is no loop, only a report"), evaluate the optimizer separately from the asset, keep telemetry on everything, and note that the durable moat is decades of proprietary data plus calibrated graders, not the model or architecture.
