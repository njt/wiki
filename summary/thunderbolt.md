---
url: https://github.com/thunderbird/thunderbolt
title: "Thunderbolt: Open-Source Cross-Platform AI Client"
author: Thunderbird (MZLA Technologies Corporation)
date_fetched: 2026-05-14
date_published: null
topics:
  - personal-agents
  - local-and-open-source-inference
---

# Thunderbolt

**Tagline:** "AI You Control: Choose your models. Own your data. Eliminate vendor lock-in."

## Overview

Thunderbolt is an open-source, cross-platform AI client from the Thunderbird team (MZLA Technologies Corporation, a subsidiary of Mozilla Foundation). It supports web, iOS, Android, Mac, Linux, and Windows. Compatible with frontier, local, and on-premises models. Enterprise-ready with support and FDEs. Self-hostable via Docker or Kubernetes. Licensed under Mozilla Public License 2.0 (MPL-2.0).

**Repository stats (as of May 2026):** 4.6k stars, 309 forks, 155 releases (v0.1.96).

## Status

The project is early-stage and under active development, currently undergoing a security audit. It depends on authentication and search functionality (search can be disabled). Users must add their own model providers. Recommends Ollama or llama.cpp for free local inference, or OpenAI-compatible API providers.

## Tech Stack

- **Languages:** TypeScript (96.4%), CSS, Shell, Astro, JavaScript, Rust
- **Backend:** Postgres database with PowerSync
- **Frontend:** Vite-based build system
- **Desktop apps:** Tauri framework
- **Testing:** Playwright, Vitest, Storybook

## Getting Started

```
make doctor    # verify development tools
make setup     # install dependencies
make up        # start PostgreSQL and PowerSync in Docker
make run       # start backend (port 8000) and frontend (port 1420)
```

## Documentation

- FAQ and deployment guides
- Architecture documentation with system diagrams
- Development quick-start and testing instructions
- Release process and telemetry information

## Community & Security

Follows Mozilla Community Participation Guidelines; includes Code of Conduct. Vulnerabilities should be reported via their vulnerability reporting form, not public issues.

## License

Mozilla Public License 2.0 (MPL-2.0).
