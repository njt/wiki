---
url: https://ai.engineer/code/2026/talks/online-ayush-cli
title: "Online: Ayush CLI — Why Your Agent Secretly Hates Your CLI"
author: Ayush (talk transcript)
date_fetched: 2026-11-26
date_published: 2026
topics:
  - mcp-and-tool-protocols
  - developer-tools
---

Ayush built a CLI (chat-history) that lets coding agents search their own past sessions across Claude Code, Codex and Cursor. Studying his own transcripts turned him into an agent-UX researcher, and this ai.engineer talk is the resulting field report: agents are your tool's real users, and the evidence of what your tool is missing is sitting in the transcripts.

His core method is the "pipe test": watch what the agent does immediately after calling your tool. Across ~20,000 extracted agent calls touching 14 CLIs, a quarter fed output through head/grep/tail/python — and 83% of pipes were the same cut-down pattern repeated against the same subcommand. That repetition is the bug report: each pipe is a guess about where the answer lives, and a size cap doesn't identify relevant output. Piping is also dangerous in bash — without pipefail, `cargo test | head` can report success on a compile error, and a `tail` can swallow the error so it never reaches the model. Advice: fail loudly, with a short summary and path to the full log in the last lines.

The fixes he shipped: selection flags (view around a match, only matching messages), compact JSON modes (~6x smaller), one call instead of an agent's for-loop over sessions. Once the skill file taught the new flags, piped reads fell from 82% to a quarter. Corroborating cases: Sanity's MCP server got zero bug reports from 3M tool calls until they shipped a feedback tool; InfraCost moved filtering into their CLI plus a skill and went from 0/6 to 6/6 on hard benchmark questions with 41% lower cost; DuckDB's agent mode cut model-read output 59%.

His closing warnings: agents take skill.md and help text literally (stale docs are bugs — a no-op `deep` flag was still passed in 95/95 searches), transcripts are the product analytics nobody reads, and agents are good at iterating on a tool once it exists but bad at designing it one-shot — "you decide what the tool should be and let the agent check against what really happened."
