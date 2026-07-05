# SDDW (Spec-Driven Development Workflow)

A Claude Code plugin by Sergii (sermakarevich) that inverts the AI coding paradigm: instead of prompting for code, you collaborate with the agent to write **specifications** — requirements, architecture, interface contracts, task breakdowns. The specs become the primary artifact: reviewable by peers, version-controlled, persistent across sessions. Code generation is a mechanical step guided by approved specs.

---

## Architecture

SDDW is a 7-step pipeline assembled from four modular markdown component types. Each step follows the same three-phase dialog pattern.

### Four-Component Architecture

Each workflow step is assembled from four independent files:

| Component | Folder | Purpose |
|-----------|--------|---------|
| **Command** | `commands/` | Thin entry point: YAML frontmatter, argument parsing, `@references` to wire the other three |
| **Instructions** | `instructions/` | Process rules — what to do, in what order, with RFC 2119 constraints |
| **Questionnaire** | `questionnaires/` | Dialog flow — how to interact with the user, one question at a time |
| **Specs** | `specs/` | Output format templates with examples and rules |

A command file is glue — e.g. `commands/requirements.md` contains nothing but frontmatter, `@~/.claude/sddw/instructions/requirements.md`, `@~/.claude/sddw/questionnaires/requirements.md`, `@~/.claude/sddw/specs/requirements.md`, and a NEXT STEP suggestion. The `@`-reference mechanism is Claude Code's native file inclusion.

### 7-Step Pipeline

1. **Requirements** (`/sddw:requirements <feature>`) → `.sddw/<feature>/requirements.md` — user stories, FRs with RFC 2119 keywords, acceptance criteria (Given/When/Then), constraints with explicit prohibitions
2. **Code Analysis** (`/sddw:code-analysis`) → `.sddw/code-analysis.md` — optional, shared across features; scans existing codebase for patterns, interfaces, flows, conventions
3. **Design** (`/sddw:design`) → `.sddw/<feature>/design/design.md` — cross-cutting architecture, data models, interface contracts, design decisions with rationale and rejected alternatives
4. **Taskify** (`/sddw:taskify`) → `.sddw/<feature>/design/tasks/task-N-<slug>.md` — dependency-ordered hybrid task files referencing design.md for cross-cutting context
5. **Implement** (`/sddw:implement --task <N>`) → code commits + `.sddw/<feature>/implement/tasks/task-N-<slug>.done.md` — TDD protocol, commit protocol, deviation handling
6. **Verify** (`/sddw:verify`) → `.sddw/<feature>/verify/report.md` — FR-by-FR pass/fail, test execution, acceptance criteria coverage, optional remediation tasks
7. **Self-Improve** (`/sddw:self-improve`) → `.sddw/<feature>/self-improve/report.md` — analyze execution trail, propose workflow file improvements with diff previews

Two utility commands: **Chat** (low-ceremony interaction with existing features, guarded by >3-files redirect heuristic) and **Help** (workflow overview, feature listing, status). Combined alias: `/sddw:design_and_taskify` runs design + taskify in one shot for small features.

### Artifact Directory Structure

```
.sddw/
├── code-analysis.md                    # shared across features (optional)
└── <feature>/
    ├── requirements.md
    ├── design/
    │   ├── design.md                   # cross-cutting: architecture, models, contracts, decisions
    │   └── tasks/
    │       ├── task-1-<slug>.md        # hybrid: task-specific inline + design.md § refs
    │       ├── task-2-<slug>.md
    │       └── task-N-fix-<slug>.md    # remediation tasks from verify
    ├── implement/tasks/
    │   └── task-N-<slug>.done.md       # completion report: commits, deviations, difficulties
    ├── verify/report.md
    └── self-improve/report.md
```

### Interaction Modes

Every step supports two modes via `--auto` flag:

- **Interactive** (default): One question at a time via `AskUserQuestion` tool. Every spec section confirmed before writing. Follow-the-thread dialog, not checklist-walking.
- **Auto** (`--auto`): Fully autonomous. All decisions made with best judgment. Specs generated directly. Architectural deviations (Rule 4) still stop and ask — this overrides `--auto`.

The safety check in `instructions/common.md` prevents garbage-in-garbage-out: if the requirements feature description is <20 words in auto mode, it downgrades to interactive.

