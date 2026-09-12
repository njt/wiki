---
url: https://github.com/instavm/security-skills
title: "Security Skills for CLI Agents"
author: instavm
date_fetched: 2026-07-18
date_published: 2025
topics:
  - security-and-sandboxing
---

A collection of 16 specialized security testing skills for AI coding agents (Claude Code, Gemini CLI, or any agent supporting MCP/Skills). Each skill is a self-contained markdown file — no code, pure prompt engineering — distilled from analysis of over 4,000 paid HackerOne bug bounty reports.

The skills form an operational playbook for web application penetration testing: grep patterns against mitmproxy traffic logs, curl-based reproduction commands, severity ratings (CRITICAL through INFO), and false positive guidance. Coverage spans IDOR, authentication flaws, business logic, SSRF, SQL injection, OTP/2FA bypass, PII exposure, leaked secrets, payment callback issues, checksum bypass, enumerable endpoints, insecure configurations, referer header leakage, API endpoint extraction, subdomain enumeration, a coordinating audit checklist, and a report generator.

Each skill is intentionally independent — no shared library, no cross-skill deduplication, no vulnerability database. Every skill takes the same input (`log.txt` from mitmproxy dump mode) and works in isolation. The trade-off is that comprehensive coverage requires running multiple skills sequentially, with the AI agent carrying findings forward and synthesizing results via the report skill's template.

The defining technical choice is text-based grep analysis of traffic logs rather than structured parsing of HAR or mitmproxy flow files. This keeps dependencies minimal and works with any text-producing capture tool, but sacrifices reliable access to request/response pairs and misses encoded or compressed content. The project's moat is its origin story: the vulnerability patterns, parameter vocabularies, and bypass encodings encode knowledge only learnable from large-scale bounty report analysis, not from documentation or academic sources.
