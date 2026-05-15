# TextForge Case Study

Aaron Stannard's detailed walkthrough of building TextForge -- an AI email automation tool that passed Google's CASA2 security audit -- using Claude Code with LLMs writing nearly all the code. The real story isn't the code; it's the six-layer discipline that made LLM-generated code production-worthy.

---

## The Product

TextForge automates email-driven sales pipeline work for Petabridge (Stannard's Akka.NET company). The stack: ASP.NET Core, Akka.NET actors, PostgreSQL, Gmail API, Blazor UI. The critical design choice: **human-in-the-loop approval** for all AI-drafted emails. Agents can create and submit drafts but cannot approve their own work -- a permission model baked into the API token scopes.

## The Six-Layer Approach

### 1. Planning Mode First

Stannard's first substantive commit was a 400-line CLAUDE.md, not code. It covered architecture, components, and six implementation phases. Most developers skip this step and get poor results. Planning mode forces the LLM into requirements gathering instead of premature implementation.

### 2. Reference Architectures

He fed Claude a production-grade Akka.NET app (DrawTogether.NET) as a structural template. The domain was different but the technology patterns were reusable. This bootstraps greenfield projects without starting from zero -- the LLM gets proven patterns, not blank-page hallucination.

### 3. Agent Skills

Markdown files encoding hard-won knowledge for reuse across sessions. Stannard's rule: codify a solution only after hitting the same non-trivial problem twice. His test email skill (Mailpit + .NET Aspire to capture SMTP locally) is a good example of domain-specific tooling that compounds.

### 4. PRDs for Permissions and Features

Fine-grained product requirements documents for specific features. The token permissions PRD is notable: `drafts:create/read/update/submit` for agents, `drafts:approve/reject` restricted to humans only. Principle of least privilege applied to the human-AI boundary.

### 5. Verification Pipeline (Evolved)

The pipeline changed significantly during development:

- **Started with:** Roslyn analyzers, `TreatWarningsAsErrors`, `.editorconfig`, unit tests
- **Dropped:** `.editorconfig` enforcement (consumed tokens on trivial formatting)
- **Dropped:** Blind trust in LLM-authored tests (coverage without depth)
- **Added:** Aspire MCP server for Playwright browser testing with OpenTelemetry tracing
- **Added:** `/dev-login` endpoint to bypass OAuth in testing
- **Added:** Snapshot tests (Verify library) capturing rendered email output -- regressions caught by diff, not manual test logic

The snapshot testing insight is particularly sharp: once the system works end-to-end, lock the outputs. Any change that alters output must explicitly update the snapshots.

### 6. Incremental Refactoring

LLMs generate "junk drawer" code organization by default (`TextForge.Models.Enums`). Periodic cleanup sprints reorganized to vertical slices by feature (`TextForge.Drafts.Approvals`). Unexpected benefit: vertical slices made code easier for Claude to navigate too.

## Key Quotes

> "Most developers receive poor LLM results by skipping planning."

> The vertical slice reorganization "simplified code exploration for both humans and Claude."

## The Scaling Problem

All six layers worked -- but every one required Stannard sitting at the keyboard. The essay ends by noting that scaling to unattended development requires [[Ralph]]-style autonomous loops and more robust verification pipelines. This is Part 2 of a series; the hard part comes next.

## Critical Analysis

This is the most honest practitioner account of LLM-assisted greenfield development I've seen. Where most "I built X with AI" posts focus on speed, Stannard focuses on the verification infrastructure that made the speed safe. The CASA2 audit passing is real evidence, not vibes.

Three things stand out:

**The editorconfig retreat is underrated.** He discovered that formatting enforcement burned tokens without improving quality -- a concrete example of the [[Feedback Loop is All You Need]] principle applied in reverse. Not all linting is equal; some deterministic checks create more noise than signal when the author is an LLM.

**Snapshot testing as the verification endgame.** Rather than writing detailed assertions for every scenario, lock the known-good output and diff against it. This is exactly the right level of verification for LLM-generated code: you don't need to understand the implementation, just that its output hasn't changed.

**The vertical slice discovery.** Reorganizing code for human readability accidentally made it better for LLM navigation too. This suggests that the "code is for machines now" thesis has a nuance: well-organized code helps both species. Feature-sliced architecture is a shared cognitive optimization.

The missing piece is cost. Stannard doesn't mention token spend, wall-clock time, or how the economics compare to writing it by hand. For a case study this thorough, the absence is conspicuous.

## Cross-Links

- [[Compound Engineering]] -- Stannard's six layers are the compound loop made concrete
- [[Specifications as the Product]] -- CLAUDE.md and PRDs as the durable artifacts
- [[Ralph]] -- the autonomous loop Stannard points to for the next phase
- [[Feedback Loop is All You Need]] -- verification pipeline as the core discipline
- [[Harness Engineering]] -- Böckeler's framework maps directly onto Stannard's pipeline evolution
- [[dotnet Slopwatch]] -- .NET-specific anti-cheat that would complement Stannard's Roslyn analyzers
- [[Slowing the Fuck Down]] -- planning mode as deliberate friction
- [[Radical Accountability]] -- AI eliminates resource excuses; taste is what's left
- [[Guardrails and Feedback Loops]] -- synthesis of the verification philosophy
- [[Agent Coding Workflow]] -- where this fits on the maturity spectrum
- [[MinMax Skills]] -- skill library pattern that Stannard's incremental approach echoes
- [[How Boris Uses Claude Code]] -- creator's workflow vs. practitioner's workflow
- [[Pre-Commit Lint Checks]] -- the linting infrastructure Stannard evolved past

---
*Sources: [[raw/software-2.0-case-study-textforge]]*
*Last updated: 2026-05-14*
