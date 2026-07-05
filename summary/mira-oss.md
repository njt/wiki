---
title: "mira-OSS"
url: https://github.com/taylorsatula/mira-OSS
date_fetched: 2026-05-14
section: "Personal Agents"
---

# MIRA OS: Comprehensive Project Summary

## Primary Purpose

MIRA OS is an open-source artificial intelligence system designed as a self-directed digital entity with persistent memory and learning capabilities. The creator describes it as "a comprehensive best-effort approximation of a continuous digital entity" that maintains conversation continuity without requiring users to start new chat sessions.

## Core Architecture

**Event-Driven Design**: Synchronous event-driven architecture where modules operate independently through subscription mechanisms. When 120 minutes elapse without new messages, a SegmentCollapseEvent triggers memory extraction, cache invalidation, and summary generation.

**Memory System**: Automatic memory decay based on usage patterns rather than manual curation. Memories earn relevance through being referenced in conversations or linked to other memories. Discrete synthesized information loads into context windows via semantic similarity, traversal, and filtering—no manual memory searches required.

## Distinctive Features

**First-Person Narrative Memory**: Rather than third-person summaries ("The assistant discussed..."), MIRA generates summaries as lived experiences ("I debugged the IndexError..."). This prevents the model from treating memories as external logs rather than personal experiences.

**Text-Based LoRA**: After each conversation segment, a feedback extractor identifies prediction errors, negative feedback, and positive feedback. Every seven active-use days, a pattern synthesizer analyzes accumulated signals and evolves behavioral directives stored in a BEHAVIORAL DIRECTIVES section that influences all subsequent interactions.

**Dynamic Tool Management**: Tools self-register on startup with zero configuration. Mira controls tool availability through an invokeother_tool function—unused tools expire from the context window after five turns, preventing token waste.

## Included Tools

Contacts, maps, email, weather, pager, reminder, web search, history search, domaindoc, speculative research, and invocation capabilities. New tools can be created through Claude Code in approximately 5 minutes.

## Document Management

Handles long-form content through domaindoc_tool, which allows autonomous expansion, collapse, and subsectioning. When content isn't needed, MIRA "closes the drawer," removing tokens from the context window while maintaining section titles for later reaccess.

## Installation Options

- **Local Deployment**: Single curl command handles platform detection, Python environment, model downloads, PostgreSQL/Valkey/Vault deployment, service verification.
- **Docker Deployment**: Single-container with s6-overlay supervision.
- **Hosted Access**: Fully featured version at miraos.org with free-tier access.

## Technical Stack

- Python (87.6%), PostgreSQL with pgvector, Valkey, HashiCorp Vault
- Primary support for Claude (especially Opus 4.5), with fallback for alternative providers
- Shell scripting, HTML frontend, PLpgSQL stored procedures

## Licensing & Philosophy

AGPL-3.0 license. Creator emphasizes commitment to matching open-source and hosted implementations. "Has the potential to someday be something more than the sum of its parts."
