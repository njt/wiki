---
title: "Elements of Agentic Systems Design"
url: https://github.com/idyllic-labs/elements-of-agentic-system-design
date_fetched: 2026-05-14
section: "LLMs"
---

# Elements of Agentic System Design

By William Chen at Idyllic Labs. Conceptual framework decomposing agentic systems into ten behavioral elements. Design space map for builders of agent frameworks, SDKs, and platforms.

## The 10 Core Elements

1. Context: Information available to the model per call
2. Memory: External storage for selective retrieval
3. Agency: Translation layer from text to effects
4. Reasoning: Grammar of call composition (chaining, looping, branching)
5. Coordination: Communication between reasoning structures
6. Artifacts: Shared persistent state for coordination
7. Autonomy: What triggers execution and owns the main loop
8. Evaluation: Determining system success
9. Feedback: Gradient signals steering behavior
10. Learning: Feedback that persists to change future behavior

## Key Patterns

### Externalization Pattern
- Memory externalizes context
- Artifacts externalize coordination
- Learning externalizes feedback

### Improvement Stack
- Evaluation measures quality
- Feedback steers current tasks
- Learning persists feedback for future tasks

## Framework Value

- Conceptual map defining terms and implementation mechanisms
- Analysis tool for decomposing any agentic system
- Debugging framework isolating problems to specific elements
- Behavior-to-code mapping connecting observed behaviors to concrete patterns

## Design Philosophy

"The model is a stateless text-to-text function. Everything else is architecture you build around it."

Intelligent-seeming behavior traces to concrete code patterns, not emergent model properties.

Licensed under CC BY 4.0. Book in development with open co-authorship.
