---
url: https://martinfowler.com/articles/archaeologist-copilot.html
title: The Archaeologist's Copilot
author: Nik Malykhin
date_fetched: 2026-07-18
date_published: 2026-07-16
site: martinfowler.com
---

# The Archaeologist's Copilot

**Author:** Nik Malykhin, published on Martin Fowler's blog (martinfowler.com)

**Publication Date:** 16 July 2026

## Overview

Nik Malykhin describes his method for modernizing a 20-year-old Java 1.5 "Big Ball of Mud" codebase using AI and Docker. The central insight: early use of LLMs gave "plausible answers that did not hold up in the codebase." Progress came when AI was used to support evidence-grounded analysis, validation in a stable Docker environment, and gradual refactoring protected by tests.

## The "Tourist" Trap

Malykhin introduces the concept of the "Tourist Prompt" — asking an LLM something like: "How do I run this?" When he tried this with the legacy codebase, the AI acted as "a polite, eager-to-please tour guide." It generated a modern `build.gradle` and a clean `HelloBlobStore.java` that looked miraculous on the surface.

But it was a lie. The AI hallucinated `commons-pool2 (v2.x)` when the legacy code used `org.apache.commons.pool (v1.x)`, with completely different APIs. It assumed a standard Maven layout (`src/main/java`) despite the non-standard Ant structure (`java/com/legacycorp...`). And its pristine example hid the fact that `SimpleBlobStoreImpl` wasn't thread-safe, error handling swallowed exceptions, and the "Unit Tests" were actually integration tests requiring a live MySQL database.

The lesson: "AI defaults to optimism." In restoration, optimism is fatal. He realized he needed to "stop acting like Tourists and start acting like Archaeologists."

## Phase I: The Analysis

Malykhin crafted an "Archaeologist Prompt" — assigning the AI a persona of Senior Legacy Systems Architect, forbidding README summaries, and ordering a Forensic Code Audit focused on four pillars: Carbon Dating (era), Architectural Integrity, Data Flow & Typing, and Safety (error handling & threading).

### Finding 1: Carbon Dating
The AI identified a `build.xml` with no `pom.xml` (pre-2010 Ant era), `org.apache.commons.pool.ObjectPool` (Version 1.x), and raw types like `Map` instead of `Map<String, String>`. Verdict: Java 1.5 code from 2005–2008.

### Finding 2: The Transliteration Trap
Worse, this was "Perl code masquerading as Java" — the original author took a procedural Perl script and forced it into Java. `SimpleBlobStoreImpl` was a massive god class handling socket connections, protocol parsing, and business logic. The code was "stringly-typed," passing raw `Map<String, String>` objects around. A typo like `get("fiel_id")` would cause a runtime crash.

### Finding 3: Lying Tests
The test suite relied on `LocalFileBlobStoreImpl`, a local-disk mock that bypassed the actual networking code, thread-unsafe pooling, and fragile protocol parser. The tests passed, but the volatile parts of the system were never exercised.

### Decision: Containment over Repair
Malykhin halted all changes. He refused to fix bugs, update dependencies, or even reformat whitespace. Instead, he moved to wrap the legacy code inside an isolated Docker environment.

## Phase II: The Wrap

He established prime directives: preserve the era (mimic 2008), contain rather than modernize, and make zero code changes. Yet his first instinct was still tainted — he tried swapping Ant for Gradle 8. The build crashed because legacy code accessed package-private classes across packages, which Ant tolerated but Gradle 8 and modern JDKs enforce strictly.

### The Time Capsule Strategy

He pivoted to recreate the exact 2008 environment via Docker. But Java 6 Docker images were only available for x86, while he was on ARM64. Emulation layers like Rosetta introduced unpredictability. "Sometimes software archaeology requires the right shovel" — he switched to a native Intel machine.

### The Wet Test

A stubborn integration test (`TestBlobStore.java`) had hardcoded assumptions — connecting to `qbert.legacycorp.com:7001` and expecting `~/Projects/blobstore/...`. Instead of modifying the test, he used Docker Compose to bend reality: spinning up a BlobStore container with a Docker network alias tricking the test into believing it was "the long-lost `qbert.legacycorp.com`," and mounting the local directory at the exact path from 2005.

The test suite ran flawlessly. "Without changing a single byte of historical source code," he had restored functionality to a twenty-year-old application.

## Phase III: The Lift

