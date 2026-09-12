---
url: https://blog.cloudflare.com/ai-code-review/
title: Orchestrating AI Code Review at Scale
author: Ryan Skidmore
date_fetched: 2026-07-05
date_published: 2026-04-20
site: Cloudflare Blog
topics:
  - ai-code-review
  - agent-orchestration
---

Cloudflare built a CI-native multi-agent code review system using OpenCode that launches up to seven specialized reviewers (security, performance, code quality, documentation, release management, compliance, AGENTS.md) with a coordinator agent that deduplicates, judges severity, and posts a single structured review. Key metrics from March 10–April 9, 2026: 131,246 review runs across 48,095 MRs in 5,169 repositories, median completion time of 3 minutes 39 seconds, median cost of $0.98 (average $1.19). The system uses risk-tiered review (trivial/lite/full), tiered model assignment (Opus for coordination, Sonnet for workhorses, Kimi K2.5 for lightweight tasks), circuit breakers with failback chains, shared context optimization (85.7% cache hit rate), and a plugin architecture with configuration via Cloudflare Workers + KV. Not a replacement for human code review, but a scaling strategy for when human review capacity is the binding constraint.

The article is notable for being the most detailed public production report of AI code review at scale — with real cost data, real metrics, and honest limitations — in an ecosystem where most teams are still experimenting.

---

## The Architecture

Cloudflare's system sits in CI, triggered by GitLab merge requests. Rather than one monolithic agent, it launches up to seven specialized reviewers as concurrent sub-sessions through OpenCode's SDK. A coordinator agent reads all seven outputs, deduplicates findings, judges severity, applies a reasonableness filter, and posts a single structured review comment.

### Why OpenCode

Chose OpenCode because Cloudflare uses it extensively internally, it's open source (45+ PRs contributed upstream), and its server-first architecture allows programmatic session creation via SDK — essential for spawning sub-reviewers.

### The Plugin Architecture

