# CEOS (Claude + EOS)

CEOS is Brad Feld's open-source Claude Code skills package that turns the Entrepreneurial Operating System (EOS) — the Gino Wickman framework for running a company — into 19 AI-assisted workflows whose entire data store is git-tracked markdown. It is the sharpest public example of [[Claude Code Skills System]] applied to a non-coding domain: the "software" being produced is a functioning operating system for a leadership team, not a codebase.

---

## Architecture

CEOS is a **skills package**, not an application. The three layers are stated in the README: upstream (`bradfeld/ceos`) ships skills/templates/docs and no company data; each company forks it and commits their EOS data into `data/`; personal preferences live in a gitignored `.ceos-user.yaml`. The only executable code is two files:

- **`setup.sh`** (367 lines) — the installer. It globs `skills/ceos-*/`, symlinks each into `~/.claude/skills/` (idempotent, checks `readlink` before re-linking), and runs a guided init that scaffolds `data/` from `templates/` using `sed`-based `{{placeholder}}` substitution. Quarter math (`current_quarter`, `quarter_end_date`) is done in pure bash.
- **`dashboard/build.py`** (~3,300 lines) — a dependency-free Python build script that parses the `data/` tree and emits `docs/index.html`, a self-contained Tailwind dashboard deployed to GitHub Pages.

The actual product is **19 `SKILL.md` files** (the README says 16; `ceos-calendar`, `ceos-dashboard`, `ceos-trends` are undocumented — the docs have drifted, with the authoring guide claiming "17" at one point). Every skill follows an identical 8-section contract specified in `docs/skill-structure.md`: YAML frontmatter → `# name` heading → When to Use → Context → Process → Output Format → Guardrails → Integration Notes.

The frontmatter is four fields: `name`, `description` (must start "Use when" — a *trigger*, not a summary), `file-access` (paths it reads/writes), and `tools-used`. The last two are described as a **security manifest**: not runtime-enforced, but a declaration of intent a reviewer compares against the skill body.

The load-bearing abstraction is the **data-ownership table** in `docs/skill-structure.md`. Each skill owns exactly one directory (`ceos-rocks` → `data/rocks/`, `ceos-ids` → `data/issues/`, `ceos-vto` → `data/vision.md`, …). Other skills may read but never write, and each skill ends with a formal declaration: a *Write Principle* ("Only `ceos-rocks` writes to `data/rocks/`"), an *Orchestration Principle* for the meeting/planning skills that read broadly but write narrowly, or a *Read-Only Principle* for `ceos-trends`, which aggregates all six EOS components and writes to none.

## Key techniques

- **Single-writer-per-directory as a concurrency model.** This is the non-obvious core. Multiple Claude Code sessions on one git repo *will* collide on popular files; CEOS makes that collision structurally impossible by assigning each data domain one writer skill, and making "mention, don't invoke" (the auto-invoke guardrail) the coupling mechanism between skills. It's the same insight as [[Team-Wide Agentic Harness]] — skills as reviewed, version-controlled team infrastructure — but the contention being solved is *business-data merge conflicts*, not code.

- **Markdown + YAML frontmatter as the database.** No DB, no SaaS. `docs/eos-primer.md` is explicit about why: human-readable, git-diffable, portable, and "git history is your audit trail." The dashboard's parsers (`load_rocks`, `load_scorecard`, `load_vision`, `load_l10`, …) are hand-rolled regex over this format — `parse_frontmatter` handles scalar-only YAML with no external dependency, `parse_milestones` reads `- [x]` checkboxes, `load_vision` splits on `## ` headings and regexes `**label:** value` lines into structured data. It's deliberately fragile (scalar-only, positional parsing of the metrics table) in exchange for zero dependencies and a format humans can write by hand.

- **The dashboard is a "database view" generated at build time, with a client-side write path.** `build.py` inlines all parsed data as `window.DASHBOARD_DATA = {...}` and ships the CRUD logic as vanilla JS calling the GitHub Contents API directly (base64 PUT with SHA-based optimistic concurrency). The clever part is a **close-quarter wizard**: a step-by-step scoring UI that marks Rocks complete/dropped, carries dropped Rocks forward to the next quarter with re-slugged filenames and re-ID'd frontmatter, then walks a *draft-review* wizard to commit or push drafts to future quarters, and can stub out the next 4 quarters by creating empty `.gitkeep` files.

