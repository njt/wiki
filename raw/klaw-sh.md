---
title: "klaw.sh"
url: https://github.com/klawsh/klaw.sh
date_fetched: 2026-05-14
section: "LLMs"
---

# klaw.sh: kubectl for AI Agents

Enterprise orchestration platform for AI agents, drawing from Kubernetes operations patterns. Single Go binary.

## Core Features

- kubectl-style agent lifecycle management (klaw get agents, klaw describe agent, klaw logs)
- Namespace-based logical isolation with scoped secrets and tool permissions
- Built-in cron scheduling for recurring agent tasks
- Slack integration (@klaw status, @klaw run)
- Multi-model support: ~300 LLM models via Anthropic, OpenAI, and OpenAI-compatible endpoints

## Architecture

- Namespaces for logical isolation
- Scheduler for cron-based execution
- Channels (Slack, CLI) for user interaction
- Router across LLM providers
- Nodes as worker processes in distributed deployments

## Deployment Models

- Single Node: development and small teams
- Distributed: central controller with multiple worker nodes, auto-dispatch

## Use Cases

Sales lead scoring with CRM integration, competitive intelligence monitoring, support ticket automation, automated analytics reporting.

## License

Source-available: free for internal business use and personal projects; licensing required for multi-tenant SaaS distribution.
