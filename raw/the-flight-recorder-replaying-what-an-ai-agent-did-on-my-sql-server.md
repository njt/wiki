---
url: https://blobeater.blog/2026/10/09/the-flight-recorder-replaying-what-an-ai-agent-did-on-my-sql-server/
date_fetched: 2026-10-14
---

“What exactly did it do on my server?” is the first question a DBA should ask about any AI agent. AgentDBA answers it from its own records: every decision in a run is written down with who made it, the reason given, what came back and how long it took.

I call it the flight recorder. This post replays one complete health check from those records, using two queries against the agent’s own database.

Read more: The flight recorder: Replaying what an AI agent did on my SQL Server## One health check, 11.2 seconds

The first result set is one row per run. This run was a full health check on local server (lab).

| What the row records | This run | 
|---|---|
| Trigger | `health_check` | 
| Model | `gpt-6.1-sol` | 
| Decisions | 6, of which the model made 4 | 
| Route | db_status (HIGH) > error_log (LOW) > backup_failures (CRITICAL) > failed_jobs (HIGH) > log_file_health (MEDIUM) | 
| Outcome | Concluded at CRITICAL, closed by the model | 

Two columns matter before any detail. One says how many of the six decisions the model made. The other says who closed the run. An agent’s record should always separate what the AI chose from what code chose for it.

## Six steps, replayed

The second result set is the recorder itself: one row per decision, in order.

| Step | At (ms) | Decided by | Action | Reason given | 
|---|---|---|---|---|
| 1 | 22 | code | db_status | Health check Phase A gate. | 
| 2 | 67 | code | error_log | Health check Phase A gate. | 
| 3 | 2,698 | LLM | backup_failures | Test whether the previously reported backup RPO breaches remain present | 
| 4 | 5,253 | LLM | failed_jobs | Test whether previously disabled jobs remain disabled… | 
| 5 | 8,220 | LLM | log_file_health | Check current log usage, truncation blockers, and VL… | 
| 6 | 11,227 | LLM | CONCLUDE | Concluding: AREA51-AKS remains in breach of back… | 

**Steps 1 and 2 were not the model’s call.** Every health check opens with database status and the error log, and code makes both calls. Had either come back CRITICAL, the agent would have escalated straight away with no model involved. They came back HIGH and LOW, so the run carried on.

From step 3 the model chose, inside a fixed boundary**.** It had three checks left, each allowed once, and then one verdict to write. It controls the order and the wording, and it has to state a reason with every action.

## Where the 11.2 seconds went

The five SQL checks took 413 milliseconds between them. Almost all of the other 10.8 seconds was the model deciding what to do next: four decisions at roughly two and a half to three seconds each. That split matters to a DBA.

## From the verdict back to the evidence

Each tool step carries an `audit_log_id`. It points to one stored row holding two things: the raw resultset SQL Server returned, and the compact summary of it that the model was shown.

Here is the raw resultset stored behind step 3, audit ID 22016.

The model never sees the raw resultset. Each tool reduces its rows to a short, structured summary in ordinary code, and only that summary reaches the model. The raw rows are kept in the audit log, so a reviewer can check the agent’s conclusion against the data the tool actually read.

So a run can be followed end to end: the conclusion, every step that led to it, who decided each step, the reason the model gave, and the data behind it.

## Trust or audit

I did not want to ask a DBA to take an AI agent’s word for anything on a production server. The recorder is my answer: one row per decision, kept in your own database, readable with a SELECT.

AgentDBA has a free evaluation at agentdba.ai if interested.
