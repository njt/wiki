---
title: "OpenSandbox"
url: https://github.com/alibaba/OpenSandbox
date_fetched: 2026-05-14
section: "Security"
topics:
  - security-and-sandboxing
  - databases-and-data
---

# OpenSandbox: AI Application Sandbox Platform (Alibaba)

## Purpose
General-purpose sandbox platform designed for AI applications, providing multi-language SDKs, unified APIs, and Docker/Kubernetes runtimes for coding agents, GUI agents, agent evaluation, code execution, and RL training.

## Core Features

**Multi-Language Support:**
Python, Java/Kotlin, JavaScript/TypeScript, C#/.NET, and Go SDKs. Sandbox Protocol defining lifecycle and execution APIs.

**Sandbox Capabilities:**
- Command execution, filesystem operations, code interpreter
- Network policy management with ingress gateway and egress controls
- Strong isolation via gVisor, Kata Containers, and Firecracker microVM
- Browser automation (Chrome, Playwright) and desktop environments (VNC, VS Code)

**Developer Tools:**
- CLI (`osb`) for sandbox creation, command execution, file operations, diagnostics
- MCP server for integration with Claude Code and Cursor
- Code Interpreter SDK for executing code across multiple languages

## Notable
- 10.6k GitHub stars, 849 forks
- Listed in CNCF Landscape
- Apache 2.0 licensed
- 44.7% Python, 30.7% Go
