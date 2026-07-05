---
url: https://github.com/n8n-io/n8n
title: "n8n — Secure Workflow Automation for Technical Teams"
author: n8n-io
date_fetched: 2026-05-14
date_published: unknown
---

# n8n

Secure workflow automation for technical teams. Fair-code licensed workflow automation platform with 400+ integrations, native AI capabilities, and self-hosting support.

188k stars, 57.6k forks. TypeScript 91.2%. Fair-code license (Sustainable Use License + n8n Enterprise License).

## Overview

n8n provides "the flexibility of code with the speed of no-code." It's a workflow automation platform that lets you build workflows visually or with code (JavaScript/Python), with 400+ built-in integrations.

The name derives from "nodemation" — combining "node" (Node-View and Node.js) with "mation" (automation), abbreviated as n8n for CLI usage.

## Key Features

- **Code When You Need It**: JavaScript/Python support with npm package integration alongside visual interface
- **AI-Native Platform**: Build AI agent workflows using LangChain with custom data and models
- **Full Control**: Self-host via fair-code license or use cloud deployment
- **Enterprise-Ready**: Advanced permissions, SSO, air-gapped deployment
- **Active Community**: 400+ integrations, 900+ ready-to-use templates

## Quick Start

NPX:
```
npx n8n
```

Docker:
```
docker volume create n8n_data
docker run -it --rm --name n8n -p 5678:5678 -v n8n_data:/home/node/.n8n docker.n8n.io/n8nio/n8n
```

Editor at http://localhost:5678

## Resources

- Documentation: docs.n8n.io
- 400+ Integrations: n8n.io/integrations
- Example Workflows: n8n.io/workflows
- AI & LangChain Guide: docs.n8n.io/advanced-ai/
- Community Forum: community.n8n.io

## License

Fair-code model: source-available, self-hostable, extensible. Distributed under Sustainable Use License and n8n Enterprise License. Not technically open source (not OSI-approved), but source is available and self-hosting is permitted.
