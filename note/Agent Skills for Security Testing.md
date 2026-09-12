# Agent Skills for Security Testing

A library of 16 specialized security testing skills for AI coding agents, built from analysis of 4,000+ paid HackerOne bug bounty reports. Each skill distills vulnerability detection patterns into grep commands, curl tests, and structured output formats — turning a general-purpose coding agent into a security testing tool. Installable via `npx skills add instavm/security-skills`.

---

## Architecture

The project is a **degenerately flat skill library** — 16 self-contained SKILL.md files in individual directories, plus a README. There is no code, no runtime, no shared library. Each skill is a markdown prompt designed to be loaded by an AI agent's skill system (Claude Code, Gemini CLI, or any MCP-compatible harness).

### Skill structure

Every skill follows an identical template:

1. **YAML frontmatter** — `name` and `description` for registry integration
2. **Prerequisites** — always `mitmdump --set flow_detail=3 2>&1 | tee log.txt`
3. **High-value patterns** — vulnerability-specific grep signatures with real HackerOne examples
4. **Testing methodology** — step-by-step bash/curl commands
5. **Severity taxonomy** — CRITICAL/HIGH/MEDIUM/LOW/INFO mapped to concrete impact examples
6. **Structured output format** — a markdown template for consistent findings
7. **False positives** — explicit patterns the agent should ignore

The 16 skills break into three functional groups:

| Group | Skills | Purpose |
|-------|--------|---------|
| **Reconnaissance** | `mitm-list-apis`, `mitm-subdomains` | Map the target's attack surface from traffic |
| **Vulnerability detection** | `mitm-find-idor`, `mitm-find-auth`, `mitm-find-bizlogic`, `mitm-find-ssrf`, `mitm-find-sqli`, `mitm-find-otp`, `mitm-find-pii`, `mitm-find-secrets`, `mitm-find-callback`, `mitm-find-checksum`, `mitm-find-enumerable`, `mitm-find-insecure`, `mitm-find-referer` | Find specific vulnerability classes |
| **Synthesis** | `mitm-security-audit`, `mitm-report` | Orchestrate checks and produce reports |

### Independence by design

