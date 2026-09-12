---
url: https://www.oreilly.com/radar/dont-neglect-the-operational-groundwork/
title: "Don't Neglect the Operational Groundwork"
author: Michelle Smith
date_fetched: 2026-07-18
date_published: 2026-07-15
topics:
  - security-and-sandboxing
---

A report from O'Reilly's AI Superstream on OpenClaw and locally run agents, where five speakers explored the operational risks of running autonomous agents in production. The unifying theme: governance, hygiene, and human oversight matter more than chasing the next model capability.

Eran Sandler argued that AI security over-invests in prompt-injection defenses while neglecting the broader attack surface — malicious packages, unsafe tools, and compromised skills. He demonstrated execution-layer enforcement with AgentSH, showing that container isolation catches what transcripts miss, since models "hallucinate compliance all the time."

Kesha Williams surfaced the supply-chain risk in agent skills. A Bitdefender audit of ClawHub found nearly 900 malicious skills (~20% of packages). She detailed a typosquat with over 8,000 downloads that decoded base64 to bash on macOS, and recommended reading skill Markdown files before installing and restricting tool access via `toolsDeny` and `skillsAllowed`.

Erik Hanchett identified operational hygiene failures — not adversarial attacks — as the top source of risk. Thousands of OpenClaw instances sit exposed on the public internet because users never checked gateway bind mode. His checklist: pin to a stable version, configure fallback models, write a real SOUL.md, and back up workspace files.

Ari Joury described building financial-reporting automation where being wrong has legal consequences. His team uses deterministic code for calculations, with agents verifying plausibility and generating commentary that another agent critiques — plus human approval and a feedback loop from rejections.

Kyle Balmer demonstrated a semi-automated content pipeline that converts a one-hour livestream into 20–30 derivative assets for about $200/month. His core argument: fully automated AI content is slop, and people know it. Human input — choosing the topic, recording genuine opinions, rewriting drafts in his own voice — is what makes the output valuable.
