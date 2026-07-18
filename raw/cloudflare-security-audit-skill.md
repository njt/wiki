---
url: https://github.com/cloudflare/security-audit-skill
date_fetched: 2026-07-18
title: Cloudflare Security Audit Skill
author: Cloudflare
date_published: 2025
---

# Cloudflare Security Audit Skill

A Claude Code skill that turns a coding agent into a security auditor. It orchestrates multiple parallel agents through a six-phase pipeline — recon, hunting, validation, reporting, structured output, and independent verification — to find exploitable vulnerabilities with real impact. This skill seeded Cloudflare's internal vulnerability discovery harness, described in their blog post "Build your own vulnerability harness."

## Architecture

The project is not a traditional application but a **prompt-engineering system** — a set of markdown files constituting a Claude Code skill. The skill is distributed as a flat file collection (no build system, no dependencies beyond Node.js for the validator), designed to be installed via `npx skills add`.

### File inventory (1,269 lines total)

| File | Lines | Purpose |
|------|-------|---------|
| `README.md` | 92 | Overview, installation, design principles |
| `SKILL.md` | 108 | Core prompt: platform terminology, setup, principles, anti-patterns |
| `RECONNAISSANCE.md` | 46 | Phase 1: parallel research agent prompts for architecture mapping |
| `HUNTING.md` | 110 | Phase 2: agent orchestration, hunting methodology, validation rules |
| `ATTACK-CLASSES.md` | 116 | Core attack class prompts (injection, access control, business logic, etc.) |
| `MEMORY-SAFETY-AND-BINARY.md` | 58 | Specialized prompts for native/binary/kernel targets |
| `AI-AND-LLM.md` | 67 | Specialized prompts for LLM-backed targets (prompt injection, confused deputy) |
| `WEB-PROTOCOL-AND-AUTH.md` | 85 | Specialized prompts for HTTP-protocol and authentication targets |
| `CLIENT-SIDE.md` | 67 | Specialized prompts for browser/SPA targets (DOM XSS, postMessage, prototype pollution) |
| `VALIDATION-AND-REPORTING.md` | 109 | Phases 3–6: validation, reporting, structured output, independent verification |
| `report-schema.json` | 210 | JSON Schema for findings.json (confirmed and rejected verdicts) |
| `validate-findings.cjs` | 201 | Zero-dependency Node.js validator implementing a JSON Schema subset interpreter |

### Six-phase pipeline

1. **Recon** (RECONNAISSANCE.md): Launches three parallel `research` agents to map architecture, trust boundaries, and input surfaces. Synthesizes output into `architecture.md`, which is injected verbatim into every Phase 2 agent prompt.

2. **Hunt** (HUNTING.md + ATTACK-CLASSES.md + 4 domain companions): Launches multiple parallel `general` agents, each assigned a specific attack class and subsystem. The number of agents scales with codebase complexity — 3–4 for a small library, 8–12+ for large applications split by both attack class and subsystem. Each agent gets the architecture summary, hunting methodology (12 heuristic angles), and validation rules copied verbatim. General agents can spawn their own `research` sub-agents to dig deeper.

3. **Validate** (VALIDATION-AND-REPORTING.md): Consolidates duplicates first (Phase 2 deliberately overlaps scopes), then launches separate `research` agents that try to *disprove* each finding. Five adversarial tests: exploitation, impact, baseline, mitigation, and parser/runtime behavior.

4. **Report** (VALIDATION-AND-REPORTING.md): Produces `REPORT.md` (executive summary, findings table, hardening notes, positive patterns) and `FINDINGS-DETAIL.md` (complete data flows for MEDIUM+ findings).

5. **Structured output** (VALIDATION-AND-REPORTING.md + report-schema.json + validate-findings.cjs): Every finding becomes a JSON object conforming to a strict schema. The schema supports two verdicts via `oneOf`: `confirmed` (full trace with entrypoint→propagation→sink chain, execution payloads, remediation, severity with likelihood/impact axes, confidence score) and `rejected` (reason for factual error). The validator (`validate-findings.cjs`) implements a subset of JSON Schema interpretation in 201 lines: type checking, required fields, enum validation, const matching, additionalProperties enforcement, and a oneOf discriminator. It adds a semantic layer for trace structure (first step must be `entrypoint`, last must be `sink`, intermediates must be `propagation`).

