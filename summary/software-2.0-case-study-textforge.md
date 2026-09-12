---
url: https://aaronstannard.com/software-2.0-case-study-textforge/
title: "Software 2.0: Planning and Verifying a Greenfield Project"
author: Aaron Stannard
date: 2026-03-13
fetched: 2026-05-14
topics:
  - agent-coding-workflow
---

# Software 2.0: Planning and Verifying a Greenfield Project

Aaron Stannard discusses core techniques for building greenfield projects with large language models, using TextForge — an AI email automation tool — as a practical case study.

## Overview

In his previous essay "Software 2.0: Code is Cheap, Good Taste is Not," Stannard introduced Software 2.0 as an emerging practice where LLMs implement code while humans serve as planners, designers, and verifiers. TextForge passed a CASA2 audit required by Google for verification, demonstrating that despite LLMs writing nearly all the code, rigorous planning and verification processes enabled it to meet high security standards.

## The Project: TextForge

Stannard runs Petabridge, a products and services company built around Akka.NET. Facing email-driven operational tasks consuming approximately two hours daily, he decided to automate his sales pipeline management using Pipedrive CRM integration.

**Initial Components:**
- A fully-featured Pipedrive CLI using .NET Ahead-of-Time compilation
- A reusable Claude Code workflow called "/sales-inbox" to query Pipedrive and fetch overdue tasks

However, these addressed only 40% of the challenge. Since 99% of deal-closing occurs via email, Stannard needed a separate email handling solution with a critical safety feature: human-in-the-loop approval.

**Human-in-the-Loop Design:**
Before LLMs send emails to customers on a user's behalf, drafts require human review and approval. This verification layer prevents AI errors from damaging customer relationships. The system includes a self-hosted .NET application with a draft approval and rejection interface.

## Planning and Designing the Project

Rather than immediately coding, Stannard's first substantive commit was authoring a CLAUDE.md file outlining project goals and implementation approach. This 400-line document included:

- Production-ready email gateway overview
- Architecture diagrams showing Nginx reverse proxy, ASP.NET Core service, Akka.NET actor system, PostgreSQL persistence, and Gmail API integration
- High-level components including Blazor UI for approvals and API endpoints
- Six implementation phases from foundation through observability and deployment

**Planning Mode Importance:**
Stannard emphasizes that most developers receive poor LLM results by skipping planning. In planning mode, the LLM focuses on requirements gathering and implementation planning rather than immediate coding. This constraint forces both the AI and developer toward structured thinking before implementation begins.

The TextForge plan outlined MVP features including Google OAuth authentication, API token management, email draft management, approval workflows, Gmail integration, HTML email editing, Slack notifications, Pipedrive context display, edit history tracking, actor-based workflows, and persistent state with exactly-once delivery guarantees.

**Reference Architectures and Samples:**
Stannard provided DrawTogether.NET, a production-grade Akka.NET reference application, as a structural model. While DrawTogether serves different domain requirements, its technology choices and patterns aligned with TextForge's needs. Reference samples help LLMs bootstrap greenfield projects by providing proven architectural patterns without requiring reimplementation from scratch.

**Agent Skills:**
Skills are markdown files providing agents with contextual instructions about when and how to apply specific techniques. Stannard created a skill for testing email integration using Mailpit and .NET Aspire, capturing SMTP traffic locally without sending real messages. He notes that creating skills incrementally — codifying solutions after encountering non-trivial problems more than once — maximizes their value across projects.

## Requirements and Refinements

With planning foundations established, Stannard drafted product requirements documents for specific features. A notable PRD addressed token permissions, defining fine-grained API scopes:

**Draft Permissions:**
- `drafts:create`, `drafts:read`, `drafts:update`, `drafts:submit` (permissible for AI agents)
- `drafts:approve`, `drafts:reject` (explicitly restricted from AI tokens to maintain human oversight)

This principle of least privilege prevents agents from approving their own drafts. Modern TextForge has evolved this design, with most users now employing MCP OAuth instead of explicit bearer tokens, but the underlying permission model remains foundational.

## Implementation

Stannard's development followed a live pairing model: he provides speech-to-text prompts, Claude implements features, he reviews and tests, reports bugs, and iterates. This cycle maintained rapid development velocity while ensuring human oversight.

**Verification Pipeline Evolution:**

*Initial approach:*
- C# compiler with Roslyn Analyzer rules and `TreatWarningsAsErrors` enabled
- `.editorconfig` and `dotnet format` linting
- Automated unit and Aspire integration tests

*Refinements:*
Stannard found that `.editorconfig` enforcement consumed excessive tokens on trivial formatting issues and was discontinued. LLM-authored tests provided coverage but often lacked depth.

*Improvements:*
- Enabled Aspire MCP server allowing Claude to conduct Playwright-based browser testing with full OpenTelemetry tracing
- Added `/dev-login` endpoint bypassing Google OAuth for testing (important given 2FA protection on accounts)
- Created Claude Code skills encouraging click-testing after substantive changes
- Supervised testing sessions with real Google accounts, helping debug OAuth token and scope issues

**Snapshot Testing:**
Once end-to-end functionality achieved completion, Stannard implemented snapshot tests using the Verify library. Tests captured rendered email output including signatures, MIME attachments, and formatting. Any pull requests modifying test output require corresponding approval file updates. This approach caught numerous regressions without requiring detailed test logic for every scenario.

**Incremental Refactoring:**
Large language models excel at initial implementation but require periodic maintenance oversight. Stannard conducted technical cleanup sprints involving:
- Replacing stringly-typed primitives with enums and value objects
- Removing duplicate code and anemic tests
- Refactoring from "junk drawer" organization (e.g., `TextForge.Models.Enums`) to vertical slices by feature (e.g., `TextForge.Drafts.Approvals`)

The vertical slice reorganization provided unexpected benefits: it simplified code exploration for both humans and Claude when working within specific feature areas.

## Scaling Beyond Attended Development

The described techniques — planning mode, reference architectures, agent skills, verification pipelines, snapshot testing, and incremental refactoring — successfully transitioned TextForge from blank repository to working product. However, each required human supervision at every step.

Scaling toward full SaaS required delegating larger multi-step changes with reduced direct oversight, necessitating tools like OpenProse and RALPH loops for breaking changes into plannable, verifiable chunks executable with minimal human intervention. This scaling phase demands more robust verification pipelines, detailed implementation plans, and comprehensive product requirements documents, which Stannard addresses in subsequent essays.

**Key Takeaway:** Software 2.0 success requires rigorous planning discipline before implementation, reference materials guiding architectural decisions, incremental verification mechanisms, and strategic technical maintenance — transforming LLM coding from potentially chaotic to production-ready.
