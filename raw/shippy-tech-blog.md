---
url: https://huggingface.co/blog/allenai/shippy-tech-blog
title: "What building Shippy taught us about building agents"
author: Kyle Wiggers (Ai2Comms)
date_fetched: 2026-07-18
date_published: 2026-07-15
---

# What building Shippy taught us about building agents

**Published:** July 15, 2026
**Author:** Kyle Wiggers (Ai2Comms)
**Organization:** Ai2 (Allen Institute for AI)

## TL;DR

Shippy is a maritime AI agent for real-time ocean domain awareness, built by the Skylight team at Ai2. The article focuses on the system architecture behind reliability in high-stakes operational settings.

---

## Agent Anatomy — Skills, Soul, and Config

The authors decompose agents into three components:

- **Soul:** The system prompt that establishes Shippy's persona and behavioral guardrails.
- **Skills:** Markdown files describing how to handle specific request types (e.g., querying the Skylight API, looking up EEZ/MPA boundaries, interpreting vessel track data, generating map links).
- **Config:** Runtime settings—the agent harness (OpenClaw), the LLM (Claude Opus 4.6), API keys injected at runtime.

The soul and skills are baked into a versioned Docker image; config changes don't require a rebuild. Skills follow the "agent-skills spec" used by Claude Code and Codex, using "plain markdown files with structured frontmatter."

### Skills listed:

1. Querying the Skylight API for Events and vessel data
2. Looking up Exclusive Economic Zones (EEZ) and Marine Protected Area (MPA) boundaries
3. Interpreting vessel track data (building on Skylight's Atlantes models)
4. Generating interactive map links

### Multi-skill orchestration:

A single query like "Are there vessels operating near the Cordillera de Coiba MPA?" draws on three skills simultaneously—Skylight data query, ProtectedSeas MPA boundary context, and vessel track interpretation—"all in a single dialogue turn."

### Soul boundaries:

Shippy "won't make legal determinations about whether a vessel is breaking the law" and "won't speculate beyond what the data supports." These are explicit in the system prompt, not implicit in fine-tuning.

---

## Deterministic Tools for a Nondeterministic Agent

The core insight: agents are unpredictable, but their tools can be made reliable.

Shippy communicates with Skylight through a **purpose-built CLI** rather than raw API calls. The API has "dozens of input types, nested filter objects, pagination cursors, and complex geometry inputs." Early prototypes where Shippy constructed API calls directly produced "a steady stream of subtle bugs."

### CLI design choices:

- Single command: `skylight events search` with typed filter flags
- Handles authentication, pagination, and structured output automatically
- Extensive `--help` text and detailed error messages
- Output always written to **local JSON file** (not piped through shell) to avoid pipe buffer issues and enable programmatic access across steps

### Underlying API:

A standardized API with multiple resource types (Events, vessels, regions, satellite imagery, vessel tracks) accessible through two operations: search and aggregate.

### Layering philosophy:

"Each layer narrows what the next layer can get wrong." The API has its own test suite, the CLI can be exercised by human or agent, and skills reference CLI commands.

---

## Sandboxed Hosting and Isolation

Skylight serves "hundreds of government agencies and NGOs across over 70 countries." Data isolation is critical—a fisheries officer in the Philippines sees only their own data.

### Mothership:

An agent hosting platform built by the team that "provisions a dedicated Kubernetes deployment for each user session." When a conversation opens, pods spin up packaging the agent runtime, skills, and Skylight CLI. The user's JWT is injected at provision time.

### Sandbox properties:

- Files written during multi-step analysis are session-scoped and never shared
- The agent can write and run code, install dependencies, and pull in datasets
- Network access is restricted to only needed services

---

## Evaluating an Agent, Not a Model

The team rejected static benchmarks and built their own eval system that "scor[es] the whole agent – model, skills, and sandbox together – against live data."

### Eval process:

1. Subject-matter experts write scenarios, rubrics, and per-criteria weights
2. A natural-language prompt runs through the sandbox
3. An LLM judge grades each criterion 0–1 with written reasoning
4. Weighted aggregate is checked against a fixed pass threshold
5. Experts annotate individual responses as correct/incorrect as ground truth

### Harbor framework:

Tasks run through Harbor, an open evaluation framework. The team wrote a Harbor plugin that spins up a real Shippy session against live data. The suite runs in parallel against a specific versioned build, producing timestamped results and a delta report.

### Key finding patterns from latest run:

- Patrol-planning tasks: Shippy "overstepped into tactical recommendations rather than decision support"
- Geometry-sensitive queries: boundary simplification caused missed Events
- One case where the agent "invented a CLI command that didn't exist"

---

## Where We're Headed

Shippy is opening to early adopters on a rolling basis. Three planned improvements:

1. **Agent-driven UI control:** Shippy will drive the Skylight map directly (moving to regions, applying filters, adjusting time ranges)
2. **Model routing:** Simple lookups routed to smaller models; complex investigations use frontier models
3. **Cross-thread memory:** Persistent facts (jurisdiction, preferred sources) applied automatically across threads

### Broader impact:

The architecture is already influencing other Ai2 platforms—EarthRanger (wildlife conservation) and OlmoEarth (Earth observation tools). "Mothership was built to be general and to host other agents, so while maritime is the first domain…we don't expect it to be the last."

---

Built by the Skylight team at Ai2. Skylight is a free maritime domain awareness platform with "300+ partners across 70 countries."

## Reader Comment

A question about how the agent tracks which CLI output files correspond to which calls: does the agent specify the output path, or does the CLI generate a unique filename? Also asks about file management across long sessions.
