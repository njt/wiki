# The Agentic Product Standard v2.0

The field-tested canonical standard for building production-grade agentic products, distilled from Anthropic, OpenAI, Cognition, Sierra, and LangChain (2024–2026). Not a framework — a specification plus Claude Code skills that operationalize it in your editor. The core thesis: **the model is the variable, the harness is the constant — invest proportionally.**

This is the closest thing the agent ecosystem has to a consensus architecture document. It synthesizes what the leading labs actually do in production (not what their blog posts say) into six principles, five composition patterns, an autonomy ladder, an eight-layer harness model, and a 15-point definition of done. The companion `agent-builder` and `agentic-product-architect` Claude Code skills turn the prose into in-editor behavior — classify your task, route to the right sub-skill, and apply the standard while you work.

## Architecture

The repo is organized as **two representations of one canon** (ADR-0002):

- **`STANDARD.md`** (451 lines) — the product-level prose. Human-first, citable, stable. The source of truth for *what* the standard says.
- **`skills/`** — the operators. Claude Code skill files that turn the canon into in-editor behavior. The source of truth for *how* to act on it.

`CONTEXT.md` holds the shared vocabulary both lean on — "the harness" or "L3" means the same thing whether you read it or an agent applies it.

The skills use a **master-router + sub-skills architecture** (ADR-0001). The `agentic-product-architect` master skill classifies user intent and dispatches to 11 specialized sub-skills (architecture-design, context-engineering, harness-engineering, tool-design-mcp, memory-architecture, tenant-isolation, durable-execution, eval-driven-dev, framework-selection, production-readiness, antipatterns-review). Each sub-skill is self-contained and independently triggerable. This is progressive disclosure — only the needed depth enters context, keeping utilization low (the standard's own 40% rule).

A separate `agent-builder` skill handles the single-agent track, bundling `AGENT_STANDARD.md` (1597 lines) and copy-paste templates for contracts, schemas, traces, and eval fixtures.

## The canonical models

### Autonomy Ladder (L0–L4)
| Level | Description | When |
|-------|-------------|------|
| L0 | Single LLM call | Classification, extraction, summarization |
| L1 | Augmented LLM (+ retrieval, tools, memory) | Q&A over docs, simple assistants |
| L2 | Workflow (deterministic code orchestrates LLM steps) | Path is known; predictability matters |
| L3 | Orchestrator-Worker (LLM decomposes within bounded graph) | Parallelizable, breadth-first work |
| L4 | Autonomous Agent Loop (LLM chooses next step) | Path cannot be enumerated; cost acceptable |

**Escalation rule**: do not climb to L+1 until L delivers ≥90% pass rate on curated evals. Most production "agents" are L2 + targeted L3. L4 is reserved for narrow phases.

### Five Composition Patterns
1. **Prompt Chaining** — sequential (outline → draft → polish)
2. **Routing** — classifier dispatches to specialist
3. **Parallelization** — fan-out + aggregation
4. **Orchestrator-Workers** — central planner + dynamic workers
5. **Evaluator-Optimizer** — generator + critic loop until acceptance

**Meta-principle**: compose these in deterministic code first. A full agent loop is the *last* resort.

### Eight-Layer Harness
```
8. Security & Identity (CROSS-CUTTING) ← threat model, injection, identity, least privilege, pinned tools
7. Observability & Tracing           ← log EVERYTHING
6. Evaluation Layer (CI gates)       ← block regressions
5. Human-in-the-Loop                 ← approval gates
4. Guardrails (input/output)         ← defense in depth
3. Durable Execution                 ← pause/resume/retry
2. Context & Memory Management       ← write/select/compress/isolate
1. Agent Loop (gather→act→verify)    ← the "agent" proper
```

Security (Layer 8) is cross-cutting — it constrains every layer beneath it. A guardrail is one tactic inside Security, not a substitute for it. Content filters top out near ~97% accuracy, so ~3% of injection attacks succeed *by design* — you mitigate that structurally, not by tuning a filter.

## Key techniques

**The 40% rule.** Keep context-window utilization below ~40% of the model's limit. Degradation past that point is non-linear — backed by Chroma's "context rot" research. Bigger windows don't repeal the rule. The frontier technique is just-in-time retrieval (Claude Code's glob+grep+read pattern) over precomputed vector RAG.

