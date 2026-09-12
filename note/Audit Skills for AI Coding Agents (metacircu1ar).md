# Audit Skills for AI Coding Agents (metacircu1ar)

A library of 11 composable audit skills that turn any AI coding agent into a production-readiness auditor. Each skill is a focused markdown playbook (42–77 lines) teaching the agent what to grep for, check, and report — auth gaps, missing input validation, hardcoded secrets, N+1 queries, and eight other dimensions. The skills are harness-agnostic (any agent that loads markdown prompts), framework-agnostic (the agent discovers the repo's patterns), and designed to produce concrete findings with file:line + impact + fix rather than vague checklists. MIT licensed, single author, 624 lines total.

---

## Architecture

### Degenerately flat skill library

The repo is 11 self-contained `SKILL.md` files in individual directories under `skills/`, plus a README. **No code, no runtime, no shared library, no dependencies.** The design is intentionally minimal — each skill is a standalone markdown prompt loadable by Claude Code, Codex, Gemini CLI, or any MCP-compatible harness.

```
skills/
  audit-input-validation/SKILL.md           — forms, API handlers, schemas
  audit-auth-and-access-control/SKILL.md    — login, sessions, roles, cross-user data
  audit-secrets-and-config/SKILL.md         — hardcoded keys, env validation, webhooks
  audit-rate-limiting/SKILL.md              — API route rate-limit coverage
  audit-cors/SKILL.md                       — origin allowlists, credentials
  audit-database-performance/SKILL.md       — indexes, pagination, connection pools
  audit-resilience-and-observability/SKILL.md — error boundaries, health checks, backups
  audit-asset-pipeline/SKILL.md             — uploads, CDN, object storage
  audit-type-safety/SKILL.md                — TypeScript bypasses, unchecked external data
  security-review/SKILL.md                  — umbrella: compose all 9 into OWASP report
  ios-prelaunch-checklist/SKILL.md          — App Store assets, signing, legal
```

### Skill anatomy

Every skill follows an identical template:

1. **YAML frontmatter** — `name` and `description` for registry integration
2. **Numbered workflow** — sequential steps: map surfaces → verify each → check tests → report
3. **Useful Searches** — a curated list of grep terms for the audit domain (e.g., `current_user`, `user_id`, `policy`, `authorize` for auth; `route`, `controller`, `schema`, `validate`, `zod`, `yup` for input validation)
4. **Output format** — a structured template (tables for coverage audits, severity-grouped findings lists)

### The composition strategy

The `security-review` skill is the orchestrator. It instructs the agent to:

1. Identify the app shape (frameworks, entrypoints, auth model, database)
2. Run all 9 focused sibling skills
3. Map findings to OWASP risk classes (broken access control, injection, cryptographic failures, etc.)
4. Deduplicate — keep each finding under its most specific category
5. Validate high-impact claims by re-reading the code path
6. Output severity-grouped findings with coverage summary

This is **LLM-mediated composition**: no deterministic orchestrator, no programmatic skill-chaining. The skill *instructs* the agent how to compose; the agent performs the composition. The same pattern appears in [[Cloudflare Security Audit Skill]]'s parallel-agent pipeline and [[PAAD — Defense-in-Depth for AI-Assisted Development]]'s specialist+verifier pattern.

The `ios-prelaunch-checklist` skill is a sister umbrella — it composes into the security review when the app targets iOS, covering App Store assets, technical setup, legal readiness, and signing.

---

## Key Techniques

### Search vocabularies as operational knowledge

Each skill's primary technique is encoding **what to grep for**. The auth skill catalogs 25+ search terms (`current_user`, `user_id`, `account_id`, `role`, `admin`, `policy`, `authorize`, `session`, `token`, `refresh`, `reset`, `localStorage`, `find`, `get`, `delete`, `update`, `download`) — not teaching the agent what auth bypass *is* (the model already knows), but where to *look*. This is operational knowledge transfer: a skilled auditor's internal checklist encoded as grep patterns.

The same pattern appears in [[Agent Skills for Security Testing]]'s parameter vocabularies (40+ IDOR parameter names across five categories) distilled from 4,000+ HackerOne reports. Both libraries encode the same insight: **the model knows the theory; the skill supplies the search surface**.

### Route-to-form mapping

The input validation skill instructs the agent to build a **route-to-form map** by tracing client API calls to server handlers. The agent follows every user-controlled input from UI through to handler, checking for validation at both layers. This is the same endpoint-inventory pattern from [[How AI Coding Agents Actually Use Your Technology]], applied to audit rather than API design — the agent discovers the attack surface before testing it.

### Cross-layer mandatory server validation

A recurring rule across multiple skills: **treat server-side validation as mandatory even when client validation exists**. TypeScript types, UI placeholders, HTML input types, and OpenAPI docs do not count as validation unless runtime code enforces them. This is a concrete defense against the "I validated it on the client" anti-pattern that every framework encourages.

### Blast-radius prioritization

Skills instruct the agent to prioritize by **blast radius**: shared utilities rank above isolated UI glue, auth/payment code ranks above documentation, `any` in a type exported from `utils.ts` ranks above `any` in a one-off component. This is the same differential-priority heuristic from [[Steering Claude Code]] and [[Building Agents for Production Systems with MCP]] — focus where the cost of being wrong is highest.

### Output-as-reliability-mechanism

Every skill mandates findings include **file:line + one-sentence impact + suggested fix matching codebase style + missing test**. This isn't cosmetic — it's a reliability mechanism. A finding without a file reference isn't a finding; a fix that doesn't match the codebase's validation conventions isn't a fix; a missing test that doesn't follow the project's test patterns won't be adopted. The structured output format forces the agent to produce **verifiable, reproducible findings** rather than vague "consider adding validation" notes.

### Cross-user data isolation as a specific sub-technique

The auth skill includes a particularly sharp section on cross-user data isolation. It flags a pattern that kills startups: `find(id)` followed by no ownership check. The instruction is blunt: "Prefer queries that include the current-user or current-tenant constraint in the database lookup itself." It also teaches the agent to check that client-supplied `user_id`, `account_id`, or `tenant_id` parameters are never trusted directly — a bug class that looks correct in feature tests but fails under any adversarial access pattern.

### OWASP mapping as standardization bridge

The `security-review` umbrella skill maps findings onto OWASP risk classes. This is a **standardization bridge** — it translates the library's internal finding format into the risk taxonomy that security teams and compliance frameworks speak. The same approach appears in [[Metis — ARM AI Security Code Review]]'s SARIF-native output and the [[Agentic AI Security Stack]]'s unified threat model.

---

## Design Decisions

### General over specific

Skills are framework-agnostic and language-agnostic (except `audit-type-safety` which targets TypeScript). Instead of hardcoding `grep -r "@PostMapping"`, skills provide semantic terms (`route`, `router`, `controller`, `handler`) and instruct the agent to "use the repository's framework conventions first." This makes skills portable but adds a step: the agent must discover the framework's routing pattern before it can audit. The bet is that **portability beats precision** for a general-purpose library — and the agent's own codebase knowledge fills the gap.

### Sequential composition over parallel

Unlike [[Cloudflare Security Audit Skill]]'s parallel-agent architecture (all dimensions run simultaneously), audit-skills composes sequentially — the `security-review` skill runs each sibling skill one at a time, with the agent carrying findings forward in context. This is simpler (no fan-out infrastructure needed) and more portable (no multi-agent orchestration dependency), but means running all 11 skills costs 11× the context, and findings from skill #11 can't inform skill #1.

### Context budgeting through narrow scoping

Each skill is 42–77 lines — small enough to fit in a single context window alongside the code it audits. This is the fundamental architecture: **decompose the audit surface into narrow dimensions, each small enough for a single agent turn**. The trade-off is that comprehensive coverage requires multiple turns; the win is that each turn produces concrete findings rather than broad but shallow coverage.

### Source-code analysis, not runtime testing

Unlike [[Agent Skills for Security Testing]] which operates on mitmproxy traffic logs (runtime analysis), these skills work against **source code alone**. No runtime environment, no test execution, no traffic capture needed — the agent reads code and reports findings from static analysis. This makes the skills usable on any repo (no setup, no credentials) but means they can't find runtime-only issues (race conditions, memory leaks, production config drift).

### No evaluation infrastructure

The library's main gap: there are no example audit reports, no benchmark of findings-per-skill, no regression suite. A skill library without evaluation is like a test suite without assertions. Compare to [[Razorback]]'s reproducible benchmarking, [[FrontierCode]]'s tiered scoring, or [[AXIS — Netlify's Agent Experience Measurement Framework]]'s four-dimension measurement. This is less a criticism and more a reflection of where the ecosystem is — skill evaluation infrastructure is nascent, and "eyeball the results" is still the state of the art.