Skills are intentionally independent — no shared state, no cross-skill deduplication, no library. This means any single skill works alone (install just `mitm-find-idor` if that's all you need), but also means running all 16 skills sequentially costs 16× the context and may produce overlapping findings. The coordinating `mitm-security-audit` skill is a lightweight 73-line checklist, not a shared protocol — synthesis is left to the LLM.

### Input model

All skills share one input: `log.txt` from mitmproxy's dump mode. The skills use plain-text grep against this log rather than parsing structured formats (HAR, mitmproxy flow files). This is a deliberate simplicity choice: universal format, zero dependencies, any traffic capture tool works. The cost is fragility — regex patterns can match noise in response bodies or encoded content.

---

## Key Techniques

### Parameter vocabularies distilled from bounty data

The core technique is encoding **domain-specific parameter name vocabularies**. For example, the IDOR skill catalogs 40+ parameter names (across five categories: user, resource, organization, content, session/token) that are statistically likely to carry object references, along with their encoding patterns (sequential, Base64, hex, UUID, short hash, padded).

This is the project's innovation: it's not teaching the agent what an IDOR *is* (the model already knows that). It's teaching the agent **where to look** and **what to grep for** — operational knowledge distilled from 132 real HackerOne IDOR reports.

```bash
grep -iE '(user|account|order|session|subscription|member|card|document|file|project|team|group)[-_]?id' log.txt
```

### Database-specific detection methodology

The SQLi skill (271 lines, the largest) encodes not just payloads but a **four-method detection framework**: error-based (grep for `ORA-`, `PG::`, `Microsoft SQL`), time-based (measure response latency), boolean-based (compare true/false condition responses), and union-based (increment NULLs to find column count). Each method comes with database-specific payloads for MySQL, PostgreSQL, MSSQL, and Oracle.

### IP representation bypass encoding

The SSRF skill teaches the agent to bypass URL filters using alternative IP representations — decimal (`2130706433` for `127.0.0.1`), hex (`0x7f000001`), octal (`0177.0.0.1`), short form (`127.1`), and DNS rebinding tricks (`localtest.me`). This is tactical exploitation knowledge from real SSRF bounty reports, not academic taxonomy.

### Flow-based business logic analysis

The business logic skill uses a different detection strategy — parameter vocabularies don't work for logic flaws. Instead it maps the application's **flows** (payment, verification, workflow state machines) and prescribes manipulation tests for each: price manipulation, status injection, race conditions via concurrent curl loops, and workflow step skipping.

### Structured output as reliability mechanism

Every skill mandates a specific output format — a markdown template with fields like `**Endpoint**`, `**Severity**`, `**Evidence**`, `**Test Command**`, `**Remediation**`. This isn't cosmetic; it's a reliability mechanism. Structured output forces the agent to produce verifiable, reproducible findings rather than vague claims. A finding without a curl command isn't a finding.

---

## Design Decisions

### Actionability over comprehensiveness

The skills are **operational playbooks**, not educational material. No OWASP references, no CVSS scoring, no vulnerability theory. Just: here's what to grep for, here's the curl command to test it, here's the severity if it works, here's the report format. This sacrifices educational value for immediate utility — the agent doesn't need to understand SQL injection to find it.

### Trust in the LLM's synthesis

The report skill (`mitm-report`) provides only a template — no deduplication logic, no cross-skill correlation, no programmatic prioritization. It relies entirely on the LLM to synthesize findings from multiple skill runs into a coherent report. This is a reasonable bet for modern models but creates quality variance across different LLM backends.

### Context budgeting through specialization

The fundamental architecture solves the context problem by decomposition: rather than loading 4,000 bug reports into context, each skill is a focused 50-270 line prompt. The trade-off is that comprehensive coverage requires running multiple skills sequentially, each in its own context window, and the agent must manually carry findings forward.

### Text-based analysis as universal interface

Using `log.txt` + grep rather than structured mitmproxy parsing is the project's defining technical choice. Benefits: universal input format, no dependencies, works with any capture tool. Costs: no structured access to headers/bodies, regex fragility, no handling of encoded or compressed content. For a commercial scanner this would be unacceptable; for an AI agent that can interpret context and handle edge cases, it's pragmatic.

---

## Comparison Notes

- **vs. Metis (ARM)**: Metis uses deterministic tree-sitter call-graph analysis with LLM confirmation; security-skills is pure LLM prompt engineering. Metis catches what's structurally visible in code; security-skills catches what's visible in runtime traffic — complementary, not overlapping.
- **vs. The Agentic AI Security Stack**: That's a unified threat model for agentic AI systems themselves. Security-skills is an operational playbook for traditional web application pentesting — different domain entirely.
- **vs. CI Forge (ciforge)**: ciforge bundles ~25 deterministic code scanners; security-skills is LLM-driven traffic analysis. ciforge works without an LLM; security-skills requires one.
- **vs. OpenCodeReview**: General code review, not security-specific. Security-skills is focused entirely on vulnerability discovery with domain-specific detection patterns.
- **vs. Skill Retriever**: A skill *discovery* mechanism for Hermes. Security-skills is a skill *library* — complementary: Skill Retriever finds skills, security-skills is a skill collection it could index.

- **vs. [[Audit Skills for AI Coding Agents (metacircu1ar)]]**: Another flat skill library for AI coding agents, but working against source code (static analysis) rather than runtime traffic. Audit-skills covers broader surface area (resilience, performance, type safety, iOS launch) with 624 lines across 11 skills; security-skills is narrower but deeper, with 4,000+ HackerOne reports backing its vulnerability-specific grep patterns. Complementary: use audit-skills for broad launch-readiness, then security-skills for deep vulnerability hunting on deployed applications.

- **vs. [[bt-re-controller — Bluetooth Firmware RE Skill]]**: Both encode domain expertise as agent skills, but the shapes are opposite. This library is 16 flat, independent grep recipes that trust the model's synthesis; bt-re-controller is one deep ~35-phase DAG with deterministic Python tooling and a rule that the model must *never* name anything from memory — it looks up spec opcode tables instead. The contrast is "many shallow skills" vs. "one deep pipeline."

The project occupies a unique niche: it's a **knowledge transfer mechanism** that encodes human pentesting expertise into LLM-consumable skill prompts. The "4,000+ HackerOne reports" is the moat — the skills encode patterns only learnable from large-scale bounty analysis, not from reading documentation.

---

*Sources: [[raw/security-skills-instavm]]*
*Last updated: 2026-07-18*
