---
url: https://martinfowler.com/articles/archaeologist-copilot.html
title: "The Archaeologist's Copilot"
author: Nik Malykhin
date_fetched: 2026-07-18
date_published: 2026-07-16
topics:
  - agent-coding-workflow
  - software-engineering-craft
---

Nik Malykhin describes a method for modernizing a 20-year-old Java 1.5 "Big Ball of Mud" codebase using AI and Docker, published on Martin Fowler's blog. The core insight: naively asking an LLM "How do I run this?" produces plausible but wrong answers — the "Tourist Prompt" trap. AI defaults to optimism, hallucinating modern toolchains and APIs that don't match the legacy reality.

The alternative is the "Archaeologist Prompt": assigning the AI a Senior Legacy Systems Architect persona and ordering a forensic audit across four pillars — Carbon Dating (era), Architectural Integrity, Data Flow & Typing, and Safety (error handling & threading). This revealed the code was Java 1.5 from 2005–2008, was "Perl code masquerading as Java" with god classes and stringly-typed raw `Map` objects, and had a test suite that bypassed all networking, threading, and parsing code through disk-based mocks.

The modernization proceeds in four phases. **Containment** wraps the legacy code in Docker, recreating the exact 2008 environment and bending reality with Docker network aliases so hardcoded test assumptions still work — zero code changes, verifiable baseline. **Lift** moves from Ant/Java 1.5 to Gradle 7.6/Java 8, chosen because Java 8 is the last version that compiles Java 1.5 source and runs natively on Apple Silicon. Swallowed exceptions that masked test failures are stripped out, turning a false-green build honestly red, then genuinely green. **Refactor** migrates structure to standard layout, replaces manual test scripts with JUnit 5, and type-safe generics via an AI-compiler feedback loop (`-Xlint:unchecked` → targeted AI refactors). A TestContainers attempt is deliberately aborted when it becomes a "Big Bang" refactor; pragmatism wins. **Final sweep** stress-tests the containerized system with 100 iterations across 10 concurrent threads, confirming thread-safety holds.

The article frames this as fundamentally about human agency: shifting mindsets (archaeologist, DevOps engineer, architect) while using AI as a force multiplier for tedious translation work. The result transforms the repository from "archaeological mystery" into "standard technical debt" — runnable, testable, and predictable on modern hardware.
