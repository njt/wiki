---
url: https://github.com/thunderbird/thunderbolt
date_fetched: 2026-07-05
backfilled: true
---

**AI You Control: Choose your models. Own your data. Eliminate vendor lock-in.**

Important

**We are excited about the amount of interest Thunderbolt has been getting and want to clarify that it is still early and under active development**. Currently, we are targeting enterprise customers that want to deploy it on-prem. We encourage you to self-host it and try it out, but there are a few caveats we are still working on:

- While we eventually plan to make Thunderbolt fully offline-first, it currently depends on authentication and search functionality (though you can disable search on the integrations screen in the app). You can deploy your own backend with Docker and sign up in order to test it locally.
- You’ll need to add your own model providers - we don’t yet have a public inference endpoint. We recommend using Thunderbolt with Ollama or llama.cpp if you want free local inference, or you can add API keys for any OpenAI-compatible model provider in the settings.

Thunderbolt is an open-source, cross-platform AI client that can be deployed on-prem anywhere.

- 🌐 Available on all major desktop and mobile platforms: web, iOS, Android, Mac, Linux, and Windows.
- 🧠 Compatible with frontier, local, and on-prem models.
- 🙋 Enterprise features, support, and FDEs available.

**Thunderbolt is under active development, currently undergoing a security audit, and preparing for enterprise production readiness.**

```
make doctor    # verify your tools — prints exact install commands for anything missing
make setup     # install frontend + backend dependencies, wire up agent symlinks
make up        # start Postgres + PowerSync in Docker
make run       # start the backend (:8000) and frontend (:1420)
```
For self-hosting with Docker Compose or Kubernetes, see `deploy/README.md`. For full dev-environment details, see `docs/development/quick-start.md`.

Found a bug? Have an idea?

- We're actively working on our docs, community, and roadmap. For now, the best way to get in touch is to File an issue.

We welcome contributions from everyone.

- **Development**: The development guide will help you get started.
- Make sure to check out the Mozilla Community Participation Guidelines.

- FAQ - Frequently asked questions
- Deployment - Self-host with Docker Compose or Kubernetes
- Development - Quick start, setup, and testing
- Architecture - System architecture and diagrams
- Storybook - Build, test, and document components
- Vite Bundle Analyzer - Analyze frontend bundle size
- Tauri Signing Keys - Generate and manage signing keys for releases
- Release Process - Instructions for creating and publishing new releases
- Telemetry - Information about data collection and privacy policy

Please read our Code of Conduct. All participants in the Thunderbolt community agree to follow these guidelines and Mozilla's Community Participation Guidelines.

If you discover a security vulnerability, please report it responsibly via our vulnerability reporting form. Please do **not** file public GitHub issues for security vulnerabilities.

Thunderbolt is licensed under the Mozilla Public License 2.0.
