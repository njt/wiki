---
title: "surf-cli"
url: https://github.com/wesen/surf-cli/
date_fetched: 2026-05-14
section: "Random"
topics:
  - developer-tools
---

# Surf CLI: Browser Automation for AI Agents

Surf is a command-line interface enabling AI agents to control Chrome browsers without configuration or API keys. The tool operates through Unix socket communication, making it compatible with various AI systems including Claude Code, GPT, Gemini, and custom agents.

Agent-Agnostic Architecture: Rather than binding to specific AI providers, Surf uses pure CLI commands over Unix sockets, allowing integration with any system capable of executing shell commands.

Zero Configuration: Installation requires only loading the extension in Chrome and running the native host installer -- no MCP servers, relay processes, or subscriptions needed.

Battle-Tested Design: Developers reverse-engineered production browser extensions and tested against "agent-hostile pages like Discord settings," implementing graceful fallbacks when Chrome DevTools Protocol becomes unavailable.

Token Efficiency: Screenshots automatically resize to 1200px to reduce token consumption. Actions auto-capture screenshots, eliminating redundant round-trips to language models.

Browser-Based AI Access: Users can query ChatGPT, Gemini, Perplexity, and Grok through existing browser logins without API keys.

Capabilities include page navigation, content reading via accessibility trees, semantic element location, clicking, typing, scrolling, JavaScript execution, network capture with filtering, multi-step workflow automation, and multi-browser support (Chrome, Chromium, Brave, Edge, Arc, Helium).

Technical architecture: CLI -> Unix Socket -> Native Host -> Chrome Extension -> Chrome DevTools Protocol/Scripting API. When CDP is unavailable, falls back to chrome.scripting API and captureVisibleTab for screenshots.

681 stars, MIT licensed.
