# Swarm Skill

jleechanorg's Claude Code `/swarm` skill — a battle-tested playbook for orchestrating multi-agent swarms distilled from ~180 agents and ~7M subagent tokens across five July 2026 workflows. It is the most operationally detailed public document on running large Claude Code Workflow-tool fan-outs in production: 14 hard rules each learned from a concrete failure, a mandatory sidekick durability layer, canonical phase shapes, and a publishability gate that catches what adversarial verification misses.

---

## Key Quotes

> "Cost-route every `agent()` call explicitly — verify with a grep, not a memory check."

The first hard rule, and the one that sets the tone for the whole document. A 40-agent fan-out on a Fable session silently ran every agent on Fable for weeks because nobody grepped the script for bare `agent()` calls. This isn't a theoretical concern — it's a $1,000+ mistake that happened and was caught by a token audit. The meta-point is deeper: **workflow scripts need mechanical pre-flight checks, not human eyeballs.** A grep is deterministic; an "I looked at it" is not.

> "A run that returns 0 confirmed findings is a VOID, not a verdict, if its `failures[]` list shows mass agent death."

Rule 3, and the most important statistical hygiene lesson in the document. Two separate workflows (15/15 and 60/60 verify agents dead) produced "0 findings" that were actually 25 real findings the dead verifiers couldn't process. Distinguishing "nothing to find" from "nobody was alive to find it" is the difference between evidence and noise. This generalizes beyond swarm workflows: any agent pipeline that reports emptiness must also report agent health.

> "A same-model swarm shares one set of blind spots."

Rule 12, and the most damning empirical finding. ~180 Claude agents unanimously passed a docset; a single codex adversarial pass found six real defect classes. The same thing happened again five days later on a production bug fix. Model diversity isn't a nice-to-have — it's the only thing that catches what intra-model redundancy structurally cannot.

> "Bursts kill, ramps don't."

Rule 4's distillation from three separate 429 storms in one evening. Every kill coincided with an instant N-wide `parallel()` fan-out; gradual ramps and `pipeline()` were never throttled. The ceiling is ~16 simultaneous cold starts — not a target, but a survivable upper bound. Counter-intuitively, serializing big stages *across* sibling swarms matters as much as within one, because provider rate limits are aggregate across all concurrent workflows on the account.

> "Under-utilization is a failure mode equal to collision."

Rule 14, from a session where five disjoint evidence-prep items sat idle for an hour waiting on one blocked gate. The instinct to freeze everything when one thing is stuck is a planning-horizon error: blocked gates block only their dependents, never sibling work.

---

## Key Themes

### The Sidekick Durability Layer (#pattern)

The sidekick is a **real Claude Code process in a real tmux session** — not a subagent, not a SendMessage-addressable teammate, but an independent process that survives the parent CLI exiting. It owns STATE.md, fans out sub-agents, and provides crash recovery: `/sidekick [model]` in a fresh session resumes from disk+git checkpoints with zero context re-derivation. The default mode (since 2026-07-11) runs lanes as in-session Agent Team teammates visible in the user's panel, with the sidekick as durability keeper and crash respawner.

