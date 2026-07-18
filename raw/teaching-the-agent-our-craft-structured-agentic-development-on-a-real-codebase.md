---
url: https://8thlight.com/insights/teaching-the-agent-our-craft-structured-agentic-development-on-a-real-codebase
title: "Teaching the Agent Our Craft: Structured Agentic Development on a Real Codebase"
author: Alex Haldeman
date_fetched: 2026-07-18
date_published: 2026-07-06
site: 8th Light
---

# Teaching the Agent Our Craft: Structured Agentic Development on a Real Codebase

**Author:** Alex Haldeman, Lead Engineer at 8th Light
**Published:** July 6, 2026

## The Mission

A startup had developed a clinically proven approach to treating neuroplastic chronic pain through a coach-led model that was lower-cost than conventional treatment. The problem was scale — the in-person model couldn't meet demand, and there were too few coaches. 8th Light was brought in to build a digital platform capable of delivering the program to every patient who needed it.

## Development Philosophy

The team applied the same disciplined approach to agentic development that they use for any software engagement: TDD, clean architecture, and code designed for change. Haldeman identifies three recurring failure modes with agents: they cover happy paths and miss edge cases; context windows fill with stale reasoning that works against the agent; and without explicit conventions, output lacks team consistency.

The framework was inspired by Tyler Burleigh's Research-Plan-Implement (RPI) model, which separates research, planning, and implementation into distinct phases with review cycles between each. The team adapted RPI into their Claude Code workflow, aiming to keep product managers and designers genuinely involved at each transition, not just developers.

## The Harness

The entire scaffolding lives in a handful of files — nothing but a `CLAUDE.md`, an `.mcp.json`, and a `.claude/` directory. None of it is application code. The file tree looks like this:

```
pain-management-app/
├── CLAUDE.md
├── .mcp.json
└── .claude/
    ├── rules/
    ├── skills/
    ├── agents/
    ├── hooks/
    └── commands/
```

### CLAUDE.md — The Knowledge Root

This file provides project context that applies everywhere: what the team is building, conventions, critical rules, and pointers to the rest of the framework. The critical rules are short and encode habits a team usually repeats in code review. Four rules are listed explicitly:

1. Don't write speculative code — every function, endpoint, or field must be used by something in the same change.
2. Tests come before implementation and assert behavior, not internals.
3. Reuse an existing component or utility before introducing a new one.
4. Comments explain WHY, not WHAT.

Haldeman emphasizes that "getting this root right matters more than any single rule beneath it."

### Path-Scoped Rules

Rather than dumping everything into one file, the team used Claude Code's path-scoped rules, which load automatically based on which files the agent is working with. Each rules file declares governed paths in its frontmatter.

**Example frontmatter:**
```yaml
---
paths:
  - "frontend/src/**/__tests__/**/*"
  - "frontend/src/**/*.test.*"
---
```

A frontend testing rule specifies which Testing Library selectors to use and when:

- `getBy*`: "Element is already in DOM (sync, throws if missing)."
- `findBy*`: "Wait for element to appear (async, throws if missing). Use instead of `waitFor` + `getBy`."
- `queryBy*`: "Assert element is NOT present (sync, returns null)."

Haldeman notes this is "the kind of guidance a developer picks up after a few code reviews" — without it, an agent produces tests that work but don't match team conventions.

The backend architecture rule demonstrates the exact dependency wiring the agent should produce:

```python
async def get_playlist_service(
    session: Annotated[AsyncSession, Depends(get_session)],
    catalog_client: Annotated[CatalogClient, Depends(get_catalog_client)],
) -> PlaylistService:
    repository = PlaylistRepository(session)
    return PlaylistService(repository, catalog_client)

PlaylistServiceDep = Annotated[PlaylistService, Depends(get_playlist_service)]
```

With that rule loaded, "the agent writes a new service the way the team writes the rest: HTTP concerns in the router, business logic in the service, SQL confined to the repository."

### MCP Integration — Bringing Product and Design In

The team used Model Context Protocol (MCP) to wire in Linear (backlog management) and Figma (design system and UI flows). Rather than copying context from tickets or design files into prompts, the agent reads from and acts on those tools directly.

The `.mcp.json` file was committed to the repository so any developer cloning the repo gets all integrations immediately. MCP servers weren't limited to collaboration tools — they connected EHR and CMS services the same way. One notable example: a developer used the Figma MCP to pull the design system directly into the agent's context and built a 1-to-1 match using custom design tokens backed by real components. Haldeman writes that "re-syncing was a single command rather than a manual audit."

