---
url: https://factory.mastra.ai/
title: "Mastra Factory — Get Started"
author: Mastra
date_fetched: 2026-11-20
date_published: n.d.
topics:
  - coding-agents-and-frameworks
  - agent-orchestration
---

Mastra Factory is an open-source "software factory": connect a repository and it turns GitHub issues into plans, implementations, and reviewed pull requests, using stages, skills, and approval gates the team configures. It runs on Mastra platform by default (auth, PostgreSQL, cloud sandboxes) or fully self-hosted, with each of authentication, storage, and sandboxes independently configurable.

The getting-started guide walks through scaffolding the Factory server (`npm create factory@latest`, in a directory deliberately separate from the target repo), configuring a credential-encryption key, starting the server, and connecting a codebase, model provider, and issue intake (GitHub issues, optionally Linear). A canonical walkthrough issue — a README contributing section — is then taken through the full pipeline: Intake → Investigate → Accept → Plan approval → Implementation → agent PR → a **separate review session** on the Review board → human merge through the repo's normal process.

The staged walkthrough is the interesting part: acceptance, plan approval, and PR merge are three distinct human gates, review happens in a separate session from authoring, and the docs insist that "an agent saying a check passed is useful context" — you also inspect tool output and the diff. Factory does not reassess a task after merge; acceptance criteria are the human's to verify.
