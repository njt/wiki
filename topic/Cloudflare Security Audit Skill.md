# Cloudflare Security Audit Skill

Cloudflare's open-source Claude Code skill that turns a coding agent into a security auditor. It orchestrates multiple parallel agents through a six-phase pipeline — recon, hunting, adversarial validation, reporting, structured output, and independent verification — to find exploitable vulnerabilities with real impact. This skill seeded Cloudflare's internal fleet-wide vulnerability discovery harness, and it's the most detailed public example of prompt-engineered security methodology for coding agents.

---

## Architecture

The skill is a flat collection of 11 markdown files (1,269 lines total) plus a JSON schema and a zero-dependency Node.js validator. It has no build system, no dependencies beyond Node.js for the validator, and is installed via `npx skills add`.

### Six-phase pipeline

**Phase 1: Recon** (`RECONNAISSANCE.md`) — Three parallel `research` agents map the codebase from different angles (overview/tech-stack/baseline, trust boundaries/access control, input surface inventory). Their outputs are synthesized into `architecture.md`, which is injected verbatim into every Phase 2 agent prompt. The number of Phase 2 agents is determined by what Phase 1 discovers — a small library gets 3–4, a complex application gets 8–12+ split by both attack class and subsystem.

**Phase 2: Hunt** (`HUNTING.md`, `ATTACK-CLASSES.md`, plus 4 domain companion files) — Multiple parallel `general` agents attack the codebase from different angles. Each agent gets a specific attack class, a subsystem scope, a 12-angle hunting heuristic, and 6 validation rules. General agents can spawn their own `research` sub-agents to dig deeper into rabbit holes without blowing their own context window.

The attack classes span 9 core categories (injection, access control, resource handling, cryptography, business logic, feature abuse, chained attacks, wildcard, obvious things) plus 4 specialized domains: memory-safety/binary (`MEMORY-SAFETY-AND-BINARY.md`), AI/LLM (`AI-AND-LLM.md`), HTTP-protocol/auth (`WEB-PROTOCOL-AND-AUTH.md`), and client-side/browser (`CLIENT-SIDE.md`). Each domain companion includes domain-specific core discipline rules and validation rules on top of the shared base.

**Phase 3: Validate** (`VALIDATION-AND-REPORTING.md`) — Findings are consolidated (Phase 2 deliberately overlaps scopes so duplicates are expected), then separate `research` agents try to *disprove* each one using five adversarial tests (exploitation, impact, baseline, mitigation, parser/runtime behavior). The agent finding a bug never validates it.

**Phase 4: Report** — Produces `REPORT.md` (executive summary, findings table, hardening notes, positive patterns) and `FINDINGS-DETAIL.md` (complete data flows for MEDIUM+ findings).

**Phase 5: Structured output** — Every finding becomes a JSON object conforming to `report-schema.json`. The schema supports two verdicts via `oneOf`: `confirmed` (requiring a trace array with entrypoint→propagation→sink steps, execution details with concrete payloads, two-axis severity scoring, confidence score, root cause template, intended behavior, and remediation) and `rejected` (requiring a reason citing the factual error). The schema enforces `additionalProperties: false` — extra fields are a hard failure. The 201-line `validate-findings.cjs` implements a JSON Schema subset interpreter plus a semantic layer that enforces trace structure (first=entrypoint, last=sink, middle=propagation).

**Phase 6: Independent verification** — Fresh `research` agents, one per confirmed finding, independently verify every factual claim in `findings.json` against actual source code. They check file paths, line numbers, function names, trace descriptions, root cause, execution payloads, conditions, remediation, and confidence scores. Returns VERIFIED, CORRECTED (with specific field corrections), or REJECTED. The human-readable report is reconciled to the verified JSON — the two must not disagree.

### Core architectural invariants

