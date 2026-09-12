---
url: https://gogogolems.substack.com/p/slowing-down-in-the-age-of-coding
title: "Slowing Down in the Age of Coding Agents: Using AI for Deep Thinking, not Tokenmaxxing"
author: Manuel Odendahl
date_fetched: 2026-05-15
date_published: 2026-03-21
topics:
  - agent-coding-workflow
  - ideas-and-culture
---

The third in Odendahl's series on AI-assisted development (following pieces on simplicity and notation). Argues that as coding agents proliferate, the bottleneck has shifted from writing code to thinking about what code should be written. His response: deliberately slow down with analog tools, annotation cycles, and vocabulary tracking.

## Core Argument

The bottleneck in agent-assisted development is no longer writing code. It's understanding what should be written. That work is slow, and it has to be.

## The Annotation Cycle

Morning reading in coffee shops (books, papers, Deep Research outputs) away from a browser. Agent produces design document or implementation diary → upload to e-ink tablet → read carefully with pen, annotating everything in margins. About 90% of annotations serve only the moment; the remainder becomes a filtered list of clarification requests, corrections, vocabulary investigations, and file references.

The physical act of forming letters slows him down enough to sit with a thought longer than he would on screen. This creates "a different kind of cognitive space — slower, less reactive, more generative."

## Vocabulary Tracking as Quality Control

Every word an LLM generates is "basically a query of the training corpus" — fetched and recombined. Words like "Controller, Manager, Registry" get imported from training data even when they don't map to real concepts in the codebase. He tracks this to prevent "jargon drift before it compounds into architectural bloat."

## The Literacy Metaphor

Agent-produced design documents function as "a literate programming document" — containing file references, API signatures, code snippets, and pseudocode. "Having a literate programming for literally every commit is life-changing."

## Prompt Engineering

Typing prompts manually forces decision-making about what matters. The full cycle — design → review on paper → refine → implement — spans "2 or 3 20-minute Codex runs," but the review step happens at handwriting speed. "A mediocre prompt produces mediocre architecture at high speed."

## Tools Referenced

- docmgr: document management across projects
- remarquee: integration with e-ink tablet
- Skills repository for ticket-research workflows
- Kagi search for non-algorithmic content
- JetBrains IDE