This is the [[Devin Fusion sidekick pattern|https://cognition.com/blog/devin-fusion]] applied to Claude Code — the orchestrator is durable, the workers are ephemeral. It's a more opinionated and production-hardened version of the pattern behind [[Fleet Supervisor (sermakarevich)]], which uses a single async Python process rather than a tmux Claude Code session.

### Adversarial Verification as Default (#pattern)

Three independent lenses prompted to refute, ≥2/3 to survive, dedup against `seen` not `confirmed`. This is adversarial verification as the default posture, not an optional pass. It's the same pattern that shows up in [[Cloudflare Security Audit Skill]] and [[Load-Bearing Assumptions]], but `/swarm` operationalizes it across five canonical phase shapes with specific agent counts and confirmed-yield rates (7/10 for design-retro, for instance).

The critical addition: **verifier death is not refutation.** A dead verifier (rate limit, API error) that kills a finding is a false kill, not evidence. This is the kind of production scar tissue that only comes from running 180-agent workflows and watching 60/60 verifiers die.

### Cost Routing as Mechanical Discipline (#tool)

Every `agent()` call gets an explicit `model:` — haiku for mechanical stages, sonnet for miners/verifiers/doc writers, top-tier only for adversarial judgment with a stated reason. The pre-flight check is a literal `grep -n "agent(" <script>` — not a human eyeball. This is [[The Advisor Strategy]] inverted: instead of a single advisor on demand, it's tiered routing across an entire fan-out, with mechanical enforcement.

The lesson from the 2026-07-07 incident (428 MiniMax messages + 367 Fable messages in one session because an unrouted fan-out silently inherited Fable) is that **model routing in workflow scripts has the same failure mode as unquoted shell variables**: silent inheritance that looks correct until you audit the bill.

### The Publishability Gate (#pattern)

Adversarial verification attacks candidate *findings*, not the *rendered docs* that ship. A healthy 3-lens verify pipeline can still publish docs with leaked credential paths, stale metrics, forbidden recommendations, or contradicting sibling docs. The publishability gate is a final single-agent pass over the entire docset checking seven dimensions: redaction, cross-doc consistency, freshness re-baseline, supersession markers, policy lens, recipe validity, and mechanical hygiene.

This is the most underappreciated insight in the document. Verification and publication are different operations. A finding that survives adversarial scrutiny isn't the same as a document that's safe to publish. The gate found all seven defect classes in already-"confirmed" docs from a ~180-agent, 5-workflow swarm. That's a 100% failure rate on published artifacts that had survived adversarial verification.

### Cross-Model Cold Review (#pattern)

Rule 12 is the empirical case for model diversity as verification infrastructure, not a luxury. The evidence is stark: ~180 Claude agents unanimously passed a docset with six real defect classes; a codex review found five real blocking defects in a Claude-reviewed production fix. The mechanism matters: codex *executed the code and reproduced the bugs* rather than reading the diff and pattern-matching against the PR's framing.

This connects to [[Nicole Forsgren on AI and Developer Productivity]]'s point about agents that always agree with you — same-model swarms are a structural monoculture with shared blind spots, regardless of how many independent agents participate.

### Domain Generality (#concept)

The document explicitly claims nothing is coding-specific. The same shapes (mine → adversarially verify → synthesize → gate) run market research, incident retros, literature reviews, ops audits, planning exercises, content pipelines. This is a testable claim — and given the empirical detail in every other section, it's more credible than most "domain-agnostic" marketing. The invariants that survive cross-domain substitution: refute-by-default verification against primary sources, disjoint writer outputs, commit-early durability, false-empty detection, a whole-artifact publishability gate, and cross-model cold review.

---

## Critical Analysis

This is the most valuable single document I've read on running Claude Code at scale. It's not a whitepaper — it's an incident report disguised as a playbook. Every rule has a date, a concrete failure, and a dollar cost in wasted tokens or frozen lane-hours. The empirical density (14 rules from ~12 distinct incidents, all dated, all reproducible) makes it credible in a way that pattern-language documents rarely are.

**What's genuinely new here:**

The **false-empty detection** rule (rule 3) fills a gap in every agent orchestration framework I've seen. Nobody else distinguishes "nothing to find" from "nobody was alive to find it." The **publishability gate** (rule 11) names a category of defect that adversarial verification structurally cannot catch — verification attacks claims, not rendered artifacts. The **CI capacity ceiling** (rule 13) is a class of failure that's invisible if you only think about agent concurrency: a 68-agent PR-branch-update sweep that's trivially cheap on the Claude API side crashed a 16-runner CI host because each "cheap" agent triggered 5-30 downstream CI jobs.

**What's debatable:**

The **mandatory cross-model cold review** (rule 12) is the right idea but the operational cost is high — running a full codex adversarial pass on every merge-readiness claim isn't free. The document's answer ("what did the broken fix cost?") is fair but not complete. There's a missing economic model here: when is the cross-model pass worth it vs. when does the same-model swarm's verification suffice? The answer probably depends on the blast radius of being wrong, but the document doesn't provide that calculus.

The **sidekick durability layer** is clever but complex. Running a real tmux Claude Code process as a durability keeper adds operational surface area: tmux sessions, /tmp state files, br beads, branch-scoped STATE.md paths, checkpoint cadences with background timers. This is the right architecture for multi-hour unattended runs, but for a 30-minute attended swarm it may be over-engineered. The document itself seems to recognize this tension (the 2026-07-11 default mode shift to in-session teammates), but the tmux-sidekick complexity remains as the fallback path.

**What's missing:**

No discussion of **when NOT to use a swarm.** The document assumes the answer is always "run a swarm" — but for a single well-scoped question, a single `agent()` call may be the right tool. The closest the document comes is the fleet-triage shape (single fan-out, 19 agents, one per lane), but a triage of one item is just an agent call.

No treatment of **swarm economics per shape.** The document gives agent counts (42, 70, 28, 19) but no token costs or wall-clock times. Given the cost-routing discipline everywhere else, this is a surprising gap. A practitioner choosing between shapes needs to know the rough cost envelope.

No mention of **synthesis quality** as a distinct concern. Phase shapes end with "1 synthesis" or "1 plan-doc writer" — but the quality of synthesis is the most model-sensitive step in the pipeline, and the document's own cost-routing rules would put it on the main-loop model (Fable). A bad synthesis from a good Collect+Verify pipeline is still a bad output.

---

## Connections

- [[Agent Orchestration]] — The hub: `/swarm` is the most battle-hardened instantiation of the planner/worker/judge pattern, with concrete agent counts and failure modes
- [[Fleet Supervisor (sermakarevich)]] — The closest production parallel: a dumb supervisor claiming tasks from a beads queue and fanning out to pluggable coder backends. `/swarm`'s sidekick is a Claude Code-native alternative to Fleet's Python asyncio loops
- [[Loop Engineering]] — `/swarm` is loop engineering operationalized: the playbook is the harness design, the sidekick is the automation, the phase shapes are the sub-agent patterns
- [[Cloudflare Security Audit Skill]] — Shares the multi-phase parallel-agent pipeline architecture (recon → hunt → adversarial validation), but `/swarm` generalizes it beyond security to any verifiable domain
- [[StrongDM Factory Techniques]] — The "commit-early, commit-often" per-doc-writer pattern and pre-built dataset approach echo StrongDM's "dark factory" patterns
- [[The Advisor Strategy]] — `/swarm` inverts it: instead of one advisor, every agent in the fan-out gets an explicit model routing decision. Same principle (right model for right task), different scale
- [[Orchestrating AI Code Review at Scale]] — Cloudflare's production system with 7 specialized agents + coordinator judge is the closest enterprise parallel. `/swarm` is the practitioner's counterpart: smaller scale, harder-learned rules, open-source
- [[Load-Bearing Assumptions]] — The adversarial verification pattern (`finder → strategist → parallel validators`) is the same shape as `/swarm`'s Collect → Verify pipeline, but `/swarm` adds the publishability gate and cross-model review that LBA doesn't address
- [[Guardrails and Feedback Loops]] — Rule 12 (cross-model cold review) is the empirical proof of the hub page's thesis: verification must be independent of generation, and same-model independence is not independence
- [[Steering Claude Code]] — The sidekick as a Claude Code skill that wraps other Claude Code sessions is a meta-instruction pattern: the skill defines how to run skills at scale
- [[Agent Swarm Model Economics]] — Cursor's peer article from the same week: the five coordination failure modes (split-brain, contention, merge conflicts, megafiles, ossification) are the taxonomy that explains why Swarm Skill's 14 hard rules are necessary, and the SQLite experiment provides the cost data Swarm Skill's shape economics are missing

---
*Sources: [[raw/swarm-skill]]*
*Last updated: 2026-07-18*