---

## Comparison Notes

- **vs. [[Agent Skills for Security Testing]]**: Both are flat skill libraries with YAML + markdown prompts. Security-testing uses mitmproxy traffic as input (runtime pentesting); audit-skills works against source code (static analysis). Security-testing has the 4,000-HackerOne-report moat; audit-skills covers broader surface area (resilience, performance, type safety, iOS launch). **Complementary**: use audit-skills for broad launch-readiness, then security-testing skills for deep vulnerability hunting on the deployed app.

- **vs. [[Cloudflare Security Audit Skill]]**: Cloudflare's is a multi-phase parallel-agent pipeline (recon → hunt → adversarial validation → structured output → independent verification) purpose-built for finding exploitable vulnerabilities. Audit-skills are single-agent, sequential playbooks for broad production-readiness coverage. Cloudflare targets **depth** (adversarial verification, independent repro); audit-skills targets **breadth** (11 dimensions, "did you forget anything?"). A team could use audit-skills as the first pass and Cloudflare's skill for deep-diving the high-risk findings.

- **vs. [[PAAD — Defense-in-Depth for AI-Assisted Development]]**: Both are Claude Code skill suites. PAAD targets the development *workflow* (spec critique, architecture review, TDD-enforced quick fixes); audit-skills targets the *codebase* (validation, auth, secrets, performance). PAAD runs *during* development; audit-skills runs *after* — at launch, before release, or during security review. Both share the composable-skills philosophy and specialist+verifier pattern, but operate at different points in the SDLC.

