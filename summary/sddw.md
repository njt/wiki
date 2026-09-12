---
url: https://github.com/sermakarevich/sddw
title: SDDW — Spec-Driven Development Workflow for Claude Code
author: Sergii (sermakarevich)
date_fetched: 2026-06-15
date_published: 2026-05
topics:
  - agent-coding-workflow
---

# SDDW — Spec-Driven Development Workflow

## Summary

SDDW is a Claude Code plugin implementing a 7-step spec-driven development pipeline: Requirements → Code Analysis (optional) → Design → Taskify → Implement → Verify → Self-Improve. Each step is assembled from four modular components: commands (thin entry points), instructions (process rules), questionnaires (dialog flows), and specs (output templates). The core insight: specifications are the primary artifact; code is a verified output. The pipeline keeps each step within a single context window (`/clear` between steps) and uses a three-phase dialog pattern (Discover → Research & Propose → Confirm & Generate) with dual interaction modes: interactive guided dialog or fully autonomous `--auto`.

## Architecture

### Four-Component Modular Structure

Every workflow step is assembled from four independent markdown files:

| Component | Purpose | Count |
|-----------|---------|-------|
| `commands/` (10 files) | Thin entry point with YAML frontmatter + `@references` to wire components together | 228 total lines |
| `instructions/` (11 files) | Process rules — what to do, in what order, with formal constraints | 941 total lines |
| `questionnaires/` (9 files) | Dialog guidance — how to interact with the user, one question at a time | 829 total lines |
| `specs/` (7 files) | Output format templates with examples and rules | 735 total lines |

The command file is the glue: it `@`-references the other three components, sets the NEXT STEP suggestion, and defines the argument interface.

### 7-Step Pipeline

1. **Requirements** (`/sddw:requirements <feature-name>`) → `.sddw/<feature>/requirements.md`
2. **Code Analysis** (`/sddw:code-analysis`) → `.sddw/code-analysis.md` (optional, shared across features)
3. **Design** (`/sddw:design`) → `.sddw/<feature>/design/design.md`
4. **Taskify** (`/sddw:taskify`) → `.sddw/<feature>/design/tasks/task-N-slug.md`
5. **Implement** (`/sddw:implement --task <N>`) → code commits + `.sddw/<feature>/implement/tasks/task-N-slug.done.md`
6. **Verify** (`/sddw:verify`) → `.sddw/<feature>/verify/report.md` + optional remediation tasks
7. **Self-Improve** (`/sddw:self-improve`) → `.sddw/<feature>/self-improve/report.md` + workflow file patches

Plus two utility commands: Chat (low-ceremony interaction with existing features) and Help (workflow overview, feature listing, status).

### Artifact Directory Structure

```
.sddw/
├── code-analysis.md              # shared across features
└── <feature-name>/
    ├── requirements.md
    ├── design/
    │   ├── design.md             # cross-cutting architecture
    │   └── tasks/
    │       ├── task-1-<slug>.md   # hybrid: inline + design.md refs
    │       ├── task-2-<slug>.md
    │       └── task-<N>-fix-<slug>.md  # remediation tasks
    ├── implement/
    │   └── tasks/
    │       └── task-<N>-<slug>.done.md
    ├── verify/
    │   └── report.md
    └── self-improve/
        └── report.md
```

## Key Techniques

### 1. Three-Phase Dialog Pattern

Every step follows the same three-phase flow:

- **Discover**: Understand context through one-question-at-a-time dialog. Follow the thread, challenge vagueness, make abstract concrete. Never dump multiple questions.
- **Research & Propose**: Research SOTA/codebase/domain, then propose each spec section as ranked options with rationale. One section at a time, wait for approval.
- **Confirm & Generate**: Summarize what will be written, get final confirmation, then write the spec file.

In `--auto` mode, all three phases execute autonomously using best judgment.

### 2. Hybrid Task Files

Task files are "hybrid" — they inline task-specific content (Files, Acceptance Criteria, Done Criteria) but **reference** cross-cutting content (Architecture, Data Models, Design Decisions) via `design.md §<Section>` pointers rather than duplicating it. This is the most architecturally interesting decision: it avoids stale duplication while ensuring the implementation agent always has full context by loading both files.

### 3. Deviation Handling (4-Rule Classification)

During implementation, deviations from the spec are classified into four rules:

| Rule | Trigger | Action | Permission |
|------|---------|--------|------------|
| 1: Bug | Broken behavior, errors | Fix → test → verify → document | Auto |
| 2: Missing Critical | Missing error handling, validation, auth | Add → test → verify → document | Auto |
| 3: Blocking | Missing deps, wrong types, broken imports | Fix blocker → verify → document | Auto |
| 4: Architectural | New DB table, schema change, switching libraries | STOP → present to user → document | Ask user |

