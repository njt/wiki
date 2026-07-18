---
url: https://github.com/instavm/security-skills
title: Security Skills for CLI Agents
author: instavm
date_fetched: 2026-07-18
date_published: 2025
---

# Security Skills for CLI Agents

A collection of 16 specialized security testing skills for AI coding agents (Claude Code, Gemini CLI, or any agent supporting MCP/Skills). Built from analysis of 4,000+ paid HackerOne bug bounty reports, each skill distills vulnerability-specific patterns into actionable grep/regex searches, curl test commands, severity ratings, and false positive guidance. Installable via `npx skills add instavm/security-skills`.

## Repository Structure

```
skills/
  mitm-find-idor/SKILL.md         — Insecure Direct Object Reference (168 lines)
  mitm-find-auth/SKILL.md         — Authentication & authorization issues (202 lines)
  mitm-find-bizlogic/SKILL.md     — Business logic flaws (251 lines)
  mitm-find-ssrf/SKILL.md         — Server-Side Request Forgery (230 lines)
  mitm-find-sqli/SKILL.md         — SQL injection patterns (271 lines)
  mitm-find-otp/SKILL.md          — OTP/2FA bypass (70 lines)
  mitm-find-pii/SKILL.md          — PII exposure (61 lines)
  mitm-find-secrets/SKILL.md      — Leaked secrets & API keys (63 lines)
  mitm-find-callback/SKILL.md     — Payment callback/webhook issues (69 lines)
  mitm-find-checksum/SKILL.md     — Checksum/integrity bypass (76 lines)
  mitm-find-enumerable/SKILL.md   — Enumerable endpoints & IDs (66 lines)
  mitm-find-insecure/SKILL.md     — Insecure configurations (60 lines)
  mitm-find-referer/SKILL.md      — Referer header leakage (57 lines)
  mitm-list-apis/SKILL.md         — API endpoint extraction from traffic (36 lines)
  mitm-subdomains/SKILL.md        — Subdomain enumeration (56 lines)
  mitm-security-audit/SKILL.md    — Comprehensive 8-step security audit checklist (73 lines)
  mitm-report/SKILL.md            — Security vulnerability report generation (71 lines)
README.md                         — Setup and usage guide (92 lines)
```

Total: ~1,830 lines across all SKILL.md files, plus README. No code — this is a pure prompt-engineering artifact. Each skill is a self-contained markdown file with YAML frontmatter (name, description) followed by structured vulnerability detection guidance.

## Architecture & Design

### Skill Format

Every skill follows a consistent template:
1. **YAML frontmatter** with `name` and `description` (for skill registry integration)
2. **Header** with the skill's specific mission and `$ARGUMENTS` variable
3. **Prerequisites block** — always `mitmdump --set flow_detail=3 2>&1 | tee log.txt`
4. **High-value patterns** — vulnerability-specific grep patterns and real HackerOne bounty examples with source URLs
5. **Vulnerability categories** — taxonomy with severity ratings (CRITICAL/HIGH/MEDIUM/LOW/INFO)
6. **Testing methodology** — step-by-step curl/bash commands for reproducing findings
7. **Structured output format** — template for each finding (endpoint, evidence, impact, remediation)
8. **False positives** — explicit patterns to ignore, preventing over-reporting

### Skill Interdependence

Each skill is **degenerately independent** — intentionally designed to work in complete isolation. There is no shared state, no vulnerability database, no cross-skill deduplication. The coordinating `mitm-security-audit` skill is a lightweight 73-line checklist that invokes the analysis patterns from individual skills sequentially, not a shared protocol layer.

This independence has a deliberate consequence: overlapping findings. An IDOR finding from `mitm-find-idor` may also appear as an auth finding from `mitm-find-auth`. The `mitm-report` skill stitches them together into a coherent report, but only at presentation time — there is no programmatic deduplication.

### Input Model

All skills share a single common input: `log.txt` produced by mitmproxy's dump mode. The skills use grep and regex against this plain-text traffic log rather than parsing structured formats like HAR or mitmproxy's native flow files. This is a deliberate simplicity trade-off:
- **Pro**: Works with any text-based traffic capture, minimal dependencies, easy to explain
- **Con**: No structured access to request/response pairs, headers, or bodies; regex-based extraction is fragile and misses encoded/compressed content

## Key Techniques (Code-Level Analysis)

### Vulnerability-Specific Grep Signatures

The core innovation is the **parameter vocabulary** — each skill encodes a curated list of parameter names that are likely to be associated with a specific vulnerability class. For example, IDOR parameters:

```
user_id, userId, user-id, uid, account_id, accountId
customer_id, customerId, member_id, memberId
profile_id, owner_id, creator_id, author_id
order_id, orderId, booking_id, bookingId, reservation_id
transaction_id, txn_id, payment_id, invoice_id
document_id, doc_id, file_id, attachment_id
```

