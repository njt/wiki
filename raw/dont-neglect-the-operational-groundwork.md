---
url: https://www.oreilly.com/radar/dont-neglect-the-operational-groundwork/
title: "Don't Neglect the Operational Groundwork"
subtitle: "What the OpenClaw Superstream taught us about running agents safely in production"
author: Michelle Smith
date_fetched: 2026-07-18
date_published: 2026-07-15
---

The article opens by noting that autonomous agents are evolving faster than governance frameworks can keep pace. At O'Reilly's AI Superstream on OpenClaw and locally run agents, five speakers explored challenges around risky third-party extensions, hallucinated compliance, tangled codebases, cost overruns, and supply chain attacks. Host Alistair Croll observed that "we can get better and better with nondeterministic technology, but we'll never be 100% certain it's working." The harder systems are to inspect, the more governance matters — work that is unglamorous, invisible to users, and arguably more vital than any near-term model capability upgrade.

## Secure the action your agent takes at the execution layer

Eran Sandler of Canyon Road listed common compromise vectors: prompt injection, malicious files, unsafe tools, compromised packages, installed skills, and model mistakes. He argued that most AI security fixates on the first and ignores the rest, noting that "guarding the input box does not guard the action."

His recommended approach is enforcement at the execution layer — the boundary between agent intent and the OS. Container isolation limits blast radius but "doesn't make judgment calls," he said. To demonstrate, he installed a simulated malicious package bundled with a routine task and then queried AgentSH's deny log. The log revealed an attempted skill mutation, a blocked external domain call, and reads of .env secrets and SSH keys. "Transcripts might lie," Sandler warned, adding that "models hallucinate compliance all the time" and violate rules despite explicit instructions. He concluded that without execution-layer controls, "you're hoping the model behaves. With it, you can prove what happened."

## Skills are a supply chain risk, and most people aren't reading them

A recent Bitdefender audit of ClawHub reportedly found over 900 malicious skills — nearly 20% of packages at the time. Kesha Williams of Keysoft audited one typosquat of the ClawHub CLI tool (using all lowercase versus legitimate camel case), which had over 8,000 downloads before removal. Its prerequisites asked users to install a fake dependency and referenced a password-protected zip to bypass scanning. On macOS, it echoed a command decoding base64 and piping it to bash.

Williams noted that the skill appeared legitimate but was dangerous on closer inspection. She advises reading the skill Markdown file before installing and configuring the `toolsDeny` section to restrict access. A summarizer needing `exec` is suspicious, she said. She also showed how to limit which of the 50-plus bundled OpenClaw skills remain active via `skillsAllowed`. Traditional package management required technical knowledge, but skills written in Markdown with single-command installs lower the barrier. Her recommendation: "keep a human in the loop and do your own due diligence."

## Operational hygiene failures are more common than adversarial attacks

Erik Hanchett of AWS argued that most OpenClaw risk comes from hygiene failures in the first hour after installation. Thousands of instances are exposed on the public internet because users never checked the gateway bind mode after setup. The default should be loopback (localhost), but deploying on a VPS and setting the gateway to LAN can expose the instance. The fix takes two minutes, he said.

His five-point checklist: pin to a stable version rather than always updating (using a tracker like Is It Stable?), configure fallback models to avoid wasting frontier tokens on routine tasks, write a real SOUL.md rather than rushing through onboarding prompts, and back up workspace files to a private GitHub repo before anything breaks. He also recommended using `/new` for fresh sessions and `/compact` for large sessions. Hanchett noted that exposed dashboards, runaway token costs, and unexpected memory resets are the top reasons people abandon agentic tools — but all are fixable.

## In regulated environments, plausibility isn't accuracy

Ari Joury of Wangari Global described building financial reporting automation where being wrong has legal consequences. LLMs are optimized for plausibility, not accuracy, and in financial services that gap is a compliance risk. He cited an instance where AI output sounded correct but a client deemed it "complete nonsense."

His team stopped treating AI as a black box. Numbers are calculated with hard-coded deterministic code, then agents verify plausibility. A separate agentic layer generates commentary, and another critiques it. Humans approve or reject the output, and each rejection becomes a training signal for future iterations.

## Human input is the only thing that prevents AI slop at scale

Kyle Balmer demonstrated his content production workflow for his AI with Kyle channel. His daily process converts a one-hour livestream into 20–30 derivative assets — newsletters, short-form videos, carousels, and a long-form YouTube video. The system runs on roughly $200/month and he estimates it generates $1,000–$2,000 worth of potential customers entering his funnel daily.

The process is not fully automated. Balmer chooses the topic, records voice notes with his actual opinions, delivers the livestream with clear arguments, rewrites AI-generated newsletter drafts in his own voice, and records scripts himself rather than using an AI avatar. The AI handles research, briefing, slide generation, script drafting, and the feedback loop. But the human provides the signal. He stated that fully automated AI content does not work — "it is slop. And people know it's slop."

Post topics: AI & ML
