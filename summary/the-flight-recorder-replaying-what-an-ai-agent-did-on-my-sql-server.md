---
url: https://blobeater.blog/2026/10/09/the-flight-recorder-replaying-what-an-ai-agent-did-on-my-sql-server/
title: "The flight recorder: Replaying what an AI agent did on my SQL Server"
author: Ade (Blobeater)
date_fetched: 2026-10-14
date_published: 2026-10-09
topics:
  - guardrails-and-feedback-loops
  - agent-architecture
---

The author (of the AgentDBA SQL Server health-check agent) demonstrates the agent's "flight recorder": every decision in a run is persisted as a row in the agent's own database, recording who made it (code or LLM), the stated reason, the action taken, and the timing. A single SQL query replays the whole run.

The worked example is one full health check in 11.2 seconds: six decisions, four by the model. The first two steps (db_status, error_log) are hard-coded gates executed by code, not the model — if either had returned CRITICAL the run would have escalated with no model involvement. The model then chose within a fixed boundary: three remaining checks, each allowed once, plus one verdict. The final verdict was CRITICAL, closed by the model.

The telling number: the five SQL checks took 413 ms combined; the other ~10.8 seconds was the model deciding what to do next, at roughly 2.5–3 seconds per decision. Separately, each tool step carries an `audit_log_id` pointing to both the raw resultset SQL Server returned and the compact summary the model was actually shown — the model never sees raw rows, but a reviewer can audit the conclusion against the data the tool read.

The stated motivation: "I did not want to ask a DBA to take an AI agent's word for anything on a production server." The recorder is the trust mechanism — one row per decision, in your own database, readable with a SELECT.