With a verifiable baseline established, he could now modernize. Any failures would be from modernization, not pre-existing rot.

### The Java 8 Choice

Java 8 was chosen because modern JDKs (17+) dropped support for compiling Java 1.5 source, while Java 6 won't run natively on ARM64. Java 8 is "the absolute last version to support the compilation of Java 1.5 targets" and one of the earliest that runs natively on modern Macs.

### The Java 17 Trap

Gradle 8 requires Java 17, which can't compile Java 1.5 source. He settled on Gradle 7.6 — the last modern-ish version that runs on Java 8, enabling the chain: Apple Silicon → Java 8 JVM → Gradle 7.6 → Java 1.5 Source.

He configured Gradle to map directly to the legacy directory structure (`srcDirs = ['java']`) and wired custom `JavaExec` tasks for legacy test entry points.

### The Lying Tests Discovery

Tests ran suspiciously fast. He found the culprit: the legacy code caught exceptions and swallowed them, printing "Failed: ..." but exiting with code 0. The build pipeline reported green even when the backend connection failed entirely.

### Hardening the Baseline

He made his first structural change — stripping out try-catch blocks to let exceptions crash the application naturally. The build turned red, which was "a massive narrative victory." After tracing and repairing connection configurations, the build returned to green — an honest green.

### The AI-Compiler Feedback Loop

He enabled `-Xlint:unchecked` and fed exact compiler warnings to the AI with targeted prompts to refactor specific lines. Raw collections like `List hosts` became `List<InetSocketAddress> hosts`. He maintained this cycle until the build succeeded with zero warnings.

## Phase IV: The Refactor

He moved source from `java/` to `src/main/java`, deleted custom Gradle workarounds, and migrated legacy `main()` test scripts to JUnit 5 — replacing `System.out.println("Error")` traps with `Assertions.assertEquals()` for automated test reporting.

### The TestContainers Trap

He attempted replacing docker-compose with TestContainers but it became a "Big Bang" refactor involving test runner, network topology, and startup logic all at once, complicated by Docker-in-Docker networking on ARM. "Momentum is oxygen" — he aborted and accepted the "External Sidecar" pattern, "deliberately choosing ground-level pragmatism over over-engineered perfection."

## The Final Sweep: Concurrency & Stress Testing

Two tasks remained: updating `LocalFileBlobStoreImpl` to implement the new generic-based `BlobStore` interface, and modernizing `StoreALot.java` — a multi-threaded load-testing tool. He had the AI act as senior performance engineer to refactor the script with generics and modern loggers, target `PooledBlobStoreImpl` against the Docker alias, and swap manual threads for `ExecutorService`.

He fired 100 iterations across 10 concurrent threads at the Docker-contained backend. The system held, proving that the thread-safety architecture relying on Apache Commons Pool was intact and that the modernizations hadn't destabilized core logic.

## Conclusion: The Handover

"Done" meant the code was runnable, testable, and predictable on modern hardware — transforming the repository from "archaeological mystery" into "standard technical debt."

### Scrubbing the Environment

He purged historical artifacts: `build.xml`, the `lib/` folder of unversioned JARs, `.classpath` and `.project` files. Running `rm build.xml" was "the final, cathartic act of modernization."

### The Project Roadmap

He generated a comprehensive `README.md` with prerequisites (Docker, Java 8+), a quick start (`./gradlew build`), and test instructions (`docker-compose up -d` then `./gradlew test`). The transformation from "Day 0 (The Archive)" to "Day N (The Product)" is summarized in a table: Ant → Gradle 8, Java 1.5 → Java 8, Manual Scripts → JUnit 5, Swallowed Exceptions → Hardened Tests.

### Final Thought

Malykhin emphasizes the lesson was "fundamentally about human agency." The "Tourist Prompt" failed because the AI lacked understanding of the environment and constraints. Success came from shifting mindsets — archaeologist, DevOps engineer, architect — directing the AI as a force multiplier. The AI handled the "tedious, repetitive translation layers" while he focused on high-level strategy. The codebase is now "runnable, testable, and predictable — fully equipped to endure the next ten years."

## Acknowledgments

Thanks to Matteo Vaccari for inspiration on AI-assisted modernization, and to Martin Fowler for feedback and guidance. The author notes he used Gemini to highlight key moments and outline sections, then reviewed and revised by hand, with a final AI pass for flow and grammar. "The experiments, conclusions, and final wording were all reviewed and edited by me and GitHub Copilot."
