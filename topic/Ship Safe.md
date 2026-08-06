# Ship Safe

An open-source multi-agent security CLI scanner that treats AI-coding-agent surface area — MCP configs, agent instruction files, prompt injection surfaces, hallucinated package imports — as a first-class security domain alongside traditional application security. 29 parallel agents, ~44K lines of JavaScript, offline-first with optional LLM-backed deep analysis.

---

## Architecture

Ship Safe is a **flat parallel-agent pipeline** — not a hierarchical swarm and not a sequential pass. Every agent extends a shared `BaseAgent` (`cli/agents/base-agent.js:163`) that provides file discovery (with `.ship-safeignore` and `.gitignore` support), a standardized finding format via `createFinding()`, regex-based pattern scanning with per-pattern language scoping, and a suppression floor for critical findings.

### The orchestration loop (`cli/agents/orchestrator.js`)

1. **Recon** — `ReconAgent` maps the attack surface: detected frameworks, languages, package managers, auth patterns. Used by individual agents' `shouldRun()` to skip irrelevant checks.
2. **File discovery** — shared file list filtered by default to exclude test/fixture files (89% of findings came from these paths in benchmark projects).
3. **Agent filtering** — by name, category, or `shouldRun()` relevance check. Unrelated agents are skipped before they burn time.
4. **Parallel execution** — agents run in chunks of 6 (configurable via `--concurrency`), each with a 30s timeout via `Promise.race`. Results accumulate in a `sharedFindings` array for cross-agent awareness (e.g., secrets agent finds a key → supply-chain agent checks if it's in a public repo).
5. **Deduplication** — by `file:line:rule` tuple.
6. **Verification** — `VerifierAgent` confirms or downgrades findings (secrets liveness checks).
7. **Deep analysis** (optional, `--deep`) — LLM-powered taint analysis with a 3-tier model cascade (e.g., Haiku → Sonnet → Opus for Anthropic).
8. **Confidence tuning** — findings in test files, docs, examples, or comment lines get their confidence downgraded to reduce noise.
9. **Scoring** — `ScoringEngine` computes a 0–100 score from confidence-weighted deductions across 8 categories, using a soft-cap curve so no single category can bottom out the score.

### Agent categories and scoring weights

| Category | Weight | Role |
|---|---|---|
| `secrets` | 15 | Hardcoded credentials, keys, tokens |
| `injection` | 15 | SQL/NoSQL injection, XSS, SSRF, command injection |
| `auth` | 15 | JWT flaws, CSRF, OAuth misconfig, BOLA/IDOR |
| `deps` | 13 | CVE-aware dependency auditing |
| `supply-chain` | 12 | Typosquatting, dependency confusion, risky install scripts, slopsquatting |
| `llm` | 12 | Prompt injection, MCP security, agent hijacking, memory poisoning, RAG poisoning |
| `api` | 10 | API security, GraphQL, mass assignment, debug endpoints |
| `config` | 8 | Docker, Terraform, Kubernetes, CORS, CSP, CI/CD |
| `quality` | 0 (unscored) | Maintainability findings — reported, never scored |

The `quality` category (weight 0, `scored: false`) is a deliberate design choice: maintainability findings like `RUST_UNWRAP_IN_PROD` or bare `except:` are worth surfacing but are not security defects. This prevents non-security findings from dragging down a security score — the same line SonarQube draws.

### Plugin system (`cli/utils/plugin-loader.js`)

Users drop custom `.js` files into `.ship-safe/agents/` that export a default class extending `BaseAgent`. The loader validates that the class implements `analyze()` before instantiation. Plugins run in-process alongside built-in agents with the same timeout and concurrency model. This is a straightforward extension point that makes the agent pool open-ended without requiring the framework to predict every use case.

### The suppression floor

A notable design choice: inline `ship-safe-ignore` comments cannot suppress findings at `critical` severity (`cli/agents/base-agent.js:142`). The reasoning is that anything that can write source code — including an AI agent with `ship_safe_suppress_finding` tool access — can write the suppression comment too. Critical findings always surface. An attempt to suppress one is logged separately, so a scan that tried to hide findings never reads like a scan that had none.

---

## Key Techniques

### Slopsquatting detection (`cli/agents/slopsquat-agent.js`)

A genuinely novel technique. AI coding assistants confidently hallucinate package names, and attackers register those names to ship malware ("slopsquatting" / "HalluSquatting"). The `SlopSquatAgent` detects phantom imports structurally and offline: a bare module specifier that is imported in source but is (a) not a Node builtin, (b) not in `package.json`, and (c) not in `node_modules`. The key insight is that `_resolvesNearby()` walks UP the directory tree checking each ancestor's `package.json` and `node_modules`, so monorepos and workspaces don't produce false positives. It also maintains a curated list of known-hallucinated names that raise confidence to `high`.

### Trust boundary attacks (`cli/agents/trust-boundary-agent.js`)

Surfaces two attack classes unique to AI coding agents:

- **GhostApproval**: a symlink with an innocuous name (e.g. `project_settings.json`) pointing at `~/.ssh/authorized_keys`. When the agent "sets up the workspace" and writes what it thinks is config, it writes through the symlink into the real target. Detection walks the repo tree with `lstat` and checks every symlink target against a list of sensitive paths.
- **Friendly Fire**: agent-read docs (README, CLAUDE.md, CONTRIBUTING.md) that instruct the agent to run commands during setup or review. Detection uses regexes for `curl|bash` patterns and "run this during setup/review" instructions, with smart downgrading when the command targets the project's own domain.

### Confidence-weighted deduplication and suppression accounting

Every scan tracks both `suppressedCount` and `floorSuppressionAttempts` at the agent level, rolled up to the scan report. A CI run can gate on "suppression attempts" as well as findings. The confidence multiplier (`high` = 1.0, `medium` = 0.6, `low` = 0.3) in the scoring engine means findings in test files and docs don't swamp the score.

### Security memory (`cli/utils/security-memory.js`)

A per-repo false-positive learning system stored in `.ship-safe/memory.json`. After deep analysis confirms a finding is a false positive, the (rule + basename + snippet hash) key is saved. On the next scan, matched findings are suppressed before scoring — the same Hermes-inspired pattern of learning from runtime decisions. Keys deliberately exclude line numbers so suppressions survive minor refactors. SHA-256 hashing of `rule::basename::snippet` makes keys stable.

### GPT-Red agent (`cli/agents/gpt-red-agent.js`)

When an LLM provider is configured, runs a bounded attacker/defender/judge simulation against agent-readable repo surfaces (README, CLAUDE.md, MCP configs, RAG documents). Four scenarios: local-file injection, MCP tool-output injection, CI/build-script poisoning, and RAG document poisoning. Falls back to deterministic offline regex checks when no provider is available. By default caps context at 6KB per file and 60KB total bundle.

### Language-scoped patterns

A pattern in `scanFileWithPatterns()` can declare `langs: ['js']` to only match against JavaScript/TypeScript files, preventing e.g. a Flask-specific regex from matching Python's `os.path.join()` inside a JavaScript file-upload check. This is an additive feature — patterns without `langs` run everywhere, so existing rules are unaffected.

---

## Design Decisions

### Optimized for precision, not just recall

The README explicitly benchmarks false-positive rates against express, requests, flask, and chalk, with grades from B to D. After v9.6.3, findings dropped from 1031 to 73 across the same four projects while maintaining detection against NodeGoat and DVWA. The confidence-tuning pass (`orchestrator.js:307`) aggressively downgrades findings in test files, documentation, and comment lines — this is the pragmatic choice for a tool meant to be run on every commit, not just during periodic audits.

### Offline-first with optional AI

Core scanning is entirely deterministic regex + pattern matching. AI is additive: `--deep` for LLM taint analysis, `--gpt-red` for red-team scenarios. The `--no-ai` flag guarantees a fully local scan. This sidesteps the cost, latency, and privacy concerns of mandatory cloud AI while still offering it as an acceleration option.

### Default-deny for test files

Test files, fixtures, and examples are excluded from scanning by default (`orchestrator.js:103`). The README reports that 89% of findings came from these paths in benchmark projects, including 528 of express's 601 `API_NO_SECURITY_HEADERS` hits in `test/` vs. 2 in `lib/`. This is a meaningful choice: the tool assumes tests are illustrative and opts projects into scanning them explicitly.

### "Ship Safe alongside, not instead of"

The README is explicit that Ship Safe does not compete with CodeQL (interprocedural taint), Gitleaks (secrets specialist), or Trivy (CVE database). It covers the narrower question of what an AI coding agent just did to your repo, CI, and local tool configuration. The coverage matrix in `docs/comparison.md` documents where it beats and loses to each tool.

### A commercial boundary at the cloud

The CLI is fully MIT-licensed. The cloud dashboard (scan history, PR Guardian, team workflows) lives in a private repo. This is a common but honest split: the scanner is open, the collaboration layer is commercial.

---

## Comparison Notes

**vs. [[VulnHunter]]**: Both are open-source agentic security tools for AI-assisted development. VulnHunter is Claude Code skills-based with a Hunt → Fix → Verify pipeline and adversarial falsification. Ship Safe is a standalone CLI with 29 parallel agents, a broader scope (including non-agent vulnerabilities like SQL injection and hardcoded secrets), and CI/CD integration via SARIF output. Ship Safe's offline-first design and suppression floor are distinctive.

**vs. [[Metis — ARM AI Security Code Review]]**: Both combine static analysis with LLM-driven verification. Metis focuses on C/C++ with tree-sitter call-graph reachability analysis. Ship Safe is polyglot (JS/TS/Python/Ruby/Go/Java/Rust and config files) with a lighter static analysis approach (regex patterns rather than AST analysis) but far broader agent coverage — 29 agents spanning code, supply chain, AI, and CI/CD.

**vs. [[Agent Skills for Security Testing]]**: That project distills 4,000+ HackerOne reports into grep commands and curl tests. Ship Safe's approach is similar in spirit (deterministic checks over LLM judgment) but ships as a packaged CLI with scoring, CI integration, and an extensible agent framework rather than as a library of skills.

**vs. [[AI Security Framework for DevSecOps]]**: PreEmptive's framework is a methodology guide. Ship Safe is an operational tool that implements many of the same principles (CI/CD enforcement, SARIF output, severity gating) as shipping software.

**vs. [[Agentic AI Security Stack]]**: Fernando Lucktemberg's 200+ page reference covers threat models and kill chains across OWASP, MITRE ATLAS, and CSA MAESTRO. Ship Safe operationalizes a subset of those concerns — particularly the MCP, agent config, and supply-chain attack surfaces — as deterministic checks with live CI gating.

---

*Sources: [[raw/ship-safe]], [[summary/ship-safe]]*
*Last updated: 2026-08-06*
*Tags: #tool #security #agents #cli #supply-chain #devsecops #SAST*
