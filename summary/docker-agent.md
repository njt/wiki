---
url: https://github.com/docker/docker-agent
title: "Docker Agent"
author: Docker
date_fetched: 2026-10-09
date_published: 2026 (ongoing repo)
topics:
  - coding-agents-and-frameworks
  - agent-orchestration
---

Docker Agent (`docker-agent`, run as the `docker agent` CLI plugin) is Docker's open-source, Go-based framework for building, running, and sharing AI agents declared entirely in YAML. A config file names agents with models, instructions, toolsets, and relationships; the binary runs them as single agents or collaborating teams, with no code required from the author.

Key features per the README: multi-agent architecture with automatic task delegation, a rich tool ecosystem of built-ins plus any MCP server (local, remote, or Docker-based), provider-agnostic model support (OpenAI, Anthropic, Gemini, AWS Bedrock, Mistral, xAI, Docker Model Runner for local models), built-in think/todo/memory tools, pluggable RAG (BM25, embeddings, hybrid search, reranking), and packaging/pushing agents to any OCI registry so they can be pulled and run anywhere.

Distribution follows Docker's instincts: the agent ships preinstalled in Docker Desktop 4.63+, via Homebrew or GitHub releases, and agents themselves are distributed artifacts (`docker agent run myorg/agent:tag`). Sandboxed execution is delegated to Docker Sandboxes. The ~630K-line Go codebase includes a TUI, an eval framework, A2A and ACP protocol servers, hooks, budgets, compaction, and a dogfooded workflow — the project builds itself with `docker agent run ./golang_developer.yaml`.

*Sources: README at https://github.com/docker/docker-agent*
