# SIS Factory — The 3D Self-Improving Software Factory

SIS (github.com/sis-factory/sis-factory, MIT) is an open-source, locally-runnable software factory: a Jira ticket labelled `sis` becomes a draft pull request, with every review comment and CI failure fed back into a measured, evidence-gated "lessons" engine. What makes it worth reading is not the 3D factory-floor visualisation (React Three Fiber robots you can click to interrogate) but that it is the first factory in this wiki where the self-improvement loop is run as an actual controlled experiment rather than an aspiration.

---

## Architecture

~17 small Python services under `components/`, one shared pure-Rules library under `libs/`, ~10.5k lines of Python plus a Next.js portal:

- **libs/contracts** — pure, I/O-free rule modules (lessons, policy, replay, handling, posture). Everything contentious — who may approve a lesson, how a replay is scored, how a reply is classified — lives here as testable functions. The `policy.py` docstring states the design rule outright: "No I/O, so they are easy to test."
- **components/orchestrator-worker** — one Temporal `TaskWorkflow` per ticket (`workflow.py`). The workflow is deterministic by construction: it only waits, signals and calls activities; anything touching a network lives in `activities.py`. Agent stages (planner → intent.md → architect design.md → developer) each run in their own sandbox, sequentially, with human approval gates on documents.
- **components/jira-connector** — the only holder of Jira credentials. Everything else talks to Jira through it.
- **components/agent-runner** — defines an `Adapter` protocol (`adapters.py`) so Claude Code, an open-source agent, or the scripted demo agent are interchangeable. The sandbox's *only* way out is `gateway.py`: a `Gateway` client holding a per-task token and the gateway address, nothing else.
- **components/sandbox-gateway** — egress broker: holds the model API keys (the sandbox has none), enforces per-task budgets with a human "approve/stop" pause, checks outbound text, serves the task archive.
- **components/git-broker** — GitHub App identity only: per-repo short-lived installation tokens with a minimal permission set (`ghapp.py` documents that admin/workflows/secrets are "deliberately absent"); branch protection, not the app, is what prevents merges.
- **components/injection-guard**, **sandbox-provisioner** (Podman/K8s, gVisor-ready), **context-api/ingest**, **embeddings**, **portal-api/portal-web/3d-view**, **network-policy** (default-deny + allow-gateway NetworkPolicies), **auth-proxy**.

## Key techniques

- **Lessons as a trusted/untrusted boundary.** `libs/contracts/lessons.py` is a small masterpiece of paranoia rendered as types. Lessons are "the one place where text is allowed to act as an instruction", so `clean_text()` strips markup that could pose as structure, `decide()` requires a human comment from a role-scoped author (`_AUTO` maps scope→minimum role: global lessons need an admin), and machine-sourced lessons (agent suggestions, mined review remarks) are always held to candidate status. The agent can propose but never approve.
- **Deterministic holdout groups.** `held_out()` uses `sha256(task_id:lesson_id) % 100 < pct` — deterministic per (task, lesson) so every context pack for the same task agrees, and independent across lessons. Human-taught lessons are always delivered; only machine-proposed ones are A/B tested.
- **Statistical retirement, not vibes.** `improve.py:lesson_effects()` compares task quality (1.0 merged as delivered, 0.5 merged after revisions/fixups, 0 not merged) between applied and held-out groups, requiring ≥10 tasks per side, a minimum delta, *and* a 1.64-SE one-sided bar "so a lucky streak does not retire a good rule."
- **Replay scoring as a proxy.** `replay.py` scores a re-run against the ticket's *merged* diff — including review revisions and human fixups — 40% file Jaccard + 60% line F1, and is honest in its docstring that similarity is a comparison-arm proxy, not a correctness verdict.
- **Mining rules from noise.** `learn.py:mine_review_lessons()` clusters imperative-sentence review remarks across PRs by Jaccard similarity (≥0.55, ≥2 distinct PRs), keeping the most concise phrasing; recurring CI failures become "run this check before you finish" lessons; only reviewers with OWNER/MEMBER/COLLABORATOR association count, bots never do.
- **Defence-in-depth on text.** Trust profiles per source (`jira_ticket` = untrusted, `repo_file` = semi-trusted, unknown = strictest), obfuscation variants (rot13 etc.) that *add* weight because hiding is itself suspicious, canary tokens in system prompts checked on every outbound surface, secret-pattern egress scanning, Luhn/IBAN-validated PII redaction before indexing.
- **Dry by default.** The demo agent (`ScriptedAdapter`) calls no model; the whole stack runs against mock Jira/GitHub in e2e tests before any key is spent.

## Design decisions

The dominant trade-off is **measurability over cleverness**: the self-improvement loop is deliberately slow, small (≤ a token budget of lessons, most-specific-first, keyword-overlap ranked) and statistically conservative, rather than embedding fine-tuning or RAG-over-everything. Pure functions hold all policy; I/O lives at the edges — this sacrifices integration elegance for auditability. Trust is structural, not behavioural: the sandbox cannot get a key, cannot reach the network except through the gateway, and cannot approve its own lessons, so no prompt can escalate it. The honest weakness: quality is proxied by merge outcome, so a lesson that improves code but attracts nit-picky reviewers reads as "hurting" — the same critique that applies to any outcome-metric factory loop.

## Comparison notes

- Unlike [[Self-Improving Software Factories]] (Warp's vision piece), SIS ships the measurement loop as code — holdout groups and z-tests, not a roadmap slide. It is the concrete counterpoint to [[Harness Engineering for Self-Improvement]]'s survey of proposal-only loops.
- Where [[Self-Hosted Sandboxed Agentic Software Factory]] is a one-person homelab stack, SIS aims at the enterprise end: multi-team, encrypted secrets, posture checks that refuse unsafe production settings — yet still runs fully local, a stance shared with [[TEHAS — A Pi Software Factory]] but scaled up from one Pi to a Temporal fleet.
- Its five-stage gate architecture (intake → isolation → tools → verification → merge) matches [[How to Build an AI Software Factory]]'s synthesis almost exactly, but adds the stage that synthesis lacks: a feedback engine that *retires* its own rules on evidence.
- The injection-guard's wrap-as-untrusted + canary + outbound-scan sandwich is a working implementation of the threat model in [[Zero Trust for AI Agents]], notably treating *unknown sources as strictest* and treating obfuscation attempts as signal rather than noise.

---
*Sources: [[raw/sis-factory]], [[summary/sis-factory]]*
*Last updated: 2026-11-09*
