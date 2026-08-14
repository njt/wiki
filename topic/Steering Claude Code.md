# Steering Claude Code

Anthropic's definitive taxonomy of the seven mechanisms Claude Code provides for delivering instructions: CLAUDE.md files, rules, skills, subagents, hooks, output styles, and appended system prompts — each with different context-loading behavior, compaction survival, and authority. The post is both reference table and opinionated design guide: it tells you not just what each mechanism does but when you're using the wrong one.

---

The core contribution is a **decision framework for instruction placement**. Every team that uses Claude Code seriously eventually faces the same problem: where do I put this instruction? The answer depends on four variables — when it should load, whether it must survive compaction, how much context it can consume, and whether it needs to be deterministic or probabilistic. The post maps the seven mechanisms onto these variables and provides concrete migration paths for the most common misplacements.

> "Every line loads into every session for every engineer working in the repo, whether it's relevant to their task or not."

This is the CLAUDE.md trap, and it's the same argument Kyle (@0xblacklight) makes in [[Writing a Good CLAUDE.md]]: you have a finite instruction budget, and every line degrades every other line. Anthropic's recommendation — keep it under 200 lines, assign an owner, review like code — operationalizes that budget. The monorepo advice (subdirectory CLAUDE.md files + `claudeMdExcludes`) is the escape hatch for teams that can't fit everything in 200 lines.

> "A real guardrail needs to be deterministic."

The most important sentence in the post. It's the operational insight that [[Guardrails and Feedback Loops]] synthesizes across sources: instructions are probabilistic, constraints are not. A `PreToolUse` hook that exits with code 2 doesn't care how long the session has been running or whether Claude is under context pressure. The post is explicit about enforcement hierarchy: prompt instructions < hooks < managed settings, with managed settings as the only way to enforce organization-wide guardrails that cannot be overridden by local config. This is the same architecture [[Bram]] operationalizes with its hash-verified worklist lifecycle.

> "Instructions that are procedural, like deploy workflows, release checklists, or review processes, belong in a skill rather than in CLAUDE.md."

The CLAUDE.md-as-facts, skills-as-procedures distinction is clean and actionable. CLAUDE.md is the codebase overview — build commands, directory layout, team conventions. Skills are the runbooks. The post notes that on compaction, invoked skills re-inject up to a shared budget with oldest-dropped-first eviction, which means skill authors need to think about what happens when their skill is the one that gets dropped. This is the same territory [[Loop Engineering]] maps: skills and subagents as composable harness components rather than monolithic prompts.

The subagent architecture is the most technically interesting section. Subagents run in isolated context windows and return only their final message plus metadata — "zero cost in main context until called." This is how [[MiMo Code]] and [[DeerFlow]] achieve scale: the orchestration plan and intermediate results live in script variables, not Claude's context window. The five-level nesting depth and dynamic workflow orchestration ("tens to hundreds of background agents") means the subagent system is not a toy delegation mechanism — it's a distributed computing substrate masquerading as a CLI feature.

## Critical Analysis

The post is **strategically silent on the failure modes**. It tells you what each mechanism does but not what happens when you get it wrong. A 30-line procedure in CLAUDE.md bloats context for every session — that's covered. But what about a skill that gets dropped during compaction because the budget was exceeded? What about a subagent that spins for 200 turns on a hallucinated task while the user waits? What about hook ordering when a `PostToolUse` hook on one tool triggers a `PreToolUse` hook on another? These are the real failure modes teams hit, and the post's silence on them is conspicuous.

The **"append-system-prompt" section undersells the caching dynamics**. The post notes that appended prompts are "cached after the first request in a session," but prompt caching in Claude's API has a 5-minute TTL and is per-prefix. Long sessions with many turns will see cache misses at unpredictable intervals. The "moderate" context cost rating is optimistic for anyone not running short, focused sessions.

The **output styles warning is buried**. "Changes to the output style will replace the default output style" unless you set `keep-coding-instructions: true` — this is a footgun that can silently transform Claude Code from a software engineer into a general assistant. The post mentions it but doesn't dwell. Any team that writes a custom output style without reading this warning will spend a confused afternoon wondering why Claude stopped running tests.

