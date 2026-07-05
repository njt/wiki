---
title: "Woodshed"
url: https://tangled.org/danabra.mov/woodshed
date_fetched: 2026-05-14
section: "AI-enhanced Coding"
---

# Woodshed: Evals for Claude Skills

Open-source development framework for creating, testing, and iterating on Claude Skills through automated evaluation and refinement cycles.

## Key Features
- **Skill Creation & Variants**: Organize multiple skill implementations (baseline, experiment, silly versions) in a structured workspace
- **Automated Testing**: Run skills against fixtures with custom evaluation criteria
- **Batch Processing**: Execute entire skill matrices up to 10 times by default
- **Idempotent Runs**: Re-running skips past results; use `--reset` to force fresh iterations
- **Evaluation Loop**: Dedicated agent evaluates outputs against `eval.md` criteria
- **Results Tracking**: Comprehensive logging in `/results/` folder

## Notable Quotes
"Create, run, rate, and iterate on your Claude Skills."
"It runs Claude in yolo mode which can and will wipe your data."

TypeScript (73.6%), MIT license. Hosted on Tangled (a decentralized platform). Alpha software, 16 stars.