Priority: Rule 4 (STOP) > Rules 1-3 (auto) > unsure → Rule 4. Rule 4 overrides even `--auto` mode.

### 4. FR-ID Traceability Chain

Functional Requirement IDs (FR-01, FR-02, etc.) trace through the entire lifecycle:

- **Requirements**: FR-01 defined with acceptance criteria
- **Design**: design.md lists FR-IDs covered in Trace section; every design element traces to an FR
- **Taskify**: every task traces to FR-IDs; every FR appears in at least one task
- **Implement**: commit messages reference FR-IDs
- **Verify**: FR-by-FR pass/fail with specific test names and error messages
- **Self-Improve**: remediation tasks classified by origin (requirements/design/implementation/external)

### 5. Remediation Loop

Verification can create new task files in `design/tasks/` (continuing numbering) for failed FRs. These use the same hybrid format and are implementable via `/sddw:implement`. The loop repeats — implement remediation tasks → re-verify → until all checks pass. Each remediation task records Severity, Origin (where the issue was introduced), and Evidence.

### 6. TDD Protocol with User-Chosen Approach

The requirements step asks the user to choose among four testing approaches: TDD (write failing tests first), Test-after (implement first, add tests after), Selective TDD (TDD for business logic, skip for config/glue), and No tests. The chosen approach carries through to the implement step, which follows the RED → GREEN → REFACTOR cycle with a heuristic: "Can you write `expect(fn(input)).toBe(output)` before writing `fn`? If yes, use TDD."

### 7. Commit Protocol

One task = one commit (but TDD tasks produce 2-3: test commit, feat commit, optional refactor commit). Structured messages: `type(feature-name): description (FR-01, FR-02)`. Individual file staging only — never `git add .`. Every commit references FR-IDs.

### 8. Self-Improving Workflow

The Self-Improve step is a meta-step that analyzes the artifact trail (deviations, difficulties, remediation origins, uncovered criteria) to propose concrete improvements to the workflow files themselves. Proposals include diff previews targeting specific instruction/questionnaire/spec files. The workflow evolves with every feature — gaps found during one feature prevent the same issues in the next.

### 9. Context Window Management

Each step instructs the user to `/clear` between steps. This is a design constraint, not a bug: by splitting work into discrete steps, each operates within a focused context where models are more accurate. The artifacts (markdown files) are the persistent memory between steps.

## Design Decisions

1. **Modular components over monolithic prompts**: Separating commands/instructions/questionnaires/specs into independent files enables independent evolution, reuse (specs/design-task.md is shared by implement, taskify, and verify), and targeted self-improvement patches. The trade-off: more files to maintain, potential for inconsistency between components.

2. **Split design and taskify (with combined alias)**: The default path separates architecture from task breakdown, enabling design review before committing to task structure. The combined alias `/sddw:design_and_taskify` exists for small features. Trade-off: two steps vs one, but the architectural review gate is worth it for non-trivial work.

3. **File-based artifacts over a database**: All specs are markdown files in `.sddw/`. This is git-friendly, human-reviewable, and accessible to any tool. The cost: no structured querying, no automated traceability beyond grep.

4. **Interactive default over auto-only**: The default is guided dialog — one question at a time, every section confirmed. `--auto` exists for vibecoding. Trade-off: more user involvement, but the specs become the user's artifact, not the agent's.

5. **Per-feature directory structure over flat**: Each feature gets its own directory under `.sddw/`. This isolates features but makes cross-feature pattern analysis harder. Code analysis (`code-analysis.md`) is the one shared artifact.

6. **Self-improvement as post-hoc analysis**: The workflow improves after each feature, not during. Trade-off: doesn't prevent the current feature's issues, but avoids disrupting execution with meta-work.

7. **Code analysis is optional and shared**: Unlike every other step which is feature-scoped, code analysis is project-scoped and optional. This reduces friction for greenfield projects but risks design decisions based on assumptions about the codebase.

8. **Chat as an escape hatch**: The Chat command bypasses the full questionnaire ceremony for quick edits and questions. It's a pragmatic admission that not every interaction needs the full spec ceremony. Guarded by a heuristic: if >3 files or >1 commit, redirect to full workflow.

## Installation

```bash
git clone https://github.com/sermakarevich/sddw.git ~/.claude/sddw
cd ~/.claude/sddw && bash bin/install.sh
```

The installer clones to `~/.claude/sddw` and copies command files to `~/.claude/commands/sddw/`. Development mode (`--local`) symlinks instead. The plugin.json registers metadata (name, version, description, author).
