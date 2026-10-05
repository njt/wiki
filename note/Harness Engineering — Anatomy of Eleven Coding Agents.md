# Harness Engineering: Anatomy of Eleven Coding Agents

The most comprehensive empirical reference for harness engineering to date: a July-2026 source-code dissection of eleven production coding-agent harnesses (plus Databricks's Omnigent meta-harness as a contrast point), organized around seven canonical subsystems, yielding 13 cross-cutting observations, 29 design patterns, a ninety-day longitudinal diff of the same systems, and a platform-turn thesis — the harness stopped being a tool and became a platform.

---

An agent is a model plus a harness. This paper names the discipline (coined February 2026, ironically from within LangChain) and then does the unglamorous work nobody else has: it greps the actual trees. Eleven systems — Claude Code, Codex CLI, Gemini CLI, Mistral Vibe, OpenHands, Aider, Mini-SWE-Agent, Hermes, Pi, OpenCode, OpenClaw — pinned to July 2026 releases, dissected across seven subsystems: agent loop, LLM integration, tools & actions, memory & context, safety & permissions, orchestration, and extensibility. Because eight of the systems were re-pinned from an April edition, the study also contains a controlled ninety-day source diff — longitudinal evidence in a field where almost nobody measures.

## The twin absences

The paper's most quotable empirical finding, and the one it verified hardest (a weeks-long search for counterexamples, then a three-month re-audit):

> Across roughly 4M lines of Python, TypeScript, and Rust, no production agent code path imports any of them [LangChain, LangGraph, AutoGen, CrewAI, ...]. Every loop is hand-rolled in the host language's native primitives... Every prompt template is plain Markdown, Jinja2, or string concatenation.

And the second absence:

> No system uses vector-embedding RAG for code retrieval; all rely on ripgrep, tree-sitter, glob, and auto-discovered Markdown context files.

This is a direct empirical confirmation of [[Three Kinds of Agentic Search]] and a knockout blow to the assumption baked into most "AI stack" advice. The reasons are domain-specific and well argued: code carries dense deterministic structure (paths, tree-sitter parses, LSP types) that similarity search can't replicate; code changes minute-to-minute so embeddings are stale almost by construction; and every coding environment already ships a near-optimal retrieval system called ripgrep. It also independently corroborates [[Rip Vector Database]] — "just-in-time" retrieval with deterministic tools beat pre-indexed embeddings in every production code path audited. When even Gemini CLI declines to use Google's own Genkit and ADK, the absence is a verdict, not an oversight.

## The seven subsystems and the floor

Every harness, from Mini-SWE-Agent's ~100 lines to Codex's 1.12M lines of Rust, must answer the same seven questions. The floor is astonishingly low: Mini-SWE-Agent reports SWE-bench numbers in the same range as systems three orders of magnitude larger. What separates floor from production is not task completion — it's safety, recovery, cost management, and extensibility. The paper's 29-pattern catalog includes genuinely novel documentation: agent-maintained memory pipelines, verify-on-stop guards, lineage compaction, session-tree version control, log-as-queue loops, syntax-aware command permissioning, cache-dialect fanout, and harness mimicry (Pi presenting Claude Code's identity to ride its OAuth backend — the first source-verified instance I've seen named).

## The platform turn

The thesis, stated with named artifacts rather than vibes:

> The coding-agent harness completed its platform turn in the first half of 2026... The competitive unit of the field is no longer the agent loop; it is the ecosystem surface around it.

Four converging signals: skills as declarative programs in natural language (9/11 systems); hooks and event buses as the dominant extension substrate; the agent loop absorbing the workflow-engine role; and harnesses shipping platform interfaces (OpenCode's embedded HTTP server, OpenHands's OpenAI-compatible gateway — "the agent as a model"). Meanwhile the harness–framework merger runs both ways: Claude Agent SDK and openai-codex SDK make harnesses importable, while LangChain's Deep Agents and Pydantic AI's harness make frameworks into harnesses — converging on the same shape from opposite directions. Codex ships an importer for Claude Code's on-disk state; vendors writing importers for each other's session stores is exactly the stage of platform competition where user data becomes the moat.

## Ninety days of evolution

The longitudinal diff is the paper's most methodologically interesting section. Convergence became *traceable imitation*: Codex adopted Claude Code's hook event vocabulary verbatim; OpenHands adopted its plugin manifest format. Patterns diffused fast — deferred tool loading went from one system to three, turn-level checkpointing from one to three. The sharpest observation: behavioral policy is migrating from prompt prose to configuration (Codex dropping no-commit rules from prompts in favor of feature flags; Mistral Vibe deleting its "Never Commit" hard rule and A/B-testing prompts server-side). Policy moves from where the model reads it to where the platform enforces it. And the methodological lesson: *inventory* claims (tool counts, feature cells) decay in weeks, while *structural* claims (loop taxonomy, the absences) have proven durable — a distinction the paper disciplined itself to observe.

## Critical take

This is what the field has needed and mostly not gotten: claims pinned to code, repeated under audit, with corrections published in place (three April observations were revised). It's honest about being descriptive rather than benchmarking, and caveats self-reported figures. Weaknesses worth noting: the twin-absence result is partly a tautology of its corpus selection — these are provider-native CLI products, which would never import a rival framework; a corpus of in-house enterprise agents might show more framework use. And "harness engineering" as a named discipline is largely a rebrand of what every agent builder was already doing — though the paper is refreshingly aware of the term's vendor-blog provenance and mines the irony (LangChain named the discipline whose frameworks its subjects uniformly refuse to use). The platform-turn thesis is the boldest claim and the best-evidenced; read alongside [[The Harness Is the Company]] it completes the argument that the moat is the runtime, not the model.

## Related pages

- Strengthens [[The Harness Is the Company]] with source-code evidence for the harness-as-platform thesis: importers, marketplaces, MDM governance, and acquisition-scale pricing are the mechanisms behind that page's argument.
- Corroborates [[Three Kinds of Agentic Search]] empirically at eleven-system scale — zero vector-embedding RAG over code across ~4M lines, deterministic JIT retrieval everywhere.
- Nuances [[Introducing Omnigent]] by auditing it as a contrast point: what the meta-harness deliberately leaves below the line, and why OS-level sandboxing resists being factored out.
- Grounds [[Building an Advanced Agentic Harness]]-style practitioner advice in a 29-pattern catalog and 18 pinned design recommendations, plus a 90-line minimum-viable-harness scaffold.

---
*Sources: [[raw/2609-00006v1]], [[summary/2609-00006v1]]*
*Last updated: 2026-10-05*
