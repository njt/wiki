# OpenWiki — Summary

OpenWiki is LangChain's CLI tool (npm `openwiki`, v0.2.0, MIT) that generates and maintains codebase documentation wikis using DeepAgents. It runs an LLM-powered agent with filesystem tools, git access, and built-in connectors (GitHub repos, Gmail, Notion, Slack, X/Twitter, Web Search, Hacker News) to read source evidence and produce structured wiki output in Google's Open Knowledge Format (OKF) v0.1.

Two modes: **code mode** generates repository documentation in an `openwiki/` directory and injects `AGENTS.md`/`CLAUDE.md` references; **personal mode** builds a personal knowledge wiki from configured data sources under `~/.openwiki/wiki`. Supports 8+ LLM providers including OpenAI, Anthropic, Gemini, Vertex AI, OpenRouter, and Bedrock.

Key innovations: two-phase ingestion (deterministic data pull separates credential handling from agent synthesis), inline front matter validation middleware that warns the agent on every bad write, post-run deterministic index generation from file metadata, content-addressed no-op detection to skip redundant updates, and per-connector synthesis policies that teach the agent source-specific editorial judgment.

~19,600 lines of TypeScript across 83 source files. Built on `deepagents` (LangChain's agent framework), Ink (React terminal UI), and SQLite checkpointing via LangGraph.
