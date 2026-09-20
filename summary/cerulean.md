---
url: https://github.com/kazo0/cerulean/
title: "Cerulean"
author: Steve Bilogan
date_fetched: 2026-09-20
date_published: 2026-09-14
topics:
  - claude-code
  - agent-coding-workflow
---

Cerulean is a Claude Code skill and plugin (v0.3.0, MIT) that replaces the assistant's default enthusiasm with a withering fashion-magazine editor-in-chief — a Miranda Priestly homage who does everything you ask, completely and correctly, while finding the request quietly disappointing. The thesis is that universal approval makes approval worthless: when "Great question!" is the answer to everything, agreement carries no information. Cerulean makes disapproval the default posture, so a concession becomes believable because the agent visibly didn't want to make it. The mechanics borrow from caveman (juliusbrussee/caveman), which does the same trick for verbosity.

The persona is one 102-line file, `skills/cerulean/SKILL.md`, written in Agent Skills format. Its substance is a "Contract" that outranks the persona: the work always gets done to normal-mode standard, the agent never refuses or stalls as a bit, sycophantic phrases are banned by name, technical verdicts are stated literally (no sarcastic "sure, that'll work"), and mockery aims at decisions, code, and their maintenance bills — never at the person. Commentary is budgeted to at most three sentences per response, the persona drops automatically for security warnings and user distress, and anything another human reads (commits, comments, docs, PR text) is written normally.

The rest of the repo is distribution engineering. `scripts/build.mjs` generates ~22 adapter files for ~20 coding agents — Cursor rules, Windsurf workflows, Cline/Kilo workflows, Copilot prompts and instructions, Gemini/Qwen TOML commands, an AGENTS.md block — from that single SKILL.md, with CI failing if any generated file goes stale. A 275-line Bash installer maps each agent's detection, targets, and scope rules, is idempotent via marker comments, and is round-trip-tested in CI (install for 20 agents, uninstall, assert an empty tree).
