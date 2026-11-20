---
url: https://github.com/mastra-ai/template-factory
title: "Mastra Factory (template-factory)"
author: Mastra AI
date_fetched: 2026-11-20
date_published: unknown
topics:
  - coding-agents-and-frameworks
  - agent-orchestration
---

Mastra Factory (repo: `template-factory`) is Mastra's open-source scaffold for a **software factory server**: connect a repository, and issues become plans, implementations, and reviewed pull requests. The repo itself is deliberately tiny — a README, an `AGENTS.md`, and a single ~100-line entry point (`src/mastra/index.ts`) that assembles a `MastraFactory` from the `@mastra/factory` npm package. The interesting design is visible in that entry point: one Postgres database (with pgvector) powers agent storage, factory app tables, a distributed project lock, and better-auth, with a paired `PgVector` for recall search on the same connection.

Deployment is the template's actual product. `npm create factory` provisions a Mastra platform project plus Postgres, writes credentials (including a `FACTORY_CREDENTIAL_ENCRYPTION_KEY` for encrypting stored model keys and integration secrets), and the same entry runs under `mastra dev` locally (local sandboxes) or `mastra deploy` / self-hosted Node (platform sandboxes). Sandboxes, storage, and auth are independently swappable — `LocalFactorySandbox` vs platform sandboxes, `PgFactoryStorage` vs other adapters.

Configurable integrations route work in: GitHub issue sync, Linear, Jira, and Slack (webhook or socket mode, with fail-fast validation that refuses partial Slack configs at startup rather than silently disabling it). The README documents an issue-to-PR walkthrough — investigate an issue in Work → Intake, watch the agent's session, then take the change through plan review and PR review.

The entry file's comments reveal the internal model: **boards own lifecycle, transition policy, and tool-result rules** (built-in Work and Review boards), a `configVersion` stamps audit rows so decisions trace back to the deployment that produced them, and a controller with workers boots at module load via top-level await so the deployer imports a fully-booted instance. One server serves both the Factory UI and the API.