These aren't generated — they're distilled from 132 real HackerOne IDOR bounty reports. The skill encodes both the parameter name AND its typical location (path, query, body, header) and the ID encoding pattern (sequential, Base64, hex, UUID, short hash, padded).

### Database-Specific Payload Tables

The SQLi skill (271 lines, the longest) encodes database-specific payloads for four databases:

```sql
# MySQL:    ' AND SLEEP(5)--
# PostgreSQL: '; SELECT pg_sleep(5)--
# MSSQL:    '; WAITFOR DELAY '0:0:5'--
# Oracle:   ' AND DBMS_PIPE.RECEIVE_MESSAGE('a',5)--
```

This isn't just a payload list — it encodes the **detection methodology**: error-based (grep for database-specific error strings), time-based (measure response time with sleep payloads), boolean-based (compare true/false condition responses), and union-based (increment NULL columns to find column count).

### SSRF Bypass Encoding Table

The SSRF skill encodes an IP representation bypass table, not just target payloads:

```
http://127.0.0.1 → http://2130706433 (decimal)
http://127.0.0.1 → http://0x7f000001 (hex)
http://127.0.0.1 → http://0177.0.0.1 (octal)
http://127.0.0.1 → http://127.1 (short form)
http://localhost → http://localtest.me (DNS bypass)
```

This is tactical knowledge from real SSRF exploitation, not academic taxonomy.

### Business Logic Flaw Taxonomy

The business logic skill (251 lines, second-longest) encodes a different approach — it can't rely on parameter vocabularies because business logic flaws are too varied. Instead it uses **flow-based analysis**: mapping financial flows (payment, checkout, redeem), verification flows (email, phone, OTP), and workflow state machines (step, stage, status), then prescribing parameter manipulation tests for each.

### Race Condition Testing via Bash

```
for i in {1..10}; do
  curl -X POST 'https://target.com/api/redeem' -d '{"code":"PROMO123"}' &
done
wait
```

Simple but effective — the skill encodes the test pattern (concurrent same-coupon redemption) along with the verification check ("was code redeemed multiple times?").

## Design Decisions & Trade-offs

### Optimized for Actionability Over Comprehensiveness

Unlike a vulnerability taxonomy or academic paper, each skill is structured as an **operational playbook**: grep pattern → curl test → severity label → report template. There is no theoretical discussion of vulnerability classes, no OWASP references, no CVSS scoring. This sacrifices educational value for immediate utility — an AI agent can execute the skill's instructions without understanding the underlying vulnerability class.

### Text-Based Traffic Analysis Over Structured Parsing

Using grep on `log.txt` rather than structured mitmproxy flow parsing is the project's defining technical choice. Benefits: universal input format, zero dependencies, works with any traffic capture tool that can produce text output. Costs: no structured access to headers, bodies, or request/response pairs; regex patterns can match false positives in response bodies or non-API content.

### Independence Over Integration

Each skill works alone. There's no library, no shared vulnerability database, no cross-skill deduplication. This means installing 16 skills costs 16× the context for overlapping coverage, but also means any single skill works without the others — the user installs only what they need.

### Trust in the Agent's Synthesis Abilities

The report skill (`mitm-report`) relies on the AI agent to synthesize findings across skills, deduplicate, and prioritize — it provides only a template, not a programmatic merge algorithm. This is a bet that the LLM will do a reasonable job at synthesis without explicit coordination logic.

### Context Budgeting via Specialization

The fundamental architecture addresses the context problem by **specialization**: rather than loading 4,000 bug reports into context, each skill is a focused 50-270 line prompt. The trade-off is that comprehensive coverage requires running multiple skills sequentially, each in its own context window, and the agent must carry findings forward manually.

## Comparison to Related Projects

- **Metis (ARM)**: Uses deterministic tree-sitter analysis + LLM confirmation, not LLM-only analysis. Security-skills is purely prompt-based; Metis uses code as ground truth with LLM as a verifier.
- **OpenCodeReview**: General code review, not security-specific. Security-skills is focused entirely on vulnerability discovery with domain-specific patterns.
- **The Agentic AI Security Stack**: Academic/unified threat model for agentic AI systems. Security-skills is an operational playbook for web application penetration testing — different domain (web apps, not AI agents).
- **Skill Retriever**: A skill discovery mechanism for Hermes. Security-skills is a skill *library* — the skills themselves, not the discovery infrastructure.
- **Steering Claude Code**: Categorizes skills as one of seven instruction-delivery mechanisms. Security-skills is a concrete instance of that category — a domain-specific skill library.

The project occupies a unique niche: it's not a framework, runtime, or scanner — it's a **knowledge transfer mechanism** that encodes human security expertise into LLM-consumable prompts. The "4,000+ HackerOne reports" is the moat — the skills encode patterns that are only learnable from large-scale bounty report analysis, not from reading documentation or academic papers.