- **Adversarial separation**: The agent that validates a finding never found it; the agent that verifies a finding never wrote its JSON. Three independent perspectives per confirmed bug.
- **Incremental coverage**: Multiple runs are additive. Testing shows a single run finds roughly half the total vulnerabilities. The skill reads prior `findings.json` to skip known issues and target gaps.
- **Dynamic baseline calibration**: Rather than hardcoding "compare to WordPress," Phase 1 discovers a meaningful comparable dynamically. If the same pattern has been exploited in the comparable, the finding is stronger; if never exploited in 20 years, understand why before reporting.
- **Source-first, confirm dynamically**: Source code is primary, but a reproduced result beats a reasoned argument. Where the target is locally buildable, extract suspect code into a minimal harness. Where confirmation needs infrastructure you don't have (proxy chain, live cache), mark "requires deployment testing" — don't report as confirmed.

---

## Key techniques

### The 12-angle hunting heuristic

Rather than generic "look for bugs," every Phase 2 agent gets a specific methodology:
1. Attack the **sad path** — error handlers, fallback branches, catch blocks, timeout paths
2. Test **boundaries** — empty, max-length, null vs undefined, zero, negative, Unicode edges
3. Find **implicit trust** between components — does the DB layer assume the API validated?
4. Reorder **operations** — call step 3 before step 1, replay completed flows
5. Race **concurrent** operations — two requests to same resource, modify while reading
6. Exploit **parser differentials** — input accepted by schema but rejected by DB, URL parsed differently by router vs app
7. Test **round-trip survival** — does escaping double-up? Is a relative path resolved differently on read vs write?
8. Abuse **configuration** — what happens when config is missing? Can an env var override a security control?
9. **Trace privilege** — for every state change, find the permission check and verify it's the right check on the right resource
10. Find **leaked context** — error messages revealing paths, timing differences, debug endpoints
11. Find **security-relevant parameter overrides** — safe defaults that user input can change
12. Find **unverified claims driving trust** — self-declared identity influencing access decisions

### Adversarial validation as false-positive filter

The skill's most distinctive technique is its three-stage adversarial pipeline: hunters report (biased toward finding), validators try to disprove (biased against), verifiers independently re-check. This mirrors a human security team's structure but executes it programmatically. Each agent that touches a finding has a different prompt and bias — the hunter's "find bugs," the validator's "your job is to DISPROVE this finding," the verifier's "you did NOT write this finding, verify every factual claim." False positives must survive two independent agents trying to kill them.

### Domain-specific attack class companions

Rather than forcing web-oriented vulnerability classes onto every target, the skill has four companion files that completely replace or augment the core attack classes based on target type:

- **Memory-safety/binary**: Does not mention injection or access control. Instead: length subtraction underflow, operator-precedence errors in size computations, sizeof pointer-depth confusion, wire-length into fixed stack buffer, UAF via embedded waiter-anchor, type confusion, uninitialized-buffer read oracle. The validation rules require De Bruijn pattern crash analysis, reclaim-and-compare UAF proof, and distinguishing crash from exploitable.

