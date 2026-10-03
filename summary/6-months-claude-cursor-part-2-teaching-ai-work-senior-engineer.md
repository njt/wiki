---
url: https://www.telerik.com/blogs/6-months-claude-cursor-part-2-teaching-ai-work-senior-engineer
title: "6 Months of Claude & Cursor, Part 2: Teaching AI to Work Like a Senior Engineer"
author: Jefferson S. Motta
date_fetched: 2026-10-03
date_published: 2026-09
topics:
  - agent-coding-workflow
  - claude-code
---

Jefferson S. Motta's second installment of his six-month Claude-and-Cursor field report. Part 1 built skills so the AI understood his project; Part 2 is the harder problem — building a skill so the AI understands *him*. His diagnosis: the persistent failure was posture, not capability. With 30+ years of experience, he kept being treated like a beginner — the AI suggested removing intentional code, offered A/B/C option lists when he already knew the answer, and apologised for paragraphs when shown to be wrong.

The fix is a personal skill, `jefferson-senior-dev`, mined from weeks of reviewing his own conversation history. Three failure patterns emerged: underestimation (suggesting his deliberate instrumentation was unnecessary), premature diagnosis (flagging errors where he'd already looked), and the multiple solution. The skill encodes rules of engagement: assume shared code is correct until there's clear evidence otherwise, don't invent caveats to look useful — with one carved-out exception for security issues, and one deliberate rule about his own known blind spot (inverted boolean conditions), where questioning him *does* add value. "A good skill teaches AI when to disagree with you."

The results: five minutes of convincing per session went to zero, and freed attention surfaced real catches like fire-and-forget discards of awaited calls. The same philosophy produced `cursor-prompt-craft` for Cursor — prescriptive prompts with exact paths and explicit "do not" sections beat exploratory ones. By July the skills ran unadjusted, and skills graduated from context, to posture calibration, to *units of work*: a CLI parameter fired the same fix-as-skill across six of seven platforms, builds green — with the caveat that a green build is not a validated fix; end-to-end testing stays human. He also candidly logs his own mistakes: a too-broad cleanup instruction that replaced `??` operators with dashes, parallel pipeline sessions corrupting reports, and CSV statistics revealing a "flaky" failure was deterministic (x86 vs AnyCPU compile). Calibrated AI accelerates whatever you told it to do, including the wrong thing.