- **Refusal-to-build security, not warning-to-build.** `build.py` binds two previously-independent env vars: a `GITHUB_WRITE_TOKEN` inlined into the page *must* be paired with a `DASHBOARD_PASSWORD`, and `protect_with_password` *must* have the `cryptography` package. If either precondition fails it raises `SystemExit` ("REFUSING TO BUILD") rather than printing a warning and shipping plaintext — the exact bug it closes (a write token published in the clear on a crawlable Pages site). The page is AES-256-GCM-encrypted (PBKDF2, 100k iterations) and decrypted in-browser via WebCrypto, so the token only ever ships inside ciphertext. The test suite greps the written bytes for the token rather than asserting an "encryption enabled" flag.

- **Prompt engineering as process engineering.** The skills are unusually literal runbooks: `ceos-ids` encodes the 5 Whys as a step ("Don't accept the first answer… the stated problem is usually a symptom"), `ceos-trends` specifies trend-arrow thresholds (↑ = completion improved by >5pp; ↑↑ = checkup score up 0.5+), and both demand "minimum data for trends" (no arrows with fewer than 2 data points) and "no prescriptive judgments" (report "60%", not "too low"). These aren't prompt tricks — they're EOS facilitator judgment encoded as instructions.

## Design decisions

- **Simplicity and portability over robustness.** Regex parsing over a YAML library; `sed` over a templating engine; no Node/Python/Docker for the core (Python only for the dashboard, which is optional). A senior engineer would flinch at the positional table parsing, but it buys the thing CEOS is selling: "clone, run `./setup.sh`, and a CEO can use it."

- **Convention over enforcement.** The file-access manifest is not runtime-enforced, and the data-ownership rule is prose. This is the honest trade-off: it makes the skills cheap to review and safe to install (they're plain markdown, no code), but it means discipline lives in the docs and can silently drift — which it already has (the 16/17/19 count discrepancy).

- **Git as the collaboration and audit layer, not a real database.** You lose query power (the "trends" skill reimplements time-series analysis as regex over dated filenames) and gain portability, diffability, and a free audit trail. It's the same bet as [[Lovelace]]'s Markdown+YAML file store, but aimed at business operating data rather than dev-project data.

- **Client-side encryption means the server never holds the key.** The dashboard's password gate protects against *public* exposure of a write token, not against a determined attacker who has the ciphertext and the JS. It's a publish-safety control, correctly scoped — the threat model is "GitHub Pages is public," not "adversary steals the repo."

## Comparison notes

- **[[Claude Code Skills System]]** — CEOS is the spec in production, and it extends the spec with something Anthropic's reference doesn't have: a cross-skill *data-ownership* contract. The frontmatter `description`-as-trigger ("Use when") maps directly to the spec's description-as-invocation-trigger; the `file-access`/`tools-used` manifest is CEOS's own addition, a lightweight substitute for `allowed-tools` for review purposes.
- **[[Team-Wide Agentic Harness]]** — Langworth's "skills are code, review them" thesis applied to *business* data rather than a dev harness. CEOS is the endpoint of that argument: the skills *are* the product, version-controlled, and `CONTRIBUTING.md` carries a "Skill Security Review" section defining what reviewers look for (no external URLs, no credential requests, no shell beyond git).
- **[[Organizational Intelligence Systems]]** — O'Reilly's recipe is "MCP data + skill-file reasoning + expert framework + human review" to *analyze* an organization. CEOS does the same thing one step earlier: it grounds an expert framework (EOS) in skill files and markdown, but to *run* the organization — weekly L10 meetings, quarterly Rocks, scorecards — not just produce analysis of it. Same primitives, different altitude.
- **[[Lead User and the Machines That Build Machines]]** — same author. CEOS is Feld's "First User" pattern made concrete: he has the sticky information (he runs companies with EOS) and the machine (Claude Code skills) is the manufacturer. Notably CEOS calls its units "skills" not "machines" — an echo of Feld's terminological point that naming forces specificity, though here the term is inherited from Claude Code's own architecture.
- **[[Agent Skills Library (dzhng)]] / [[Audit Skills for AI Coding Agents (metacircu1ar)]]** — comparable "library of N composable skills" shape, but CEOS's skills are domain-specific (business operations) and coupled through a shared data tree and ownership table, whereas those libraries are domain-agnostic and compose without shared state.

#tool #project #claude-code #skills #agents #business #local-first

---
*Sources: [[raw/ceos]], [[summary/ceos]]*
*Last updated: 2026-08-21*
