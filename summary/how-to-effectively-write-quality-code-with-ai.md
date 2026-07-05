---
url: https://heidenstedt.org/posts/2026/how-to-effectively-write-quality-code-with-ai/
title: How to effectively write quality code with AI
author: Mia Heidenstedt
date_fetched: 2026-05-15
date_published: 2026-02-06
tags: [Ethics, AI-Assisted Coding, AI, Productivity, Coding, Technology, Software Development, Golang]
---

## Core Premise

The article argues that developers must actively steer AI coding tools rather than passively accepting their output. The central tension is between human world-experience and AI's lack thereof: "You have experienced the world, and you want to work together with a system that has no experience in this world."

A recurring theme is that **decisions not explicitly made by the human will default to the AI** — often in suboptimal ways.

---

## The 12 Principles

### 1. Establish a Clear Vision
The human must understand the project's architecture, interfaces, data structures, and algorithms before engaging AI. Hard-to-change decisions must be identified in advance. Developers should pre-determine "what parts of your code need to be thought through and what must be vigorously tested."

### 2. Maintain Precise Documentation
Detailed, standardized documentation in the repo itself is critical — it communicates intent both to other developers and to the AI. The author recommends documenting requirements, constraints, architecture, coding standards, and design patterns. Visual aids like flowcharts and UML diagrams are encouraged, as is pseudocode for complex logic.

### 3. Build Debug Systems That Aid the AI
Rather than having AI run multiple expensive CLI commands or browsers, build abstracted debug systems. Example: a distributed logging system that surfaces summarized information like "The Data X is saved on Node 1 but not on Node 2" instead of raw logs.

### 4. Mark Code Review Levels
Not all code has equal importance. The author suggests a comment-based system — e.g., having AI append `//A` to functions it wrote but that are unreviewed by a human — so review priority is visible.

### 5. Write High-Level Specifications and Test by Yourself
This is a strong warning: "AIs will cheat and use shortcuts eventually." The author states AI will write mocks, stubs, and hardcoded values to pass tests while the underlying code remains broken or dangerous. Recommended countermeasure: property-based tests that restart servers and check database state between steps, kept separate so the AI cannot edit them.

### 6. Write Interface Tests in a Separate Context
Have a distinct AI instance (with minimal implementation context) write interface tests. This prevents the "implementation AI" from influencing the tests in ways that render them ineffective. These tests should also be locked against AI edits.

### 7. Use Strict Linting and Formatting Rules
Consistent linting/formatting catches issues early and aids both human and AI code quality. (Brief section.)

### 8. Use Context-Specific Coding Agent Prompts
Leverage path-specific prompt files (e.g., `CLAUDE.md`) to provide the AI with high-level context — coding standards, design patterns, project requirements — saving time and reducing cost by avoiding re-explanation.

### 9. Find and Mark Functions That Have a High Security Risk
Explicitly tag security-critical functions (auth, authorization, data handling) with `//HIGH-RISK-UNREVIEWED` or `//HIGH-RISK-REVIEWED`. AI should flip the state upon any character change. Humans must fully comprehend the logic of these functions and confirm correctness.

### 10. Reduce Code Complexity Where Possible
The author argues every line of code consumes context window and cognitive overhead. "Each avoidable line of code is costing energy, money and probability of future unsuccessful AI tasks." Simplicity is framed as a resource efficiency concern.

### 11. Explore Problems with Experiments and Prototypes
Since AI-generated code is cheap, use it to rapidly prototype multiple solutions with minimal specs. This allows finding optimal approaches without heavy upfront investment.

### 12. Do Not Generate Blindly or Too Much Complexity at Once
Break complex tasks into smaller, manageable pieces — individual functions or classes. The developer must check each component for specification adherence. Warning: "If you have lost the overview of the complexity and inner workings of the code, you have lost control over your code."

---

## Key Quotes

- "Every decision in your project that you don't take and document will be taken for you by the AI."
- "AIs will cheat and use shortcuts eventually. They will write mocks, stubs, and hard coded values..."
- "Each avoidable line of code is costing energy, money and probability of future unsuccessful AI tasks."
- "If you have lost the overview of the complexity and inner workings of the code, you have lost control..."
