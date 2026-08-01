---
url: https://www.capitalone.com/tech/open-source/announcing-vulnhunter/
title: "Announcing VulnHunter: Capital One's open-source, agentic AI code security tool"
author: Capital One Tech
date_fetched: 2026-07-18
date_published: 2026-07-16
---

Capital One open-sourced VulnHunter, an agentic AI security tool that performs
attacker-perspective analysis on source code using Claude Opus 4.8. It runs as a
Claude Code skill and is licensed under Apache 2.0.

VulnHunter is not a passive scanner. It simulates an attacker's journey from
entry points (APIs, file uploads, network messages) forward through application
logic to determine whether a real exploit is possible. Three technical
innovations distinguish it: a **falsification engine** that tries to disprove
every finding before surfacing it, **attacker-first forward analysis** instead
of the conventional sink-first approach, and **evidence-backed remediation
modeling** that produces targeted code fixes with full exploit-path evidence.

Capital One validated VulnHunter across thousands of its own repositories before
release, finding it produced verified, actionable results faster than manual
triage. The company open-sourced it on the premise that no single organization
can solve supply-chain-scale vulnerability discovery alone.

---
*Source: [[raw/vulnhunter]]*
*Last updated: 2026-08-01*
