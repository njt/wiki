# Reduce Logging Costs — Summary

Michael Shpilt argues logging costs can consume 5–10%+ of cloud hosting spend, and that most logs are redundant and never queried. He presents five strategies: sampling (head and tail, with the trade-off that rare events are exactly what you care about), tiered storage (hot/warm/cold, with cross-tier querying as the friction point), in-code cleanup (removing duplication, aggregating, reducing verbosity — treating telemetry as code), switching to cheaper vendors (solves cost but not noise, and migration is expensive), and the nuclear option of removing INFO-level logs entirely ("almost completely blind in production"). The thesis: sustainable cost reduction comes from reducing unnecessary telemetry before it leaves your application, not from cheaper storage or smarter sampling after the fact. Observability cost optimization is shifting from storage economics to treating telemetry as code requiring continuous maintenance.

---
*Source: [[raw/reduce-logging-costs]]*
