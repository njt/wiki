---
url: https://burr.apache.org/
title: "Apache Burr (Incubating)"
author: Apache Software Foundation / DAGWorks Inc.
date_fetched: 2026-06-11
date_published: unknown
---

# Apache Burr — Full Page Content (WebFetch)

## Overview

Apache Burr is an open-source Python framework (currently undergoing Apache Incubation) for building AI applications that make decisions. It describes itself as enabling developers to "build reliable AI agents and applications" ranging from simple chatbots to complex multi-agent systems. The framework is "Pure Python, no magic."

**Current status:** Apache Incubating project (not yet fully endorsed by the ASF). Sponsored by the Apache Incubator. Copyright assigned to The Apache Software Foundation (© 2026).

## Core Positioning

Burr competes with or complements frameworks like LangChain, CrewAI, AutoGen, and Agency Swarm. Testimonials on the page directly compare it to LangChain, with one user quoted as saying "Moving from LangChain to Burr was a game-changer!" and another stating Burr "provides a more robust framework for designing complex behaviors" versus those alternatives.

## Key Features (6 listed)

1. **Simple Python API** — Applications are defined as actions and transitions. No DSL, no YAML; just "Python functions and decorators." The example shows `@action(reads=[...], writes=[...])` decorators on functions, with `ApplicationBuilder` to compose them.

2. **Built-in Observability** — A Burr UI allows real-time monitoring, debugging, and tracing of every application step, showing state changes as they occur.

3. **Persistence & State Management** — State is automatically persisted to disk, databases, or custom backends. Applications can be resumed from where they left off. One user highlighted that "state management part is really helpful for creating state snapshots and build debugging."

4. **Human-in-the-Loop** — Execution can be paused, waiting for human input at any step. Designed for approval workflows and interactive agents.

5. **Branching & Parallelism** — Supports running actions in parallel, fan-out/fan-in patterns, complex DAGs, and composing sub-applications for modular design.

6. **Testing & Replay** — Past runs can be replayed, individual actions unit-tested, and state transitions validated.

## Architecture (from code example)

The framework is built around:

- **`State` objects** — mutable state containers passed between actions
- **`@action` decorator** — marks a function as an action, declaring which state keys it reads from and writes to
- **`ApplicationBuilder`** — fluent builder pattern to configure actions, transitions, state, and tracking
- **`app.run()`** — executes the application with halt conditions and input parameters

A minimal chatbot example shows a single `chat` action with a self-loop transition (`("chat", "chat")`), initialized with an empty message list, using a local tracker, and halted after the chat action completes.

## Integrations (listed on page)

The page shows these integrations, organized by category:

- **LLMs:** OpenAI, Anthropic, Instructor
- **Frameworks:** LangChain, Hamilton, Haystack
- **Validation:** Pydantic
- **UI:** Streamlit
- **Serving:** FastAPI
- **Storage:** PostgreSQL

The page states: "Burr integrates with the tools and frameworks you already use. No lock-in, no wrappers."

## Documentation Structure

The footer links to:
- **Documentation** — `/docs` (primary docs site)
- **Examples** — GitHub repo at `/apache/burr/tree/main/examples`
- **YouTube** channel (@DAGWorks-Inc)
- **GitHub Issues** (Roadmap)
- **GitHub Releases** (Changelog)
- **Contributing guide** — `CONTRIBUTING.rst`

## Community Channels

- **Discord** — "Chat with maintainers and the community" (discord.gg/6Zy2DwP4f3)
- **GitHub** — Star the repo, file issues, contribute (github.com/apache/burr)
- **Twitter/X** — @burr_framework for updates and announcements
- Metrics shown: k+ GitHub Stars, 0k+ PyPI Downloads, 0+ Discord Members (note: numbers appear to be placeholder zeros on the page, suggesting dynamic counters)

## User Testimonials / Use Cases

Organizations mentioned using Burr:

1. **Peanut Robotics** — AI-driven robots; CTO Ashish Ghosh praised Burr's "elegant yet comprehensive state management solution" for rolling out robots with AI decision-making, contrasting it with "other obfuscating LLM frameworks."

2. **Watto.ai** — Founder Ishita called using Burr "a no-brainer if you want to build a modular AI application," specifically loving "their UI which makes debugging a piece of cake."

3. **Paxton AI** — Staff Software Engineer Matthew Rideout expressed surprise at how Burr "predicted this exact need" with "no weird esoteric concepts just because it's AI."

4. **Provectus** — Senior Solutions Architect Rinat Gareev valued Burr's state management for "creating state snapshots and build debugging, replaying and even building evaluation cases."

5. **CognitiveGraphs** — Co-founder Hadi Nayebi compared Burr favorably to "LangChain, CrewAi, AutoGen, Agency Swarm, etc" for designing complex behaviors.

6. **TaskHuman** — DS Architect Aditya K. said pivoting "our entire codebase to it" from LangChain was straightforward.

7. **Reddit (r/LocalLlama)** — An anonymous developer suggested LangChain wasn't truly production-ready and advised: "Honestly, take a look at Burr. Thank me later."

## Footer / Legal Structure

The footer organizes into four sections:
- **Project:** Features, Integrations, Roadmap, Changelog
- **Resources:** Documentation, Examples, YouTube
- **Community:** Discord, GitHub, Twitter/X, Contributing
- **Apache:** Homepage, Incubator, License, Thanks, Security, Sponsorship, Privacy Policy, Events

The incubation disclaimer notes that while "incubation status is not necessarily a reflection of the completeness or stability of the code," it "does indicate that the project has yet to be fully endorsed by the ASF." Trademarks for Apache Burr, Burr, Apache, the Apache feather logo, and the Apache Burr project logo are asserted.
