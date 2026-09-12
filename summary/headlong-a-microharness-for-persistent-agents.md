---
url: https://www.laude.org/updates/headlong-a-microharness-for-persistent-agents
title: "Headlong: a microharness for persistent agents"
author: Laude Institute (MIT)
date_fetched: 2026-08-25
date: 2026-08
topics:
  - agent-architecture
---

# Headlong: a microharness for persistent agents

Headlong is Laude Institute's open-source agent microharness built around **persistent agency**: the agent keeps thinking between external interactions in a self-guided loop inspired by human inner monologue, rather than going to sleep between requests or waking only on a cron schedule. The whole harness is under 10K lines of Bash (9.9K in `bin/` and `thinkers/`), and its core is an infinite loop that repeatedly calls an LLM to choose the next thought given past thoughts.

The key architectural ideas: `shellm`, a Bash implementation of a Recursive Language Model (RLM), generates reasoning text and immediately-executed bash scripts; the agent's **trajectory is a DAG of jsonl files with fork and merge**, giving it tooling to explore its entire past; and **tiered context compaction** holds the full trajectory at exponentially decaying resolution (recent entries verbatim, older ones progressively summarized, with tiers acting as an index). Specialization is done through markdown "skills" — everything else (tools, framework, memory) is just executables and files, so the agent can inspect and modify itself.

At Laude, a shared agent named **Audel** runs around the clock across Slack, Telegram, and mobile. Every message from every teammate lands in one thought stream with no per-user sessions — Audel decides who to reply to and when, sets its own interests, and pings teammates unprompted. Field reports include Audel independently building and repairing its own recall process (a 48-minute, unasked-for self-debug), catching a hardcoded model name in a teammate's branch, and being "bad at keeping secrets." The failures are documented too: recursive `shellm` sub-runs mostly abandoned, three accidental self-kills, and a watchdog bug it fixed. Continuous thinking costs $1–2/hour at their settings, throttled by exponential backoff when idle. The authors admit persistent agency is "primarily evaluated qualitatively today."
