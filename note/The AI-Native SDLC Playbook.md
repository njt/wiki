# The AI-Native SDLC Playbook

Anthropic's Applied AI team lays out a full methodology for rebuilding the software development lifecycle around agentic coding. The central claim: the SDLC was engineered for an era when writing code was the bottleneck, and now that agents have collapsed that cost, the bottleneck — and the controls — have moved to the stages around the build. The answer is not more AI, but a reworked process where every stage commits a version-controlled artifact (`intent.md` → `spec.md` → `plan.md` → diff/tests → PR findings → incident record) that the next stage reads, and where governance is enforced by deterministic hooks rather than human line-by-line review.

---

## Key Quotes

> "The traditional SDLC was designed to maximize efficiency in an era where the most time-consuming and expensive stage was writing and implementing code, which is no longer the case."

This is the load-bearing premise, and it is correct — but it is also the argument for *process* change, not *tool* change. Where most "AI-native development" writing stops at "use Claude Code," this document insists the controls themselves have to be rebuilt. It shares that diagnosis with [[The New Software Lifecycle]] and [[AI-Driven Development Life Cycle]], but goes furthest on the governance half.

> "Every stage commits an artifact the next stage can read. Together, the intent, the spec, the plan, the diff and the review findings are the audit trail."

The artifact chain is the playbook's real contribution. It is a generalization of [[Planning With Files]] and [[recursive-mode]] — not just "plan before code" but *every* phase writing a file the next phase consumes, so the whole process is auditable from git history alone. The .md files in early stages exist because a product owner and an agent can both act on the same file.

> "Human attention concentrates at the gates, reviewing what the agent flagged rather than starting each stage from scratch."

A cleaner statement of the human-in-the-loop shift than most. The human role becomes a series of decision points (accept the intent, accept the spec, approve the PR, authorize the release) rather than a series of authorship tasks. This is the same "human checkpoints, not human review" idea in [[AI-Driven Development Life Cycle]], expressed as engineering practice rather than consulting terminology.

> "A skill is a control, though an advisory one. … The skill makes violations rare and the hook makes them close to impossible."

The single most important distinction in the document, and the one the rest of the wiki has been circling. It formalizes what [[Guardrails and Feedback Loops]] argues from first principles: instructions in model context are suggestions; deterministic enforcement is a constraint. Skills encode policy into the agent's behavior; hooks are the deterministic backstop that fires on every matching action.

> "The agent may act up to the production gate and cannot pass it."

The deployment boundary in one line. Autonomy is tiered by environment — free in dev, gated in staging, human-authorized in production — and the gate is a hook, not a policy document. This is the clearest public articulation of how to reconcile "agent ships everything" with "regulated enterprise requires sign-off."

> "The loop keeps running. Human judgement stays above it."

The closing line, and the playbook's answer to the autonomy question. Stage 6 (Maintain) runs headless — a deterministic script detects a breached control band, invokes Claude, and the finding re-enters as a new `intent.md` — but the loop is bounded by confidence gates between stages and human triage of the queue.

---

## Key Themes

- **#concept The committed artifact as audit trail.** Each stage writes a file the next reads, so the chain of commits *is* the record: who asked, what was produced, who approved. For early stages, .md files win because both a product owner and an agent can read them.

- **#pattern Skills (advisory) vs. hooks (deterministic).** The control stack has two layers. A skill makes Claude *likely* to apply policy while code is written; a hook *forces* it. "The skill makes violations rare and the hook makes them close to impossible." This is the same layered-enforcement model as [[Guardrails and Feedback Loops]] and [[Steering Claude Code]].

- **#concept The bottleneck moves left and right of build.** Plan, review/test, and deploy now run at human speed while build runs at agent speed — so those are the stages that need redesign. Security teams "sized for human output" are the canonical example: either the review queue builds or code ships under-reviewed.

- **#tool Claude Code as the substrate.** The whole playbook is, unabashedly, Claude Code's feature set — `intent.md`/`spec.md`/`plan.md` files, CLAUDE.md, skills, hooks, managed settings, evals in CI, `claude -p` in pipelines, Claude Tag. It is simultaneously the most thorough engineering methodology Anthropic has published and a product tour.

