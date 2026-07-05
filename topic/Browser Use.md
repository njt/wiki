# Browser Use

An AI agent platform for automating web tasks through natural language. Point it at a URL, tell it what to do, and it drives a hardened Chromium fork with stealth anti-detection (fingerprint randomization, residential proxies, Cloudflare bypass). The clever bit is deterministic rerun: first execution creates a cached script (~$0.10), subsequent runs with different parameters cost $0 in LLM fees.

---

## Key Quotes

> No standout quotes -- it's API documentation.

## Key Themes

#browser-automation #web-scraping #anti-detection #caching #cloud-agents

The deterministic rerun pattern is genuinely novel. Most browser automation tools treat every run as a fresh LLM call. Caching the *script* (not just the result) means you pay once for the AI to figure out the workflow, then replay it cheaply with different inputs. The Grow Therapy example (12 geography-specialty combinations for ~$0.10 total after initial caching) demonstrates real cost efficiency.

The stealth infrastructure (anti-fingerprinting, residential proxies, bot-detection bypass) is where the security implications get interesting. As the annotation notes: "Do not think about the security of this page. That will not make you happy."

## Critical Analysis

This is powerful and somewhat alarming. The combination of AI-driven browser automation with anti-detection stealth means you can programmatically interact with any website as if you were a human, at scale. The legitimate uses (testing, data extraction, form filling) are real. The illegitimate uses (credential stuffing, scraping behind paywalls, impersonation) are equally real.

The human-in-the-loop feature (agents pause for approval) is a nice safety valve but doesn't address the fundamental dual-use problem. The `llms-full.txt` URL as a single-page documentation dump for AI consumption is pragmatic -- point Claude at it and it learns the whole API. That's either efficient knowledge transfer or a social engineering vector depending on your threat model. Compare with [[Gemma Gem]] for an on-device alternative that doesn't require cloud infrastructure.

---
*Sources: [[summary/browser-use]]*
*Last updated: 2026-05-14*
