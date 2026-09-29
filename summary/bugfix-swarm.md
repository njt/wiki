---
url: https://github.com/taylorsatula/bugfix-swarm
title: "bugfix-swarm — a read-only bug-hunt swarm for Pi"
author: taylursatula
date_fetched: 2026-09-29
date_published: unknown (repository, active through 2026-09)
topics:
  - agent-orchestration
  - guardrails-and-feedback-loops
---

bugfix-swarm is a Pi package (one extension + seven skills + four global subagent definitions) that runs a human-gated, read-only static-analysis bug hunt over a codebase. A funnel: a short human kickoff opens the run; waves of hunter and adversarial-verifier subagents each write one report file to disk and return only a schema-validated envelope; a dedicated dedupe agent merges the whole corpus into one `BUG_REPORT.md`; the orchestrator slices it into repair buckets; the human decides every repair at a `checkbox_picker`; executor agents fix accepted rows and verify by live execution, not by re-reading.

The whole thing is mostly prose, not code — ~3,100 lines of markdown doctrine versus a 22-line TypeScript extension that registers the skills and the checkbox-picker tool. The runbook (`skills/bugfix-swarm/SKILL.md`, 799 lines) is a nine-phase pipeline with explicit gates: no picker before verifier verdicts land, no fix dispatch before a human accepts the row, commits only when the human asks. State discipline is the real engineering: conclusions live in the kata issue tracker (one ticket per finding, typed evidence required to close, a permanent monotone "refuted" set), raw evidence lives in per-pass report files the orchestrator never reads, and a handoff skill defines the five facts that exist only in conversation so a run can survive multiple context-window compactions.

Three design choices stand out. First, information-state design for passes — blind hunters, primed verifiers, blind third passes, adjudicators, rechecks — with a convergence bar (two passes agreeing, at least one frame-independent) before any finding reaches a decision. Second, the read/write boundary: investigation agents change nothing; only a human decision moves a site from read to write. Third, framing the scarce resource as human attention rather than tokens: runs are overnight, unmetered, and paced so the human never blocks on agents and agents never block on the human except at gates.

The repo also carries a filled example workflow script (`examples/census-hunt.example.js`) from a real census — 28 hunters, 21 verifiers, 133 findings over a ~90k-LOC tree — with fail-loud sanity checks on the baked preamble, learned after an args-propagation failure silently voided a wave.
