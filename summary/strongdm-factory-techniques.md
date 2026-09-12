---
url: https://factory.strongdm.ai/techniques
title: "Techniques | StrongDM Software Factory"
author: StrongDM (Justin McCarthy)
date_fetched: 2026-05-15
date_published: unknown
topics:
  - agent-architecture
---

# Techniques | StrongDM Software Factory

"Practical Techniques" — patterns the team frequently uses while building with the Software Factory.

## Six Techniques

### 1. Digital Twin Universe (DTU)
Clones the observable behaviors of critical third-party dependencies. Enables validation at volumes beyond production limits with deterministic, replayable conditions.

### 2. Gene Transfusion
Transfers working patterns between codebases by directing agents to concrete examples. A good solution paired with a reference can be reproduced in fresh contexts.

### 3. The Filesystem
Models navigate repos efficiently and modify their own context via file read/write. "Directories, indexes, and on-disk state become a practical memory substrate."

### 4. Shift Work
Separates interactive tasks from fully specified ones. Once intent is complete — via specs, tests, or existing apps — agents run end-to-end without iterative back-and-forth.

### 5. Semport
"Semantically-aware automated ports, one time or ongoing." Moves code between languages or frameworks while preserving original intent.

### 6. Pyramid Summaries
Reversible summarization at multiple zoom levels. Compresses context while retaining the ability to expand back to full detail.

## The Validation Constraint

A system built with "zero hand-written code and zero traditional review" that must:
- Grow from cascades of natural-language specifications
- Be validated automatically without semantic inspection of source

Code is treated like an ML model snapshot — opaque weights whose correctness is inferred solely from external behavior. "Internal structure is treated as opaque."
