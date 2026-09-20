---
url: https://www.freecodecamp.org/news/how-to-fix-a-leaked-api-key/
title: "How to Fix a Leaked API Key"
date_fetched: 2026-09-20
topics:
  - security-and-sandboxing
  - software-engineering-craft
---

A practical incident-response guide for the moment a developer discovers an API key sitting in a Git repository. The core claim is that deletion is not remediation: once a credential has been committed, it must be treated as compromised regardless of how quickly it is removed, because Git history preserves it and automated scanners harvest exposed keys within minutes. The article prescribes a fixed order of operations — Invalidate → Investigate → Remove → Replace → Prevent — with revocation always preceding history cleanup, since rewriting a repository cannot recall copies that already exist in forks, CI logs, or scanner databases.

The middle sections walk through the mechanics: checking provider logs and billing for abuse, moving secrets from source code into environment variables, maintaining a `.env.example` alongside a gitignored `.env`, and rewriting history with `git filter-repo` (with a mirror-clone backup and a warning about force-push disruption to collaborators). A recurring theme is that cleanup is broader than the repository: build artifacts, Docker images, PR comments, and CI/CD variables all retain secrets, and the replacement key must be rotated in every environment, not just production.

Prevention gets equal weight: least-privilege keys, per-environment credentials, secret scanning with tools like Gitleaks, pre-commit hooks, and reviewing staged diffs before committing. The article closes with a 23-step incident checklist and a non-shaming framing — leaks signal missing guardrails, not personal failure.
