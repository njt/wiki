---
url: https://docs.controlplane.com/mcp/overview
title: MCP Server
author: Control Plane (no individual author listed)
date_fetched: 2026-05-14
date_published: unknown
topics:
  - mcp-and-tool-protocols
---

# Control Plane MCP Server — Overview

"Connect AI assistants and tools to Control Plane using the Model Context Protocol (MCP)"

## Server Details

- **Endpoint:** `https://mcp.cpln.io/mcp`
- **Authentication:** Service account token in `Authorization` header
- **Tools:** 80+ tools for managing Control Plane resources
- **Resources:** 8 virtual resources (guide, about, CLI guide, canon, docs, OpenAPI, templates)
- **Prompts:** 8 curated prompts (General Assistant, GVC Operations, Workload Deployment, Secret Patterns, Troubleshooting, Image Management, Expert Assistant, Platform Engineer)

## Quick Start

1. Create a Service Account — generate an API key; add to "superusers" or "viewers" groups
2. Configure Your Client — tool-specific setup for Antigravity, Claude Code, Codex, Cursor, Gemini CLI, VS Code
3. Start Building — interact with Control Plane through natural language

## Service Account Permissions

Two built-in groups:
- **viewers** — `view` on all resources; read-only exploration
- **superusers** — `manage` on all resources; full automation

Custom permissions: create groups with specific members, define policies with targeted permissions.

Recommended setups:
- Quick exploration → viewers
- Full dev access → superusers
- Production automation → custom group with specific policies
- Team access → group per team with policies scoped to GVCs

## Initialization Flow

1. Read the Guide (`cpln+virtual://guide`) — tool taxonomy and best practices
2. Read Server Info (`cpln+virtual://about`) — server version, capabilities
3. Set Context (`set_context`) — establish default org and GVC
4. Begin Operations — use 80+ tools

## AI Plugin

"Auto-configures this MCP server and adds 23 skills, 8 agents, 8 commands, and 8 guardrail rules."

## Best Practices

1. Start with Context — always set org and GVC at conversation start
2. Be Specific — provide specific values for memory, CPU, replicas
3. Confirm Destructive Actions — configure assistant to confirm before deletes/updates
4. Use Tags — add tags for organization and filtering

Recommended prompt structure: Context → Action → Target → Details

## Compatible Clients

Any MCP remote-server-compatible client. Explicit guides for: Antigravity, Claude Code CLI, Codex, Cursor IDE, Gemini CLI, VS Code.