6. **Independent verification** (VALIDATION-AND-REPORTING.md): Fresh `research` agents, one per confirmed finding, independently verify every factual claim in `findings.json` against actual source code. They check file paths, line numbers, function names, trace descriptions, root cause statements, execution payloads, conditions, remediation, and confidence scores. Returns VERIFIED, CORRECTED, or REJECTED.

### Key architectural invariants

- **Adversarial separation**: The agent that checks a finding is never the agent that found it (Phase 3 hunters vs Phase 2 hunters; Phase 6 verifiers vs Phase 5 writers).
- **Incremental coverage**: Multiple runs against the same repo are additive. Each run reads prior `findings.json` to skip known issues and target gaps left by previous runs. Testing shows a single run finds roughly half the total vulnerabilities across multiple runs.
- **Dynamic baseline calibration**: Rather than hardcoding a comparable (e.g., "compare to WordPress"), Phase 1 agents identify the application type and find a meaningful comparable dynamically. The comparable calibrates effort — if the same pattern has been exploited there, the finding is stronger; if never exploited in 20 years, understand why before reporting.
- **Source-first, confirm dynamically**: This is a source audit, but claims you can execute beat ones you can only argue. Where the target is locally buildable, reproduce the crash, run the payload, or extract suspect code into a minimal harness. Where confirmation needs infrastructure you don't have (proxy chain, live cache, production auth), mark it "requires deployment testing" — don't report as confirmed.

## Methodology innovations

### The 12-angle hunting heuristic (HUNTING.md)

Rather than generic "look for bugs" instructions, every Phase 2 agent receives a specific 12-angle methodology:
1. Attack the sad path (error handlers, fallback branches, catch blocks, timeout paths)
2. Boundary conditions (empty, max-length, null vs undefined, zero, negative, Unicode)
3. Implicit trust between components (does the DB layer assume the API validated?)
4. Operation ordering (call step 3 before step 1, replay completed flows)
5. Concurrency (two requests to same resource, modify while reading, delete while iterating)
6. Parser differentials (input accepted by schema but rejected by database, URL parsed differently by router vs app)
7. Round-trip survival (data stored then retrieved — is it the same? Does escaping double-up?)
8. Configuration control (what happens when config is missing? Can an env var override a security control?)
9. Privilege tracing ("follow the money" — for every state change, trace back to the permission check)
10. Leaked context (error messages revealing paths, timing differences, debug endpoints)
11. Security-relevant parameter overrides (safe defaults that user input can change)
12. Unverified claims driving trust decisions (self-declared identity influencing access)

### Domain-specific hunting classes

The core ATTACK-CLASSES.md covers 9 classes for general web/app targets:

- **Injection**: Direct and indirect (stored → retrieved → dangerous context), injection through field names/keys/headers/metadata, injection into secondary systems (logs, caches, search indexes)
- **Access control**: Beyond "does the check exist?" — checks the right permission for the right resource via the right mechanism, bulk operations enforcing per-item permissions, multiple access paths with inconsistent checks
- **Resource and file handling**: Path traversal (including symlinks, encoded sequences, null bytes), SSRF (including redirects, DNS rebinding, URL parser differentials), TOCTOU races
- **Cryptography and secrets**: Weak randomness, hardcoded secrets, missing HMAC verification, nonce reuse, timing side-channels, what happens when crypto *fails*
- **Business logic**: State machine violations, race conditions with business impact, numeric/quantity manipulation, implicit trust assumptions, time-based logic, default/fallback behavior
- **Feature abuse and data leakage**: Export as exfiltration, import as injection, search as oracle, enumeration through side effects, preview/draft/staging leakage, notification/webhook as SSRF
- **Chained attacks**: Multi-step chains (info disclosure + IDOR + missing rate limit), cross-component trust gaps, second-order attacks, scope/capability escalation, timing/ordering gaps, rollback/recovery abuse
- **Wildcard**: No assigned category — finds what nobody thought to look for. Investigates half-finished features, hidden endpoints, git history, sabotage vectors, environmental assumptions, and the gaps in test coverage
- **Obvious things**: Thorough, literal check of hardcoded secrets, TODO/FIXME comments referencing security, debug mode gating, unprotected admin endpoints, `.env` files in repo, missing cookie attributes, open redirects

Four companion files extend coverage for specialized targets:

