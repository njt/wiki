---
title: "maestro"
url: https://github.com/SnapdragonPartners/maestro
date_fetched: 2026-05-14
section: "AI-enhanced Coding"
topics:
  - agent-orchestration
---

# Maestro App Factory

Multi-agent orchestration tool that uses AI to generate production-ready applications by organizing LLMs to operate like high-performing human development teams.

## Philosophy
"The goal is production-ready apps, not just code snippets." Organizes agents into distinct roles mirroring professional software teams.

## Core Agent Roles

- **PM (Product Manager)**: Conducts interactive requirements interviews, adapts questions based on user expertise, reads existing codebases for context, generates requirements specifications.
- **Architect**: Transforms requirements into technical specs, breaks specs into stories, enforces engineering principles (DRY, YAGNI), reviews code, merges PRs -- but does not write code directly.
- **Coders**: Pull stories from queues, develop plans, implement code, run tests, submit PRs for architect review. They terminate and restart between stories.

## Workflow
1. PM conducts interview and generates spec
2. Architect reviews and approves spec
3. Architect breaks spec into stories and dispatches
4. Coders plan, implement, and test
5. Architect reviews code and merges PRs
6. Coders terminate; new ones spawn for new work

## Operating Modes
- **Standard**: Main workflow with GitHub integration
- **Airplane Mode**: Fully offline using local Gitea and Ollama
- **Claude Code Mode**: Uses Claude Code as subprocess
- **Hotfix Mode**: Express path for urgent production fixes
- **Maintenance Mode**: Automated technical debt management

## Notable Features
- Knowledge graph capturing architectural patterns in `.maestro/knowledge.dot`
- Metrics dashboard tracking specs, stories, token usage, costs, test results
- State persistence via SQLite
- Mix-and-match LLM providers by agent type

Single binary (~15 MB), Go (95%), MIT license, Docker required.
