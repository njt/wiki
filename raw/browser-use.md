---
title: "Browser Use"
url: https://docs.browser-use.com/llms-full.txt
date_fetched: 2026-05-14
section: "AI-enhanced Coding"
---

# Browser Use Cloud Documentation

AI agent platform that automates web tasks through natural language instructions. Data extraction, form filling, multi-step workflows, research, monitoring, testing, scheduling across thousands of web apps.

## Models
- Claude Sonnet 4.6 (recommended default, $3.60/$18.00 per 1M tokens)
- Claude Opus 4.6 (complex tasks, $6.00/$30.00)
- GPT-5.4 mini (simple tasks, $0.90/$5.40)

## Key Features

**Deterministic Rerun:** Templates with `@{{parameter}}` markers. First execution ~$0.10 cached; reruns $0 LLM cost.

**Workspaces:** Persistent file storage for agent read/write.

**Sessions & Follow-ups:** Persistent browser state across tasks.

**Human-in-the-Loop:** Agents pause for human interaction via live preview URLs.

## Browser Infrastructure
- Hardened Chromium fork with stealth
- Anti-fingerprinting (Canvas, WebGL, fonts, navigator)
- Residential proxies, 195+ countries
- Cloudflare/bot-detection bypass
- Cookie banner auto-dismissal

## Auth Methods
Profiles (saved cookies), Agent Mail (verification codes), TOTP, 1Password integration, domain-scoped secrets.

## Integration
MCP Server (Claude, Cursor, Windsurf), n8n, OpenClaw, webhooks, marketplace.

Sessions timeout after 15 min idle (max 4 hours).
