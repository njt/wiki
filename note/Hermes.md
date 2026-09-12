# Hermes

Nous Research's open-source personal agent framework with a self-improving loop: it creates and refines skills from experience, maintains persistent memory, and searches past conversations for continuity. Supports every messaging platform (Telegram, Discord, Slack, WhatsApp, Signal, Email), any LLM provider, and runs on anything from a $5 VPS to a GPU cluster. 149k GitHub stars and 8,200+ commits make it one of the most active agent projects.

---

## Key Quotes

(No standout quotes from the README -- it's feature-oriented documentation rather than argumentative prose.)

## Key Themes

#personal-agents #self-improving #multi-platform #open-source #hermes

The self-improving loop is the distinguishing feature: Hermes creates skills from experience rather than requiring manual skill authoring. This is the same aspiration as [[mira-OSS]]'s "text-based LoRA" approach, but implemented through explicit skill files rather than behavioral directive evolution.

The deployment flexibility (local, Docker, SSH, Singularity, Modal, Daytona, Vercel) and seven terminal backends reflect a design philosophy that the agent should run wherever you need it. This contrasts with [[clawdBot]], which focuses on being easy to set up on your own device, and [[Rowboat]], which is local-first by design.

The 40+ integrated tools, cron scheduler, and MCP integration make this a batteries-included framework. The Skills Hub at agentskills.io is an interesting community layer -- shared skills across Hermes instances.

## Critical Analysis

At 149k stars and 8,200 commits, Hermes has massive community momentum, but the breadth of features (seven backends, every messaging platform, serverless deployment) raises maintenance questions. Agent frameworks that try to be everything to everyone tend to be mediocre at any specific use case.

The self-improving loop is the most interesting technical claim, but the README doesn't explain the mechanism in detail. How does skill refinement work? What prevents skill drift? How do you audit what the agent has learned? These questions matter enormously for anyone deploying this in production.

The model flexibility (any LLM provider) is a double-edged sword: it means the framework can't deeply optimize for any single model's capabilities. Compare with [[mira-OSS]], which is explicitly built around Claude's strengths.

---
*Sources: [[summary/hermes]]*
*Last updated: 2026-05-14*
