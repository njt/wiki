---
url: https://swamp-club.com/behavior
title: "Behavior-Centric AI Design"
author: Adam Jacob
date_fetched: 2026-10-18
date_published: 2026-10-18
topics:
  - agent-architecture
  - ai-product-and-business
---

# Behavior-Centric AI Design

Adam Jacob (of Swamp) argues that the industry's default "agent-centric" design — an LLM deciding the control flow of the application — produces monolithic, fragile, token-hungry systems that rarely clear ROI, and proposes instead a "behavior-centric" pattern: decompose intent into reusable, typed, composable behaviors that own their data and call frontier intelligence only where a job actually needs it.

## The agent-centric critique

Since ChatGPT and the 2023 function-calling API, the default architecture has been "the LLM is the heart of the program" — context, tools, and reasoning combine inside one agent boundary that decides what happens next. Jacob argues every scaling add-on (RAG, skills, MCP, gateways, routers, sub-agents, memory schemes) has been layered on top of a design whose core problem is untouched: probabilistic control flow. A trip-planning agent that sometimes decides not to book a hotel is correct only if the LLM makes the right decision at every point — which "cuts directly against the nature of the LLM."

The usual fix is more agents to validate agents, which drives the cost spiral: every agent spends tokens gathering context and reasoning about next steps, then needs yet more agents to check it. The result matches the dismal stats (Gartner's 40%+ agentic-project cancellation prediction; McKinsey's 37% of enterprises showing any financial impact). A subtle point: fragile agent systems are tolerable to their designers, who know why they fail and just retry — but become unacceptable at organizational scale, both socially (unforgiving users) and technically (widening deployment, observability, and security footprint).

## The behavior-centric proposal

Build a corpus of reusable behaviors with typed, validated inputs and outputs, structured data conforming to versioned schemas, isolated credentials — with the platform enforcing these properties at runtime, not by convention. A "plan-trip" workflow composes behaviors; only the first step (extracting trip parameters) calls a frontier model, and that call is scoped to a single well-defined job rather than driving control flow. No tokens are spent on orchestration; everything after extraction is deterministic code. The agent-facing surface shrinks to a single tool call into the workflow.

Claimed results: 14x fewer tokens and 6x lower cost per trip plan (footnoted comparison of a LangChain agent vs a Swamp implementation), and for production code-review workflows an 8x token reduction and 3x speed improvement. The deeper benefit is the compounding corpus: behaviors built by DevOps get reused by app teams and SREs.

## The vendor caveat

The essay is also a pitch: Swamp is "the proof," and the harness uses frontier intelligence at build time to encode intent into behaviors rather than at runtime to decide what to do. The pattern is sound and rhymes with what other practitioners have converged on; the benchmarks are the vendor's own, unoptimized-vs-unoptimized but self-selected.