- **MEMORY-SAFETY-AND-BINARY.md**: For C/C++/Rust-unsafe, kernel modules, parsers, firmware. Classes: spatial OOB (length subtraction underflow, operator-precedence errors, sizeof pointer-depth confusion, wire-length into fixed stack buffer), temporal UAF (embedded waiter-anchor freed without draining, cached raw pointer + reallocating owner), type confusion (read-and-write confusion, hierarchical-walker leaf check skipped), value bugs (uninitialized buffer + observable compare = read oracle). Kernel-specific: user-copy bounds + double-fetch, object lifecycle/retain-release, unchecked downcast, world-writable powerful interfaces, validate-then-act-on-stale-state. Requires debuggable target, De Bruijn pattern crash analysis, reclaim-and-compare UAF proof, and distinguishing crash from exploitable.
- **AI-AND-LLM.md**: For chatbots, RAG pipelines, tool-calling agents, MCP servers/clients. Key principle: "the model can be prompt-injected" is not a finding — the injection must cross a boundary. Classes: indirect injection via retrieved/ingested content (the high-value class), tool-argument injection (model output → sink), direct injection into privileged capability, prompt-template/delimiter injection, excessive agency/confused deputy, unbounded action loops/cost abuse, sub-agent/MCP trust inheritance, insecure output rendering (XSS via model output, image-link exfiltration), system-prompt extraction to a real secret, cross-session context bleed. Validation requires naming the boundary crossed, proving the trusting code path, and not asserting capabilities the source doesn't show.
- **WEB-PROTOCOL-AND-AUTH.md**: For reverse proxies, CDNs, API gateways, HTTP parsers, auth protocol implementations. Core principle: "framing bugs live in disagreement, not in one parser." Classes: request smuggling/desync, web cache poisoning (unkeyed input), cache deception, host-header trust, CRLF/response header injection, JWT verification defects (alg confusion, decode-without-verify, missing claim checks, key-selection injection), OAuth/OIDC flow defects (redirect_uri validation, missing/weak state, PKCE, id_token validation, IdP confusion), SAML assertion defects (signature wrapping, signature exclusion, XXE, comment truncation, missing replay/binding checks), session-management defects, password-reset/account-recovery defects. Critical constraint: framing/cache bugs frequently depend on components not in the audited tree — must mark unverifiable rather than downgrade.
- **CLIENT-SIDE.md**: For SPAs, browser extensions, embedded webviews. Core principle: "server escaping ends where the fragment begins." Classes: DOM-based XSS, DOM clobbering, postMessage origin trust, cross-site WebSocket hijacking, CORS with credentials, clickjacking, reverse tabnabbing, client-side open redirect/navigation, prototype pollution (with gadget chain requirement). Framework auto-escaping is a real mitigation — only opt-out paths (dangerouslySetInnerHTML, v-html, bypassSecurityTrust*) are candidates.

### Validation as adversarial architecture

The validation strategy is a three-stage adversarial pipeline:
1. **Hunter self-check** (Phase 2): Every agent applies 6 validation rules before reporting — construct concrete attack, confirm meaningful impact, check for mitigating layers, compare against baseline, verify parser/runtime assumptions.
2. **Adversarial disprove** (Phase 3): Separate agents explicitly told "Your job is to DISPROVE this finding" apply 5 tests (exploitation, impact, baseline, mitigation, parser/runtime) and return CONFIRMED or REJECTED with code evidence.
3. **Independent verification** (Phase 6): Fresh agents verify every factual claim in the structured output — file paths, line numbers, function names, trace descriptions, root cause, execution payloads, conditions, remediation, and confidence. This catches blind spots the writer couldn't see.

This is fundamentally different from a scanner that reports everything and expects the human to triage. The adversarial architecture makes false positives expensive (they must survive two independent agents trying to kill them) while real bugs survive because they're anchored to verified code paths.

### Structured output as truth enforcement

The `report-schema.json` doesn't just define a format — it enforces rigor. A finding requires:
- A `trace` array (minItems: 2) with sequential entrypoint→propagation→sink steps, each with exact file, line number, function name, and description
- An `execution` object with attacker perspective, concrete payloads, step-by-step instructions, and expected observable result
- A `severity` object with separate likelihood and impact scores, each with a reason
- A `confidence` score with explanation
- A `root_cause` using the template "[function] in [file] does not [missing action], allowing [consequence]"
- `intended_behavior` explaining what the developer was trying to build

The schema enforces `additionalProperties: false`, so any extra field is a hard failure. The validator enforces trace semantics (entrypoint first, sink last, propagation between) — constraints the JSON Schema subset can't express. This structure forces the agent to either verify every detail or reject the finding.

## Design decisions and trade-offs

