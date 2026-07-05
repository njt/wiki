---
url: https://github.com/Joi/ai-agent-learnings/blob/main/architecture.md
title: "Switchboard/jibrain: A Production Knowledge Architecture for AI Agents"
author: Joi
date_fetched: 2026-05-14
date_published: 2026-03-28
---

# Switchboard/jibrain: A Production Knowledge Architecture for AI Agents

**Author:** The repository is under the GitHub user Joi. This document describes a personal knowledge system built and actively used by this individual.

## Summary

A comprehensive architecture document describing Switchboard/jibrain — a production knowledge system for AI agents built on an Obsidian vault. The system uses a three-tier knowledge pipeline (intake → atlas → domains) with strict containment boundaries, a formal routing decision tree, frontmatter as the contract between agents and the system, multi-agent workspace isolation, and a suite of health monitoring including a "Seven Gates" pipeline completeness audit. Deployed across multiple machines with Syncthing peer-to-peer sync. The document is opinionated, battle-tested, and describes a system in active daily use.

## Core Architecture: Three-Tier Knowledge Pipeline

The system organizes content across three tiers:

**intake/** — temporal, disposable, flat directory. Everything enters here as drafts.
**atlas/** — durable, authoritative, typed subdirectories. Promoted permanent knowledge.
**domains/** — deep, domain-specific schema. Graduated areas with their own structure for mature topics.

### Why Three Tiers

- **Search pollution**: Meeting notes get mixed with concepts — temporal content never reaches atlas/
- **Stale knowledge**: No draft vs. verified distinction — handled by `status: draft` vs `status: final`
- **Queue blindness**: New content becomes indistinguishable from old — intake/ is explicitly a queue
- **Depth ceiling**: All topics treated equally — domains/ allows deep structure for mature topics

The "key insight" is that intake/ is disposable, with value living in atlas/ and domains/. Agents can write freely to intake/ without polluting the permanent knowledge base.

## Routing Decision Tree

1. Is this durable domain knowledge (concept, person, organization, reference)?
   - YES → Does it already exist in atlas/ or domains/?
     - YES → Update existing file ("reweave, not duplicate")
     - NO → intake/ for triage and promotion
   - UNCERTAIN → Is it an observation about the system itself?
     - YES → intake/.observations/
   - NO → Is it operational or temporal?
     - YES → _review/ or don't persist

## Frontmatter as Contract

Every file requires YAML frontmatter. Called "the contract between agents and the system."

Intake schema: type, description (~150 chars), source, source_url, source_date, tags, status: draft, agent
Atlas schema: type, description (~150 chars), tags, status: final, promoted_from, promoted_date, last_verified

The description field is "the single most important field" — enabling **filter-before-read** so agents scan descriptions before opening files. "A vault of 2,400 files is too large to read exhaustively."

## Containment

Boundaries are "structural requirements, not suggestions": temporal vs durable, private vs public, queue vs archive, agent workspace. Failure modes include temporal in durable (meeting prep polluting concept search), operational in knowledge, private in public, queue stagnation (>90 days).

## The Reweave Pass

After promoting to atlas/: now that this entity exists, what existing notes should link to it? Algorithm identifies recently promoted files, extracts key entities, searches for related unlinked files, generates prioritized action report.

Core principle: "Connecting 5 existing concepts is worth more than 5 new unconnected files."

## Agent Observations

Dedicated directory (`intake/.observations/`) for agents to log gaps, contradictions, connections, friction, and structural issues noticed during work. "Most knowledge systems only capture what you deliberately put in" — observations capture what the system itself reveals.

## Seven Gates Audit

Seven structural checks: Intent (type+description), Precision (frontmatter schema), Materia (source provenance), Containment, Currency (last_verified), Cost (intake-to-atlas ratio), Verification (orphans, broken links).

## Multi-Agent Architecture

Specialized agents with explicit workspace boundaries: Curator, Meeting Extractor, Meeting Prep, Triage, Context Advisor. "No single agent can corrupt the entire system."

Agent identity persistence designed but not yet built — agents start fresh each session.

## Async Extraction Pipeline (Knowledge-Intake Sprite)

Persistent FastAPI extraction service on Firecracker microVM via sprites.dev. `POST /intake` for URL extraction, `POST /intake/structured` for pre-extracted JSON.

## Heartbeat System

Deployed March 13, 2026. 15-minute collection and triage cycle. Conservative auto-promote (6 gates, no AI) + scheduled AI triage when 5+ pending or at hours 7, 12, 17, 22.

## Multi-Machine Sync via Syncthing

Peer-to-peer, encrypted, no cloud dependency. Three machines: primary-mac (full vault), agent-mac (jibrain/ only), knowledge-intake sprite (jibrain/ text-only subset, send-only).

## Ethoswarm Integration

Always-on intake via Telegram Curator Mind. Phase 1 operational. Phase 2 (bidirectional sync) in progress.

## Intelligent LLM Routing (iblai-router)

Deployed March 4, 2026. Rule-based router selecting Haiku/Sonnet/Opus based on message complexity. 14-dimension weighted scorer, <1ms. 80% cost savings vs Opus-for-everything across 368 requests, 0 routing errors.

## Domain Graduation

When an atlas topic reaches ~15+ files with distinct vocabulary, it graduates to domains/ with its own internal structure. Exemplar: chanoyu/ (Japanese tea ceremony), which has its own git repo.

## GTD Integration

Apple Reminders ↔ Obsidian vault, Beads (dev issues) ↔ jibrain, email triggers knowledge capture, morning routine includes jibrain health check, weekly review includes triage + promotion.

## Health Monitoring

Orphan detection, dead-end detection, broken link detection, Seven Gates audit, stale contact detection, intake queue size, observation count. Every morning includes a jibrain status pulse.

## Search Infrastructure (QMD)

Three modes over ~4,300 files: keyword (exact matching), vector (embedding similarity), deep (query expansion + hybrid + reranking). Collections: jibrain (~2,400), dailynote (~1,000), people (~900, private).

## Team-Facing Knowledge Interface (Onyx)

Onyx (open-source RAG, YC W24) as team-facing read-only projection. Vault → Projection Pipeline (sensitivity filter) → Onyx File Upload Connector → Team search + chat. Docker Compose on GCP (~$131/month). Custom agents with system prompts stored in code repo.

Multi-interface envelope pattern: Onyx, Chat bot, Slack, Email all normalize into NormalizedEnvelope → Policy Engine → Actions.

## What Worked

1. Zero-fork strategy: vanilla upstream Onyx
2. Vault projection with sensitivity filter
3. Pydantic + pure-function policy engine
4. Multi-channel envelope pattern
5. File-backed audit: YAML files in git repo beat a database for compliance

## What Would Be Done Differently

1. Automate corpus refresh from day one
2. Script Onyx admin setup via API
3. Start with Slack connector for faster team adoption

## What We Don't Have Yet

- Agent identity persistence (designed, not built)
- Adversarial verification
- Confidence decay
- Cross-vault federation
- jibot cron (approved, not implemented)
- Source attribution/compensation

## Credits and Influences

Obsidian, GTD (David Allen), Ars Contexta, Kieckhefer (1994), Mechanics of Magic (nraford7), Ethoswarm (amind.ai), Colin Raney / Fred, sprites.dev, Syncthing, Amplifier (Microsoft), QMD