The **rules vs. subdirectory CLAUDE.md distinction** could be sharper. The post says to use path-scoped rules for "cross-cutting concerns" and subdirectory CLAUDE.md for "conventions specific to a subdirectory," but in practice these overlap heavily. A Zod validation rule for `src/api/**` could live in `src/api/CLAUDE.md` or in `.claude/rules/` with a `paths:` frontmatter. The real differentiator — which the post hints at but doesn't state — is that rules survive compaction differently (re-injected vs. lost-until-touched). That's the operational difference that should drive the choice.

## Connections

This post is the **reference manual** for the instruction-placement problem that shows up across the wiki:

- [[Claude Code Mastery]] covers the same territory from the practitioner side — Arpan Patel's field manual for the `.claude` directory ecosystem
- [[Writing a Good CLAUDE.md]] provides the instruction-budget argument that explains *why* keeping CLAUDE.md small matters
- [[Guardrails and Feedback Loops]] synthesizes the "deterministic enforcement beats probabilistic instructions" thesis across multiple sources
- [[Bram]] operationalizes `PreToolUse` hooks for hash-verified worklist enforcement — a worked example of the post's most important recommendation
- [[Loop Engineering]] treats skills, subagents, and hooks as composable harness components in a maturity model
- [[A Guide to Claude Code 2.0]] is the other definitive tour of this feature surface, from daily practice
- [[James Montemagno — Copilot Custom Instructions]] shows the same instruction-budget problem in GitHub Copilot's ecosystem
- [[The Clanker Constitution]] is a ready-made payload for the global-system-prompt / CLAUDE.md slot — Kenn's seven-clause default operating principles, with clause 7 taking a concrete stance on the placement question (AGENTS.md as canonical, CLAUDE.md importing it) this page leaves mostly open
- [[Running an AI-Native Engineering Org]] shows how Anthropic's own team uses these mechanisms at scale

The skills system's full frontmatter surface — 20+ fields controlling invocation, execution context, tool pre-approval, and argument passing — is documented in [[Claude Code Skills System]]. The description budget economics (1% of context window, least-used-skills dropped first) add a quantitative dimension to the "when to use which mechanism" decision: a skill with a long description may silently lose its triggering keywords as the budget fills, which is a failure mode the post's taxonomy doesn't capture.

The post also implicitly connects to the broader agent architecture conversation: subagents as isolated context windows are the same pattern as [[The Advisor Strategy]] (specialized models called on demand) and [[MiMo Code]]'s independent writer subagent for memory extraction. And [[The Grid — Agent Identity Architecture|The Grid]]'s identity disks suggest an eighth mechanism beyond the seven Anthropic documents: file-based personality parameters (`context_hunger: 19/20`, `tolerance_for_fake_alignment: 2/20`) that don't tell the agent what to do but who to be — a layer below CLAUDE.md's facts and skills' procedures, shaping *how* the agent approaches any task rather than *what* it does for a specific one. For a worked example of skills encoding complex multi-phase agent pipelines, see [[Cloudflare Security Audit Skill]] — a skill that orchestrates parallel subagents through a six-phase security audit with adversarial validation and structured output enforcement. For a skill suite that applies the same procedural-guardrails-as-skills philosophy across the full SDLC (spec review, plan alignment, architecture analysis, code review, accessibility audit), see [[PAAD — Defense-in-Depth for AI-Assisted Development]].

**The rulebook as a seventh-mechanism pattern:** [[AI Code Migration with Claude Code]] introduces the migration rulebook — a living document that agents both follow *and improve*. When systemic issues surface, you add one sentence to the rulebook and regenerate the affected batch. This is CLAUDE.md-as-compile-target: the rulebook is the durable artifact; the generated code is disposable. It sits at the intersection of three steering mechanisms — it has CLAUDE.md's always-loaded persistence, a skill's procedural authority, and a hook's deterministic enforcement (via the compiler/test suite as referee).

---

*Sources: [[summary/steering-claude-code-skills-hooks-rules-subagents]]*
*Last updated: 2026-07-29*

*See also: [[Teaching the Agent Our Craft]] — Alex Haldeman's 8th Light field report is the most concrete production case study of the full seven-mechanism taxonomy: CLAUDE.md as knowledge root, path-scoped rules for domain conventions, skills for structured RPI workflows, subagents with PreToolUse enforcement, and MCP-wired Linear/Figma. [[New Rules of Context Engineering]] — Thariq Shihipar's companion post validates the taxonomy from the other direction: Anthropic deleted 80% of the system prompt (the same mechanisms this page maps) with no regression on Claude 5 models, confirming that the instruction-budget problem is real and the fix is structural simplification, not better wording.*