### Optimized for precision over recall

The skill explicitly optimizes for **low false positive rate** at the cost of **missing some real vulnerabilities**. "Only report what you can exploit" is the first principle. The adversarial validation architecture, the structured output requirements, the dynamic confirmation preference — all push toward "fewer, verified findings" rather than "report everything suspicious." Testing confirms a single run catches ~50% of total vulnerabilities across multiple runs, and the skill is designed for multiple additive passes rather than trying to maximize single-run coverage.

### Context budgeting through agent specialization

Rather than giving one agent the entire codebase and all attack classes (which would overflow context), the skill decomposes the problem: Phase 1 agents each explore one dimension (overview, trust boundaries, input surfaces), Phase 2 agents each get one attack class scoped to one subsystem. The `general` agent type is chosen over `research` specifically because it can spawn sub-agents — when a hunter finds a rabbit hole, it doesn't bloat its own context, it delegates. This is explicit context budgeting by design, not an accident of implementation.

### Dynamic over static: baseline calibration and scope selection

Unlike static security scanners with hardcoded rules, the skill makes two key decisions dynamically: (1) the comparable baseline is discovered during Phase 1, not supplied by the user or hardcoded; (2) the number and composition of Phase 2 agents is determined by Phase 1's architecture analysis — a small library gets 3–4 agents, a complex app gets 8–12+ split by both class and subsystem. This avoids both overkill (wasting context on nonexistent attack surfaces) and undercoverage (missing major subsystems because of a fixed agent count).

### Agent-neutral by design, Claude Code in practice

The SKILL.md includes explicit "platform terminology" mapping (Task tool, research agent, general agent, subagent_type) to make the skill portable across coding agents. But the practical implementation assumes Claude Code's capabilities: parallel sub-agent spawning, structured output tools, file system access for reading source code. The skill works because Claude Code can execute these patterns; a coding agent without parallel sub-agent support or without the ability to read arbitrary filesystem paths would not be able to follow the methodology.

### Weakness: No persistent state between runs

The skill reads prior `findings.json` to skip known findings and target gaps, but there's no deduplication database, no cross-repo learning, no attack-class-to-vulnerability mapping that improves over time. Each audit is independent; the skill itself doesn't learn. Compare to Cloudflare's internal vulnerability harness (which this skill seeded), which is described as a "multi-stage, fleet-wide system" — the skill is the single-repo starting point, not the production system.

### Weakness: Depends on agent model quality

The quality of the audit depends entirely on the quality of the underlying model's code-reading ability. The skill provides the orchestration, methodology, and quality gates, but if the model cannot accurately trace data flow through indirection, cannot distinguish meaningful from trivial, or has a known false-positive bias for certain pattern classes, the adversarial validation can only catch some of those errors — the validators use the same model family.

## Comparison with related approaches

### vs Static analysis scanners (Semgrep, CodeQL, SonarQube)

Traditional SAST tools use pattern matching or dataflow analysis to find known vulnerability patterns. The security-audit skill is complementary: it finds the things scanners can't — business logic errors, chained attacks, trust boundary violations, and context-dependent design flaws. The skill's "Wildcard" and "Business logic" agents are explicitly hunting for the class of bugs that rule-based scanners miss. The trade-off is scalability (an LLM audit costs far more per line than a scanner) and determinism (different runs yield different findings).

### vs Penetration testing tools (Burp Suite, ZAP)

Dynamic testing tools probe running applications. The security-audit skill is a source-code audit — it can find vulnerabilities reachable through code paths that dynamic testing might miss (obscure endpoints, race condition windows, configuration-dependent paths) but cannot confirm runtime behavior (cache configuration, proxy chains, live token validation) without deploying the target.

### vs Manual security review

The skill automates the *structure* of a manual review (parallel investigation, adversarial validation, structured reporting) but the *quality* depends on model capability. In the Cloudflare blog post describing the internal harness this skill seeded, multiple parallel agents produced higher-quality findings than single-agent approaches. The key insight is that the adversarial architecture (find → try to disprove → independent verify) mirrors the structure of a good manual security team but executes it programmatically.

### vs The vulnerability harness it seeded

Per the README, this skill is the "single-repo starting point" from which Cloudflare's internal vulnerability harness evolved. The internal harness is described as "multi-stage, fleet-wide" — implying it scans across many repos, possibly with persistent state, learning, and integration into CI/CD. The skill is the extractable, shareable methodology without the production infrastructure.