- **AI/LLM**: Rejects "the model can be prompt-injected" as a standalone finding — requires the injection to cross a boundary (victim's context, privileged capability, downstream sink). Key concept: "the model is a confused deputy that will faithfully carry attacker instructions across a trust boundary the developer assumed the model would respect." Distinguishes between cross-session injection (high-value) and same-session self-injection (not a finding unless the model has capabilities the user lacks).

- **HTTP-protocol/auth**: Core insight: "framing bugs live in disagreement, not in one parser." Smuggling, cache poisoning, and desync exist because two components interpret the same bytes differently. Requires naming both components and the divergent parse. Critical constraint: these bugs frequently depend on components not in the audited tree — must mark unverifiable, not downgrade.

- **Client-side**: Core insight: "server escaping ends where the fragment begins." Data after `#`, `window.name`, and `postMessage` payloads never reach the server, so server-side filters can't see them. In auto-escaping frameworks, the candidate list *is* every `dangerouslySetInnerHTML`/`v-html`/`bypassSecurityTrust*` call — start there.

### Structured output as rigor enforcement

The JSON schema doesn't just define a format — it forces verification. A finding requires exact file paths, line numbers, function names, and trace descriptions. The `root_cause` field uses a template: "[function] in [file] does not [missing action], allowing [consequence]." The severity system requires separate likelihood and impact scores, each with a reason. The `execution` object requires attacker perspective, concrete payloads, step-by-step instructions, and an expected observable result. If the agent can't fill every field from source code evidence, the finding gets rejected rather than soft-pedaled.

---

## Design decisions

### Optimized for precision over recall

The skill explicitly trades recall for precision. "Only report what you can exploit" is the first principle. The adversarial architecture, the structured output requirements, the dynamic confirmation preference, the "kill false positives aggressively" instruction — all push toward fewer, verified findings. This is the right trade-off for a tool whose output humans will act on: three MEDIUMs you can fix beat ten LOWs that waste time. But it means the skill is unsuitable for broad vulnerability surface scanning; for that, pair it with a SAST tool and use the skill for deep-dive verification.

### Context budgeting through agent decomposition

Rather than giving one agent everything (which overflows context), the skill decomposes the audit: Phase 1 by dimension (overview, trust, surfaces), Phase 2 by attack-class × subsystem (each agent sees only its scope), Phase 3+ by finding (one agent per finding or batch). The choice of `general` over `research` for hunters is because general agents can spawn sub-agents — when a hunter finds a rabbit hole, it delegates rather than bloating its own context. This is explicit context budgeting.

### Agent-neutral by design

The SKILL.md includes explicit platform terminology mapping (Task tool, research agent, general agent, subagent_type) to make the methodology portable across coding agents. In practice, the skill requires parallel sub-agent spawning, file system access for reading source, and structured output — capabilities that Claude Code provides natively but that limit portability to other platforms.

### Weakness: no persistent learning

Unlike the internal Cloudflare vulnerability harness this skill seeded (described as "multi-stage, fleet-wide"), the skill has no cross-repo learning, no attack-class-to-vulnerability mapping that improves over time, no deduplication database beyond reading prior `findings.json`. Each audit is independent. The skill is the starting point, not the production system.

### Weakness: model quality is the ceiling

The adversarial validation can catch factual errors (wrong file paths, incorrect trace steps) but is limited in catching the model's systematic blind spots — if the model consistently misreads a certain code pattern or has a known false-positive bias for a particular class, the validators (using the same model family) may share those blind spots.

---

## Comparison notes

Unlike [[Metis — ARM AI Security Code Review]], which combines tree-sitter call-graph reachability analysis with LLM confirmation (deterministic + probabilistic hybrid), the security-audit skill is pure prompt-engineered agent orchestration — no static analysis, no deterministic reachability. The trade-off: Metis can handle 19+ languages with consistent call-graph precision; the security-audit skill can find logic bugs and chained attacks that static analysis misses, but is language-agnostic only to the extent the underlying model reads code well.

Unlike [[Orchestrating AI Code Review at Scale]] (Cloudflare's production code review system with 131K reviews, tiered models, and circuit breakers), the security-audit skill is a single-repo tool without production infrastructure — no tiered model routing, no CI/CD integration, no persistent state. The production review system and the security audit skill share the same DNA (parallel agents, adversarial validation, Cloudflare authorship) but target different problems (code review correctness vs security vulnerability discovery).

Unlike traditional SAST tools (Semgrep, CodeQL), the security-audit skill finds what rule-based scanners can't: business logic errors, chained attacks, trust boundary violations, context-dependent design flaws. The trade-off is cost per line (LLM tokens vs CPU cycles) and determinism (different runs yield different findings).

Unique among open-source security tools, the skill's adversarial architecture — find → try to disprove → independently verify → structured output enforcement — is a methodology contribution independent of the prompt content. The same pipeline structure could be applied to other audit domains (compliance, performance, accessibility) by swapping the attack class prompts for domain-specific investigation prompts.

---
*Sources: [[raw/cloudflare-security-audit-skill]]*
*Last updated: 2026-07-18*