- **#pattern Human-at-the-gates.** Humans stop authoring and start deciding. The product owner reviews (but doesn't write) the spec; the engineer accepts the plan before code exists; the code owner approves the PR; the release manager authorizes the deploy.

- **#pattern Closing the loop.** Maintenance becomes the *start* of the next cycle rather than the end of the last one. A deterministic monitor breaches a band, Claude diagnoses, and the outcome is a new `intent.md` — no human in the invocation path, but a human in the triage queue.

---

## Critical Analysis

**What's genuinely valuable.** The artifact chain (`intent.md` → `spec.md` → `plan.md`) is the most portable idea here and does not require Claude at all — it is a disciplined [[Spec-Driven Development]] pipeline with a governance spine. The skills-versus-hooks distinction is the clearest public statement of a principle the wiki has been assembling piecemeal. And the deployment-boundary rule ("act up to the gate, not past it") is the most concrete answer yet to the enterprise objection that agentic coding cannot pass audit. This is [[The New Software Lifecycle]]'s map turned into an operating manual.

**What's suspicious.** It is, first, a vendor document. Every play terminates in a Claude Code feature — skills, hooks, managed settings, Claude Tag, MCP, the Agent SDK. That does not make the ideas wrong, but it explains the ambition-to-evidence ratio: the measurement sections are entirely *leading/lagging indicator* pairs with no published numbers from a real adopting organization. Compare [[Orchestrating AI Code Review at Scale]], where Cloudflare publishes actual cost and volume figures. Anthropic asserts; it does not yet demonstrate.

**What it understates.** The entire apparatus assumes a platform team that writes skills, hooks, evals, and managed settings *before* value arrives. That is a real and hard-to-fund investment — the exact gap [[The New Software Lifecycle]] flags as cultural rather than technical. And there is an unresolved tension at the heart: Build advocates "auto-accept becomes the default for routine work" while the closing line insists "human judgement stays above it." The playbook reconciles these only by trusting the guardrails (spec, blast radius, tests) to have been built *first* — which is [[Compound Engineering]]'s bet, stated as an assumption rather than a proven outcome.

**Where it fits in the wiki.** This is the missing middle the wiki has been building toward. [[AI-Driven Development Life Cycle]] is the AWS consulting version (same claim, thinner governance, more theology); [[The Claude Code Playbook]] is the shallow beginner version; [[Agent Coding Workflow]] is the practitioner-level hub. This document is the enterprise-governance version — the first source that treats the *process*, not the tool, as the thing that has to change, and backs it with a control model (skills/hooks/evals/gates) that survives a regulator's stare.

---

## Cross-References

- [[AI-Driven Development Life Cycle]] — AWS's competing full-SDLC methodology; same "AI at the center" claim, but Anthropic's version is concrete where AWS's is theological, and governance-first where AWS's is phase-first
- [[The New Software Lifecycle]] — Osmani's map of the uneven compression this playbook operationalizes stage by stage
- [[The Claude Code Playbook]] — the beginner-to-intermediate counterpart; this document is its enterprise-grade answer
- [[Agent Coding Workflow]] — the synthesis hub where this slots in as the governance/process layer
- [[Guardrails and Feedback Loops]] — the first-principles case for deterministic enforcement that the skills-vs-hooks split formalizes
- [[Specifications as the Product]] — the artifact chain is spec-as-product applied across the whole lifecycle, not just the build
- [[Planning With Files]] — plan-first work as a session pattern; this generalizes it to a full lifecycle
- [[Steering Claude Code]] — Anthropic's taxonomy of the instruction-delivery mechanisms (CLAUDE.md, skills, hooks) this playbook assembles into a process
- [[Claude Code Skills System]] — the skills architecture that carries the "advisory control" layer
- [[Agentic Code Review]] — the review-loop plays (Claude reviews and addresses comments) in enterprise form
- [[Demystifying Evals for AI Agents]] — the evals-as-QA stage's underlying discipline
- [[Introducing Claude Tag]] — the channel-first incident intake that feeds Stage 6's loop

---

*Sources: [[raw/the-ai-native-sdlc-playbook]], [[summary/the-ai-native-sdlc-playbook]]*
*Last updated: 2026-08-22*