## Key Techniques

### Three-Phase Dialog Pattern

Every step follows the same structure (defined in each questionnaire):

- **Discover**: One question at a time. Follow the thread, challenge vagueness ("users" means who? "fast" means what?), make abstract concrete. Anti-patterns explicitly called out: multiple questions at once, checklist-walking, interrogation, script-following.
- **Research & Propose**: Research SOTA/codebase/domain. Propose each spec section as 2-3 ranked options with rationale. User accepts, modifies, or provides their own. One section at a time.
- **Confirm & Generate**: Summarize what will be written. User confirms. Write the spec.

This is not a rigid script — the questionnaire says "weave questions naturally based on what's missing" and "use the user's answer to shape the follow-up."

### Hybrid Task Files (The Central Innovation)

Task files at `instructions/taskify.md` and `specs/design-task.md` are "hybrid" — they **inline** task-specific content (Files, Acceptance Criteria, Done Criteria) but **reference** cross-cutting content via `design.md §<Section>` pointers rather than duplicating architecture, data models, and design decisions. This is the most architecturally interesting decision in the workflow:

- The implementation agent loads both the task file *and* `design.md` — full context without duplication
- Design changes update one file; all tasks automatically see the update
- Trade-off: requires loading two files per task, adds a level of indirection

### Deviation Handling (4-Rule System)

Defined in `instructions/implement.md`, deviations from spec during implementation are classified into four rules:

| Rule | Trigger | Action | Permission |
|------|---------|--------|------------|
| **1: Bug** | Broken behavior, errors, type errors, security vulnerabilities | Fix → test → verify → document | Auto |
| **2: Missing Critical** | Missing error handling, validation, auth checks, input sanitization | Add → test → verify → document | Auto |
| **3: Blocking** | Missing deps, wrong types, broken imports, missing config | Fix blocker → verify → document | Auto |
| **4: Architectural** | New DB table, schema change, new service, switching libraries, breaking API | STOP → present to user → document | Ask user |

Priority: Rule 4 > Rules 1-3 > unsure → Rule 4. Rule 4 overrides even `--auto` mode — the risk of silent architectural drift is too high. All deviations are documented in completion reports.

### FR-ID Traceability Chain

Functional Requirement IDs (FR-01, FR-02, etc.) trace through the entire lifecycle:

- **Requirements**: FRs defined with RFC 2119 keywords and acceptance criteria
- **Design**: Every design element traces to ≥1 FR-ID; every FR appears in ≥1 design element
- **Taskify**: Every task traces to FR-IDs; every FR appears in ≥1 task; explicit `Depends on:` field
- **Implement**: Commit messages reference FR-IDs (`type(feature): description (FR-01, FR-02)`)
- **Verify**: FR-by-FR pass/fail with specific test names and error messages
- **Self-Improve**: Remediation tasks classified by Origin (requirements/design/implementation/external)

### TDD Protocol

The requirements step captures the user's testing approach (TDD, Test-after, Selective TDD, No tests), which carries through to implementation. The RED → GREEN → REFACTOR cycle has concrete rules: limit refinement to 3-5 iterations, never modify tests to make them pass, fix the implementation instead. The heuristic: "Can you write `expect(fn(input)).toBe(output)` before writing `fn`? If yes, use TDD."

### Commit Protocol

One task = one logical commit, but TDD tasks produce 2-3 commits (test → feat → optional refactor). Messages follow `type(feature-name): description (FR-01, FR-02)`. Individual file staging only — `git add .` is explicitly forbidden.

### Remediation Loop

When verification finds issues, it creates remediation task files in `design/tasks/` (continuing numbering). These use the same hybrid format as design tasks and are implementable via `/sddw:implement`. The loop: implement remediation → re-verify → until all checks pass. Each remediation task records Severity (FAIL/PARTIAL), Origin (where the issue was introduced), and Evidence (specific failing test name).

### Self-Improving Workflow

The Self-Improve step (`instructions/self-improve.md`) is SDDW's meta-capability: it analyzes the artifact trail (deviations, difficulties, remediation origins, uncovered criteria) and proposes concrete improvements to the workflow files themselves. Proposals include diff previews targeting specific instruction/questionnaire/spec files. Changes are never applied without user approval, even in `--auto`. The workflow evolves with every feature.