## Building the Workflow — Two Core Skills

### /story-writer

This skill handles the research and planning phases. Given a Linear ticket ID, it reads the existing description; without one, it creates a new ticket. Either way, it explores the codebase before drafting — not to write code, but so the implementation steps reference real domain folders and constraints.

The agent conducts an interview loop, asking clarifying questions until requirements are specific enough to write acceptance criteria. The structured output format has three sections:

- **Overview** — purpose and scope in a few sentences
- **Implementation Steps** — meaningful units grouped by concern, pitched at what to build, not file-by-file how
- **Acceptance Criteria** — checks someone can perform in the running app

A key habit: including the Figma MCP link for the relevant design file in the story description, so when `/tdd-build` reads the ticket, it pulls the actual designs into context.

### /tdd-build

This skill reads the Linear ticket, explores relevant codebase areas, enters plan mode, and proposes implementation as numbered TDD cycles. Each cycle specifies a test file, behavioral spec, exact command to run, and a RED gate reason — why the test should fail before any implementation exists.

**Example cycle from the article:**

```
Cycle 1: GET /playlist/{id} returns playlist with tracks
- Test file: backend/tests/playlists/test_playlist_router.py
- Test spec: GET /playlist/{id} returns 200 with {id, name, tracks[{id, name}]}
- Test command: cd backend && python -m pytest tests/playlists/test_playlist_router.py -v
- RED gate: endpoint does not exist
- Implementation: add GET /playlist/{id} router, service, and repository
- Files: backend/src/playlists/router.py (new), service.py (new), repository.py (new)
```

Nothing gets written until the developer reviews and approves the plan.

For each cycle, the main agent dispatches a `test-writer` subagent with only the behavioral spec. The subagent starts from a fresh context window with no knowledge of the implementation. Haldeman explains that "a test written with the implementation already in view tends to describe that implementation, not the behavior we actually care about." The subagent runs the tests, confirms failure for the right reason, and returns a RED gate report. Only then does the main agent write the minimum code to pass.

### The test-writer subagent configuration

```yaml
---
name: test-writer
description: Writes failing tests for a single TDD cycle, isolated from any implementation.
tools: Read, Glob, Grep, Write, Edit, Bash
hooks:
  PreToolUse:
    - matcher: "Edit|Write"
      hooks:
        - type: command
          command: "python3 \"$CLAUDE_PROJECT_DIR/.claude/hooks/restrict_test_writer_paths.py\""
---
```

The hook fires on every `Edit` and `Write` the subagent makes, checking against an allowlist:

```python
ALLOWED_PATTERNS = (
    re.compile(r"(?:^|/)backend/tests/"),
    re.compile(r"(?:^|/)__tests__/"),
    re.compile(r"\.test\.[^/]+$"),
    re.compile(r"(?:^|/)frontend/src/testing/"),
)
```

The check only runs for the test-writer; all other agents pass through untouched. When a path fails, a structured decision is returned before the file is touched. Haldeman notes: "The prompt states the intent, and the hook is what actually enforces it."

## Results

The clearest outcome was **how much a small team could carry**. Two developers took on what would normally require more hands: a multi-tenant platform, an AI coaching companion bounded by safety constraints, and a data architecture designed to keep sensitive information out of reach. The harness made that possible — "the conventions, the architecture rules, and the test-first cycle were already encoded for the agent to follow."

Speed wasn't the only metric. Haldeman notes that scope stayed under control rather than being quietly dropped: "That is usually what separates genuine progress from debt you only discover later." The same structure that kept the agent honest — research before planning, a failing test before implementation — kept the codebase coherent under pressure.

Wiring Linear and Figma through MCP "collapsed the lag between a design decision, a product priority, and the code that implemented it." Re-syncing a built component against an updated design became a single command instead of a manual audit.

Haldeman is explicit that "this approach was an accelerant to our team's abilities, not a replacement." Engineers still stepped in to scaffold code when a cleaner structure was needed, then folded lessons back into the rules so the agent carried them forward. That feedback loop — "engineers kept improving the harness and the harness let them cover more ground" — is what made the approach worth keeping. What started as experimentation is now a repeatable practice.

## How to Get Started

The article closes with an invitation: "Have an idea, workflow, or challenge that has been a blocker for your business? Let's talk."

## About the Author

**Alex Haldeman** is a Lead Engineer with over a decade of experience across healthcare, finance, government, and defense. He works in a test-driven, agile style and has focused recently on AI-powered systems and agentic development, often serving as technical lead on cross-functional teams.
