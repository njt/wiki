---
title: "Experience Design for Agents"
url: https://kurtiskemple.com/blog/agentic-experience-design/
date_fetched: 2026-05-14
section: "LLMs"
---

# Agentic Experience Design

By Kurtis Kemple. Framework for designing the interaction layer of AI agents, arguing experience design is the single biggest factor in adoption or abandonment.

## Core Thesis

"The single biggest factor in whether an agent gets adopted or abandoned is experience design." Agents represent a fundamentally new interaction paradigm -- partial autonomy -- requiring a distinct design discipline.

## The Responsibility Model (Four Layers)

1. Human Layer: Owns intent and judgment. Defines what needs to happen, evaluates outcomes. Judgment cannot be delegated.
2. Agent Layer: Owns planning and outcomes. Determines what to do given a goal, accountable for results.
3. Workflow Layer: Owns automation. Executes structured, repeatable sequences with predictability.
4. Tool Layer: Owns execution. Performs single functions on demand without memory or judgment.

## The Autonomy Model

- Humans: Full autonomy to intervene at every level
- Agents: Bounded autonomy within defined scopes
- Workflows: Deterministic autonomy (no deviation)
- Tools: Zero autonomy (respond only to invocation)

## Three Enabling Conditions

### Planning Requires Context
Context is "the informational environment that makes coherent planning possible over time." Includes goals, constraints, decisions, current state, history, and open questions.

### Authority Enables Outcomes
Authority must remain aligned with human judgment through: transparency, steerability, resilience under failure, reversibility, guardrails.

### Progressive Authority
Authority expands as agents demonstrate competence -- both for individual users earning trust and for admins rolling out capabilities gradually.

## Design Posture

"Ship with narrow, inspectable, reversible defaults. Make the authority structure visible."

## Critical Failure Mode

Drift: when context management lapses and the agent's understanding of user intent becomes stale and incoherent over longer interactions.