## Design Decisions

1. **Specifications as the primary artifact, code as output**: Inverts the standard AI coding loop. Specs are reviewable, version-controlled, persistent. Cited research: detailed specs reduce AI code errors by up to 50% (Piskala, 2026), security defects by 73% (Marri, 2026).
2. **Modular components over monolithic prompts**: Separating commands/instructions/questionnaires/specs enables independent evolution, targeted self-improvement patches, and reuse (e.g. `specs/design-task.md` is shared by implement, taskify, and verify).
3. **Split design and taskify**: Allows architecture review before task breakdown. The combined alias exists for small features. This is the right call for non-trivial work — architecture deserves its own review gate.
4. **Interactive default**: Every spec section confirmed by the user. The agent proposes, the user decides. This makes the specs the user's artifact, not the agent's — crucial for team review and long-lived projects.
5. **File-based artifacts (markdown in `.sddw/`)**: Git-friendly, human-reviewable, no tooling dependency. The cost: no structured querying, traceability relies on grep and FR-ID convention.
6. **Context window as step boundary**: Each step runs in a fresh context (`/clear`). Artifacts (files) are the persistent memory. This is a constraint-turned-feature: models are more accurate in focused contexts.
7. **Self-improvement is post-hoc, not real-time**: The workflow improves after each feature, not during. Trade-off: doesn't prevent the current feature's issues, but avoids disrupting execution with meta-work.
8. **Chat as escape hatch from ceremony**: Not every interaction needs the full spec pipeline. Chat (`instructions/chat.md`) loads artifacts silently and handles questions, spec updates, and quick implementations with minimal ceremony. Guarded by a heuristic: >3 files or >1 commit → redirect to full workflow.

## Comparison Notes

**vs [[Specifications as the Product]]**: SDDW is the operational implementation of that thesis. Where "Specifications as the Product" argues *why*, SDDW provides the *how* — concrete file formats, dialog patterns, and a pipeline that makes specs the durable artifact.

**vs [[Fleet of Agents (sermakarevich)]]**: Same author, different problem. Fleet of Agents is about parallel agent orchestration. SDDW is about serial workflow quality — one feature, one pipeline, careful gates between steps. Complementary: SDDW could feed tasks into a Fleet-style parallel implementation pool.

**vs [[Agent Coding Workflow]]**: SDDW occupies the "compound engineering" end of the maturity spectrum. It's prescriptive where Agent Coding Workflow is descriptive — SDDW tells you exactly what files to write and what dialog pattern to follow.

**vs [[How to Write a Good Spec for Agents]]**: SDDW's requirements/design/taskify specs are a concrete instantiation of good spec-writing principles for agents. The FR-ID traceability chain and explicit prohibitions (SHALL NOT) directly address spec quality.

**vs [[The Plan Is the Program]]**: SDDW embodies this philosophy. The spec files (requirements.md, design.md, task files) ARE the program in a very literal sense — they're the input that drives deterministic code generation.

**vs other Claude Code plugins** ([[Resident (ESP32 Sandbox)]], [[TriadJS]], [[Understand-Anything]]): SDDW is the most architecturally sophisticated Claude Code plugin in the wiki. It's not just a command — it's a complete software development methodology encoded in markdown files with self-improvement capabilities.

**vs [[Compound Engineering]]**: SDDW operationalizes compound engineering's insight that process accumulates. The self-improve step is compound engineering applied to the workflow itself — each feature makes the next one smoother.

**vs Jinzang/Kenny Baas-Schwegler-style spec-first**: SDDW is more pragmatic than formal methods. It uses Given/When/Then acceptance criteria and FR traceability but stops short of formal verification. It's spec-first for the AI age, not spec-first for the formal methods age.

**Limitations**: The plugin is entirely markdown — no programmatic enforcement of rules, no automated traceability beyond grep, no integration with issue trackers or CI. The `/clear` between steps means context is lost; artifacts must carry all state. The self-improve step can only improve workflow text, not workflow structure — it can add a sentence to a questionnaire but can't add a new step to the pipeline.

---

*Sources: [[summary/sddw.md]]*
*Last updated: 2026-06-15*
