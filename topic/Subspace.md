# Subspace

A tool that lets AI coding agents hand a Markdown file to a human for review in a native terminal TUI, then returns structured, machine-readable feedback (comments and suggested edits with text-selector anchoring) to the same agent session — without granting the human approval authority. Distributed as plugins for Claude Code and Codex, backed by a proprietary TUI binary.

---

## Architecture

Subspace is a three-layer system where the open layers (plugin manifests + bash dispatch scripts) wrap a closed-source binary that does the heavy lifting.

**Layer 1 — Skill file** (`skills/r/SKILL.md`, 131 lines): Loaded by the AI agent host (Claude Code or Codex). Defines the input contract (exactly one Markdown file, optional terminal name), terminal auto-detection logic, the permission model, and the result-reporting rules. Crucially, it defines what the skill *won't* do: plan, probe, retry, fall back, route workflows, or grant authority. The word "Decision" is architecturally forbidden.

**Layer 2 — Dispatch scripts** (614 lines of bash across 7 files): Each supported terminal gets a dedicated entry point (`review-zellij`, `review-tmux`, etc.) that sources `invocation-common` for the shared lifecycle. Two dispatch models exist:
- **Synchronous** (tmux popup, Zellij floating pane): Block the invoking process until the review surface closes.
- **Asynchronous** (Apple Terminal, Ghostty, Herdr, CMUX): Launch a self-contained "capsule" script that reports completion through a named FIFO.

The shared lifecycle (`invocation-common`, 213 lines) performs rigorous preflight validation in a fixed sequence: canonicalize the artifact path (up to 40 symlink levels), pin it with SHA-256, locate and version-check the `subspace-tui` binary (exact match required), verify the validator executable, create a mode-0700 scratch directory, dispatch, collect the result, validate it, deliver it to stdout, and clean up.

**Layer 3 — Proprietary binary** (`subspace-tui` v0.10.0-beta.3): Closed-source. Constructs the Briefing (a structured review document), snapshots the artifact, renders the native TUI, creates mode-0600 JSON results, and manages continuation state for open reviews.

**Result contract** (`validate-one-file-result`, 122 lines of jq): Two valid shapes — `review-v1-result` (closed, with annotations) and `review-v1-open-result` (continuation available). Annotations are either comments (free-text body) or suggestions (original → proposed text, anchored with W3C TextQuoteSelectors). The validator does cryptographic artifact revision verification, permission-mode checking, and briefing hash derivation — not just schema validation.

## Key Techniques

**Strict no-fallback terminal selection** (`skills/r/SKILL.md:34-40`): Terminals are probed in a fixed priority order (Zellij → tmux → Herdr → CMUX → Ghostty → Apple Terminal) by checking environment variables. If ANY signal from a family is present but not ALL, the system fails with a diagnostic naming the missing signal. It never falls through to the next terminal. This is deliberately wasteful — it trades convenience for clear, actionable error messages.

**Capsule pattern for async terminals** (`invocation-common:76-86`): For terminals that can't block the calling process, the system constructs a self-contained script that `cd`s to the artifact directory, runs `subspace-tui`, and reports its exit status through a named FIFO. Trap handlers catch HUP/INT/TERM and report specific status codes (129/130/143). The capsule is mode-0700, the FIFO is mode-0600.

**Artifact revision pinning** (`invocation-common:128-132`): The file is SHA-256 hashed before any human interaction. The validator later confirms the snapshot inside the result matches. This prevents the class of bugs where the file changes between agent handoff and human review.

**Dual-validation of result integrity** (`validate-one-file-result`): Beyond JSON schema validation, the validator confirms byte-for-byte identity between result and capture, verifies file permissions, checks briefing hash derivation, and (for open results) validates the entire package directory structure including the snapshot, briefing.json, and review log.

## Design Decisions

**Proprietary core, open integration**: The hard part (TUI, Briefing, continuation state) is closed-source. The integration layer (plugin manifests, terminal dispatch, validation) is Apache-2.0. This is the inverse of most "open core" models — the binary is the moat, not the plugin.

**Single file only**: Exactly one Markdown file, no directories, no multi-file, no other formats. This constraint simplifies everything downstream: the Briefing model, artifact identity, and result contract all scale down to a single entity.

**Neutral authority**: The result schema has no "approved"/"rejected" field. The skill text explicitly forbids calling any outcome an approval, verdict, Decision, or gate result. Feedback is input to the invoking session, never authorization to edit. An invoking workflow decides what to do with the feedback — Subspace refuses to decide for it.

**No automatic remediation**: The skill doesn't apply suggestions, fix files, or route workflows. It only reports. This keeps it composable — any workflow system can consume the result and decide its own response. Compare to tools that apply fixes automatically or record binding decisions.

**Terminal hosts as equal citizens**: Despite vastly different capabilities (blocking popups vs. GUI app launch), each terminal gets a dedicated, equally-weighted entry point. There's no "primary" terminal — just a fixed detection priority. The shared lifecycle handles differences through an `invocation_async` flag rather than per-terminal special cases.

## Comparison Notes

[[Local Review]] does local git branch review with markdown export for coding agents, but operates offline and exports for later agent consumption. Subspace operates inline during an agent session and returns structured JSON for immediate consumption.

[[The End of Code Review]] and [[Agentic Code Review]] debate the spectrum from mandatory human review to fully automated. Subspace occupies a precise middle ground: it enables human review in an agentic workflow without granting the human approval authority. The review is input, not a gate — which aligns with the "human-on-the-loop" model Addy Osmani describes.

[[Orchestrating AI Code Review at Scale]] (Cloudflare) uses 7 specialized AI agents + a coordinator judge. Subspace takes the opposite approach: one human reviewer, no AI in the review process, no judgment — just structured feedback.

[[Aviator Verify]] replaces code review with intent-based verification (did the agent do what was agreed?). Subspace doesn't verify intent; it collects human feedback that a workflow might later consume. They're complementary layers in a verification stack.

[[cmux]] is one of Subspace's supported terminals — a macOS-native terminal built on libghostty designed for managing multiple AI coding agent sessions. Subspace integrates with cmux's `new-surface`/`respawn-pane`/`close-surface`/`focus-panel` API, and cmux is the only terminal entry that provides explicit post-review surface cleanup.

---

*Tags: #tool #project #agents #review*
*Sources: [[raw/subspace-beta]]*
*Last updated: 2026-07-25*
