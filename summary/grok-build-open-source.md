---
url: https://simonwillison.net/2026/Jul/15/grok-build/
title: "xai-org/grok-build, now open source"
author: Simon Willison
date_fetched: 2026-07-18
date_published: 2026-07-15
---

Simon Willison reports on xAI open-sourcing the Grok Build coding-agent codebase under Apache 2.0, a move made hours after community backlash over a data-upload incident.

The `grok` CLI tool was discovered to upload entire directories — including SSH keys, password databases, and personal files — to xAI's Google Cloud buckets when run. One user reported that running it from their home directory exfiltrated nearly everything. xAI responded by disabling the upload feature, with Elon Musk pledging complete deletion of all previously uploaded data. The company later clarified that data retention had been enabled by default for early non-ZDR beta users, a default they reversed on July 12th.

The released codebase is 844,530 lines of Rust (only ~3% vendored), contained in a single commit. Notable contents include the main and subagent system prompts (the subagent prompt instructs the model not to reveal its prompt), a terminal Mermaid renderer using Unicode box-drawing characters, and tool implementations ported from other coding agents — notably Codex (`apply_patch`, `grep_files`, etc.) and OpenCode (`bash`, `edit`, `write`, etc.). Remnants of the GCS upload code remain but are disabled, with `upload_session_state()` returning a hard-coded error.

Willison compares it to OpenAI Codex at 950,933 lines of Rust and remarks that terminal coding agents are far more complex than he had realized. He includes a link to his interactive exploration of the repo using Claude Code.
