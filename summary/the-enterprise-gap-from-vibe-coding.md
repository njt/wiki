---
url: https://wolkensteiner.substack.com/p/the-enterprise-gap-from-vibe-coding
title: "The Enterprise Gap: From 'Vibe Coding' to Executable Architecture"
author: Wolkensteiner
site: wolkensteiner.substack.com
date_fetched: 2026-08-06
topics:
  - guardrails-and-feedback-loops
  - agent-coding-workflow
---

A non-technical colleague built a working app in a single day using an AI app builder. To leadership, it looked like magic. But under the hood, the system was held together by duct tape: half the data lived in Supabase, half was hardcoded mock JSON in frontend components; zero backend logic with every database call made straight from the client; zero authentication — no SSO, no roles, no audit logging.

The author, brought in as architect to make it production-ready, describes the process as "retrofitting reality onto a prototype that was never built to hold it." The result works, but it isn't how anyone would design the system from actual requirements. It's a prototype wearing compliance bandages.

The core argument: tools like Lovable, Cursor, and Claude Code optimize for *speed to pixels* — instant visual feedback by taking every shortcut available. But nobody evaluates production software on how fast it renders a form. They evaluate repeatability, maintainability, security, and whether it still holds together a year later. "Prompt and hope doesn't scale past a demo."

The solution proposed is decoupling planning from execution: a plain TOML plan template (writable by LLM or human), checked against policy before anything executes, then run by a deterministic engine with strict guardrails — no model improvising through runtime errors.

The author open-sourced **rigorix-oss**, a deterministic execution engine for "bounded autonomy" (MIT/Apache-2.0 dual license). It ships as a CLI, GitHub Action, and MCP integration. Quality issues get a validate-fix loop; policy violations halt execution cold with a clear report of what crossed the line.