- **vs. [[Agent Skills Library (dzhng)]]**: dzhng's 19-skill library is a software factory operating system (plan → slice → build → verify → repeat); audit-skills is a QA/audit toolkit. dzhng **builds**; audit-skills **checks**. Both share harness-agnostic design, skills-as-composable-units philosophy, and curatorial discipline (fewer invocation points over more granularity). A workflow combining both would have dzhng's skills building features and audit-skills' skills reviewing them before merge.

- **vs. [[VulnHunter]]**: Capital One's closed-loop Hunt → Fix → Verify pipeline uses forward-trace analysis with adversarial falsification. Audit-skills uses static pattern-matching + LLM reasoning. VulnHunter closes the loop (it also fixes); audit-skills only reports (fixes are left to the human or another agent). VulnHunter requires Capital One's internal Claude Code infrastructure; audit-skills works anywhere.

- **vs. [[Metis — ARM AI Security Code Review]]**: Metis uses deterministic tree-sitter call-graph reachability analysis with LLM confirmation; audit-skills is pure LLM prompt engineering. Metis catches what's structurally visible in code graphs; audit-skills catches what's semantically suspicious (the `find(id)` without ownership check, the hardcoded `API_KEY` in a test file, the rate limit that only keys on IP when it should also key on user). **Complementary**: Metis for structural security, audit-skills for semantic and operational concerns.

The library occupies a distinct niche: it's a **breadth-first production-readiness audit toolkit** for AI agents. Not a deep vulnerability scanner, not a development workflow guard, not a security orchestration pipeline — a set of focused playbooks for the "did we forget anything?" question every launch should answer.

---

*Sources: [[raw/audit-skills]]*
*Last updated: 2026-07-25*