A composable `ReviewPlugin` interface with three lifecycle phases:
- **Bootstrap hooks** — concurrent, non-fatal (template fetch failures don't halt)
- **Configure hooks** — sequential, fatal (VCS failures stop the job)
- **postConfigure** — async work like fetching remote model overrides

Seven plugins handle GitLab integration, AI Gateway config, compliance against internal engineering RFCs, observability via Braintrust, AGENTS.md verification, remote model overrides, and fire-and-forget telemetry.

### Two-Layer Orchestration

1. **Coordinator Process:** OpenCode spawned as child process via `Bun.spawn`. Prompt passed via stdin (avoiding `ARG_MAX` kernel limits). Output arrives as JSONL events on stdout.
2. **Review Plugin:** A runtime plugin provides `spawn_reviewers` tool inside OpenCode. Launches sub-reviewer sessions through OpenCode's SDK client, each in its own session with independent file read, grep, and search capabilities.

### The Streaming Pipeline

Coordinator output processed in real-time, buffered and flushed every 100 lines or 50ms. Watches for `step_finish` (token tracking), `error` (retry), and `reason: "length"` on step_finish (max_tokens hit → auto-retry). A heartbeat log every 30 seconds ("Model is thinking... Ns since last output") prevents users from canceling what appears hung.

---

## Specialized Reviewers

Instead of one big prompt, seven domain-specific agents with tightly scoped prompts saying what to flag **and what NOT to flag**. The author's key insight:

**"Telling an LLM what NOT to do is where the actual prompt engineering value resides."**

Each reviewer produces structured XML findings with severity: `critical`, `warning`, or `suggestion`.

### Security Reviewer — The Negation Principle

**What to flag:** injection vulnerabilities, auth/authz bypasses, hardcoded secrets, insecure crypto, missing input validation.

**What NOT to flag:** theoretical risks requiring unlikely preconditions, defense-in-depth when primary defenses are adequate, issues in unchanged code, "consider using library X" suggestions.

### Tiered Model Assignment

- **Top-tier (Claude Opus 4.7, GPT-5.4):** Coordinator only — the "hardest job" of reading seven outputs and making final judgment calls
- **Standard-tier (Claude Sonnet 4.6, GPT-5.3 Codex):** Workhorses — Code Quality, Security, Performance
- **Kimi K2.5:** Lightweight text-heavy tasks — Documentation, Release, AGENTS.md

All model assignments overridable dynamically via Cloudflare Worker.

---

## The Coordinator's Role — Judge Pass

After spawning sub-reviewers, the coordinator performs:

1. **Deduplication** — same issue from multiple reviewers kept once
2. **Re-categorization** — moves findings to appropriate sections
3. **Reasonableness filter** — drops speculative issues, nitpicks, false positives

### Approval Decision Rubric

| Condition | Decision |
|---|---|
| All LGTM or only trivial suggestions | approved |
| Only suggestion-severity items | approved_with_comments |
| Some warnings, no production risk | approved_with_comments |
| Multiple warnings suggesting risk pattern | minor_issues |
| Any critical item or production safety risk | significant_concerns (block merge) |

The bias is explicitly toward approval. A "break glass" escape hatch exists — if a human comments "break glass," the system forces approval regardless of AI findings. Used only 288 times (0.6% of MRs).

---

## Risk Tiers

| Tier | Lines | Files | Agents | Avg Cost |
|---|---|---|---|---|
| Trivial | ≤10 | ≤20 | 2 | $0.20 |
| Lite | ≤100 | ≤20 | 4 | $0.67 |
| Full | >100 or >50 files | Any | 7+ | $1.68 |

Security-sensitive files (touching `auth/`, `crypto/`, etc.) always trigger full review. Trivial tier downgrades coordinator from Opus to Sonnet.

### Diff Filtering

Before agents see code, the pipeline strips noise files: lock files, vendored dependencies, minified assets, source maps. Generated files (marked `// @generated` etc.) are filtered, but database migrations are explicitly exempted — they contain schema changes needing review.

---

## Infrastructure Engineering

### Concurrent Orchestration: `spawn_reviewers`

Manages up to seven concurrent reviewer sessions with:
- **Circuit breakers** — three states per model tier; one probe request after 2-min cooldown
- **Failback chains** — e.g., `opus-4-7` → `opus-4-6` → null
- **Per-task timeouts** — 5 minutes (10 for code quality)
- **Overall timeout** — 25 minutes hard cap
- **Retry budget** — 2 minutes minimum
- **Inactivity detection** — kill sessions with 60s no output

Only retryable API errors (429, 503) trigger failback. Auth errors, context overflow, aborts, and structured output errors skip failback — a different model won't fix bad credentials.

### Model Routing Control Plane

CI job fetches model routing from a Cloudflare Worker backed by Workers KV. Disabling a provider in KV filters all its models within five seconds. Config also carries failback overrides — reshaping entire model routing topology from a single Worker update.

### Prompt Injection Defense

Agent prompts built at runtime by concatenating agent-specific markdown with shared `REVIEWER_SHARED.md`. User-controlled content (MR descriptions) sanitized — boundary tags like `</mr_body><mr_details>` stripped via regex to prevent XML structure breakout.

### Shared Context for Token Efficiency

Per-file patch files written to a `diff_directory`. A shared context file (`shared-mr-context.txt`) extracted from coordinator prompt and written to disk — avoiding 7× duplication of MR context across concurrent reviewers. Result: **85.7% cache hit rate**, saving "an estimated five figures."

---

## AGENTS.md Reviewer — Preventing Instruction Rot

Built specifically to catch when teams change tooling without updating their AI instructions. If a team migrates from Jest to Vitest without updating AGENTS.md, the AI will keep writing Jest tests.

Changes classified into materiality tiers:
- **High:** package manager changes, test framework changes, build tool changes, directory restructures, new required env vars, CI/CD changes
- **Medium:** major dependency bumps, new linting rules, API client changes
- **Low:** bug fixes, feature additions using existing patterns, minor dep updates, CSS changes

Anti-patterns penalized: generic filler ("write clean code"), files over 200 lines causing context bloat, tool names without runnable commands.

---

## Re-Reviews — Incremental Awareness

When new commits push, the system runs incremental re-review aware of previous findings:
- Fixed findings omitted; DiffNote threads auto-resolved
- Unfixed findings re-emitted to keep threads alive
- User-resolved findings respected unless the issue worsened
- "won't fix" or "acknowledged" resolves finding; "I disagree" prompts coordinator to read justification and either resolve or argue back

The reviewer also handles "one lighthearted question per MR" to "build rapport with developers who are being reviewed (sometimes brutally) by a robot."

---

## Scale Metrics (March 10 – April 9, 2026)

- **131,246 review runs** across **48,095 MRs** in **5,169 repositories**
- Average MR reviewed **2.7 times**
- **Median completion: 3 minutes 39 seconds**
- **Average cost: $1.19** (median $0.98, P99 $4.45)
- **~120 billion tokens**, 85.7% cache hit rate
- **159,103 findings** (~1.2 per review)
- Security flagged highest proportion of critical issues (4%)

**Cost by tier:**
- Trivial: 24,529 reviews, avg $0.20
- Lite: 27,558 reviews, avg $0.67
- Full: 78,611 reviews, avg $1.68

---

## Limitations (Honestly Acknowledged)

- **Architectural awareness:** Reviewers see the diff but not the full system design rationale
- **Cross-system impact:** Can flag API contract changes but can't verify downstream consumers updated
- **Subtle concurrency bugs:** Race conditions depending on timing hard to catch from static diff
- **Cost scales with diff size:** "Large MRs are inherently expensive to review"

The system is "not a replacement for human code review, at least not yet with today's models."