**Bitter-pilled maintenance.** Tag every rule as anti-fragile (keep: eval sets, data pipelines, tool contracts) or fragile (cut/re-test: chain-of-thought orchestrators, output-format parsers, retry cascades). Test: "Would a smarter model make this rule unnecessary?" If yes, it's scaffolding — remove it. The harness should shrink as models improve.

**Closed enumerations over open vocabularies.** For any rule the model has shown willingness to satisfy cosmetically (selecting a category, naming a capability), inline the *complete allowed set* in the runtime context. A pointer to another file leaks the vocabulary under pressure — the model fills the gap with plausible-but-invented values.

**Derived anti-criteria.** Every forbidden action in the agent contract must yield at least one code-asserted test that fails if the forbidden thing happens. Prose forbiddance is not enforcement. "Do not send email" without `expect(trace.events.filter(e => e.type === "email.send")).toHaveLength(0)` is wishful thinking.

**Hard-to-vary acceptance criteria.** A criterion is well-formed only if you can name the single probe (Read/Grep/Bash/curl/SELECT/test run) that returns yes/no. If you cannot name the falsifying test, it is not yet a criterion — it is a wish.

**Conjecture/refutation learning trail.** Each production failure regression carries four fields: the belief that turned out wrong, the trace that broke it, the corrected understanding, and the new assertion added. An entry missing any field is a note, not a regression.

**pass^k reliability metric.** Track `pass^k` (does it succeed on ALL k attempts), not just `pass@1`. A right answer reached by a reckless path is a latent incident.

**The eval pyramid.** Level 1: code assertions (every change, cheap). Level 2: LLM-as-judge (on cadence, **binary only**, calibrated against ≥100 human labels, TPR/TNR tracked every release). Level 3: human review (~20-50 traces on major changes). The key rule: Likert scales break alignment with human raters — binary only.

**Permission tiers (P0–P6).** Read (P0, no approval) → Draft (P1) → Internal Write (P2) → External Write (P3, approval required) → Financial (P4) → Communication (P5) → Destructive (P6, always approval). Permissions enforced in code, never by prompt.

**MCP supply-chain controls.** Community MCP servers are untrusted supply chain. Pin tool definitions by cryptographic hash; alert on any change. Install only from an allow-listed registry, version-pinned and signature-checked. OAuth 2.1 + Resource Indicators; never pass tokens through.

**The lethal trifecta.** An agent with (1) private data access, (2) untrusted content exposure, AND (3) external communication ability becomes an exfiltration tool via prompt injection. Run this structural check on every deployment; if all three present, break one leg before shipping.

## Design decisions

**Prose + operators split.** Two audiences, one canon. STANDARD.md is optimized for human reading and citation; skills/ are optimized for agent execution. The cost: they must not drift — canon changes update both representations in the same commit.

**Master-router over monolithic skill.** Progressive disclosure keeps context low. The cost: routing quality is a maintenance obligation — a stale skill description sends requests to the wrong sub-skill.

**Stable canons, churning vendors.** The architectural canons (autonomy ladder, 5 patterns, single-vs-multi, harness) are deliberately stable. Framework rankings and vendor specifics are expected to churn — those PRs are the easy yes. This is explicit in GOVERNANCE.md.

**Orchestrator-subagent consensus.** The 2026 field has settled on a single lead spawning isolated subagents and consuming their summaries — not peer-to-peer agent buses or free-form agent "debates." Anthropic's research system, Claude Code's Task tool, and Cognition's 2026 follow-up converge here. Peer-to-peer multiplies context, compounds errors, and resists evaluation.

