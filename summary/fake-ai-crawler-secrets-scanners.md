---
url: https://tin.computer/blog/fake-ai-crawler-secrets-scanners
title: "Octopus, Gecko and Tortoise: the scanners hunting your AI keys"
author: Emre (tin.computer)
date_fetched: 2026-11-20
date_published: 2026-11-18
topics:
  - security-and-sandboxing
  - claude-code
---

A nine-week passive observation (22 September – 18 November 2026) of ten self-run websites documents three distinct secrets-hunting scanners targeting the credential files that AI coding agents leave behind.

**Octopus** is the big one: a likely-Go program that spoofs 32 AI crawler user agents (GPTBot, ClaudeBot, etc.), rotates one name per request, rents a fresh Google Cloud machine per target site across 15 regions, and fires ~300 secret-file requests in ten seconds. It first fingerprints the site like a normal Chrome browser — reading robots.txt, sitemaps, build manifests like `/.next/build-manifest.json` and `/.vite/manifest.json` — then tailors its 457-path list to the detected framework (different lists for static, Next.js, and Vite sites). Its target list tracks public CVEs: Langflow/Flowise code-execution paths appeared two weeks after advisories, and LiteLLM key-exposure paths arrived 12 November. It begins every run with a canary path `/z9x8c7v6b5-debug-trigger-<hostname>`.

**Gecko** is a .NET program (identified by its exact TLS cipher list) running mostly from Google Cloud Belgium plus cheap offshore Windows VPS. Its signature is a Chrome 128 user agent truncated at 60 characters — cutting off "like Gecko", hence the name. It sends ~200 requests in under a second for one fixed list, and that list is growing towards coding-agent credentials specifically: `/.claude/.credentials.json`, `/.codex/auth.json`, `/.cursor/mcp.json`, `/.gemini/settings.json`, MCP configs and agent logins. AI-tool paths grew from 4% of its requests in October to 10% by mid-November.

**Tortoise** is a small, slow Go scanner (one request every few seconds) probing Spring/PHP/Rails debug pages, seen only twice, from a budget VPS host.

Key conclusions: a crawler name is not proof of identity — 29 of Octopus's 32 fake names are in the popular ai.robots.txt allowlist, so user-agent blocking both fails to stop scanners and harms real crawlers; block by behaviour (dotfile paths, the truncated user agent, canary paths) instead. The exposure mechanism is new and agent-specific: build pipelines copying whole projects into output, misconfigured containers, and committed `settings.local.json` files put home-directory agent credentials on the public internet. The authors' own sites leaked nothing — every secret-path request got a 404 — and the article ends with concrete curl checks, .gitignore entries, and Cloudflare/nginx rules.