**Binary maturity scorecard.** The SCORECARD.md (M0–M3) is deliberately binary — a half-met control is a No. Your level is the highest band where every gate item is satisfied. One unmet gate caps you at the level below. No partial credit, no skipping.

**Security as v2.0's headline.** Principle 6, Layer 8, and the red-team kit (`templates/security/`) were all added in v2.0 (June 2026). The argument: content filters top out at ~97%, so structural controls (identity, least privilege, isolation, pinned tools) are the discipline — guardrails are one tactic inside it, not the whole thing.

**Cost as engineering constraint.** v2.0 also added Layer 9 (Cost & FinOps): per-run token ceilings enforced in code, the ~15× multi-agent economics rule, and the FinOps Foundation's finding that 98% of orgs now manage AI spend. A runaway autonomous session without a circuit breaker is an unbounded invoice.

## Comparison notes

Unlike [[Agent-Native Architectures (Every)]] which focuses on principles for designing products where agents are first-class users, this standard focuses on *building* the agents themselves — the harness, the eval discipline, the security model.

Unlike [[Components of a Coding Agent]] which provides a taxonomy of agent components, this is prescriptive: it says "do this, don't do that, here's the exact contract format."

Unlike [[Elements of Agentic Systems Design]] which is a descriptive taxonomy, this is a normative standard backed by runnable skills that enforce it during development.

Unlike [[Elysia]] (Weaviate's decision-tree framework) which constrains tool choice per node, this standard advocates MCP by default with <20 active tools and RAG-MCP for larger catalogs — a different trade-off: protocol standardization over framework-level constraint.

The [[Golem Covenant]] v0.1 spec (five-organ taxonomy, default-deny, tested return-to-dust) is a narrower security framework for agent boundaries. This standard covers security (Principle 6, Layer 8) plus the full architecture, eval, and operations stack.

The 40% rule and context engineering framework (write/select/compress/isolate) align with [[Agent Memory and Context]] and [[How Hightouch Built Their Long-Running Agent Harness]], but this standard is more prescriptive — it names the exact operations and the exact threshold.

The "determinism by default" principle echoes [[Smart Models Dumb Pipes]] — LLMs as judgment machines, not Q&A machines. The harness is the product.

The [[Load-Bearing Assumptions]] skill (surfacing falsifiable claims in code plans) is the kind of Level 1 eval this standard would mandate — code assertions on every change.

## Tags

#standard #project #agents #architecture #harness #evals #context-engineering #security #MCP #specification #claude-code-skills

## Related pages

- [[Agent Coding Workflow]] — the practitioner's daily loop this standard informs
- [[Agent Orchestration]] — multi-agent coordination patterns (this standard says: orchestrator-subagent)
- [[Agent Memory and Context]] — the context engineering discipline this standard centers
- [[Guardrails and Feedback Loops]] — deterministic enforcement, not prompt instructions
- [[Security and Sandboxing]] — the structural security this standard's Principle 6 demands
- [[Components of a Coding Agent]] — the harness taxonomy this standard operationalizes
- [[Specifications as the Product]] — the standard-as-durable-artifact philosophy
- [[Elements of Agentic Systems Design]] — the descriptive taxonomy this standard makes prescriptive
- [[Agent-Native Architectures (Every)]] — complementary design principles
- [[Golem Covenant]] — narrower agent-boundary security spec
- [[Smart Models Dumb Pipes]] — shared philosophy: harness over model
- [[Load-Bearing Assumptions]] — the kind of code assertion this standard would mandate
- [[Building Agents for Production Systems with MCP]] — MCP as the standard integration layer
- [[10 Principles for Agent-Native CLIs]] — Trevin Chow's agent-first design rules
- [[Moltbook]] — the lethal trifecta in production

---
*Source: [github.com/Moai-Team-LLC/agentic-product-standard](https://github.com/Moai-Team-LLC/agentic-product-standard) v2.0.0, ingested 2026-06-09.*
