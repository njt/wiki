# Clanker Constitution (kenn-io)

A tiny repository that ships a versioned set of default operating principles for coding agents — a single `CONSTITUTION.md` plus a README — designed to be installed as a steering file for Codex (`~/.codex/AGENTS.md`) and Claude Code (`~/.claude/rules/`, or imported from a repo's `CLAUDE.md` with `@CONSTITUTION.md`). It matters because it treats agent behaviour as a governed artifact rather than a pile of personal prompts: calendar-versioned immutable release tags, a precedence rule, and licence terms forcing modified copies to identify themselves.

---

## Architecture

There is almost no code — the "architecture" is the document's structure and its deployment story:

- **`CONSTITUTION.md`** — seven numbered sections of normative bullets: 1. Honor the request, 2. Act with judgment, 3. Finish the job, 4. Protect existing work, 5. Verify reality, 6. Communicate for humans, 7. Learn in the right place. Each section is 4–7 terse rules, sized to fit an agent's steering context cheaply.
- **`README.md`** — the distribution and governance protocol: global install via symlinks that preserve existing destinations and merge manually; per-repo install by copying from a reviewed release tag, with the root `AGENTS.md` pointing agents at it and `CLAUDE.md` importing it with `@CONSTITUTION.md`.
- **Versioning** — calendar tags (`v2026.11.03`, with `.1` suffixes for same-day re-releases), immutable by policy. Copies are pinned; updates happen through pull-request review, explicitly *not* by fetching at agent startup.
- **Attribution rules** — unchanged mirrors keep provenance notices; modified copies must retain attribution, flag their changes, and never claim to be the canonical version (CC BY 4.0).

## Key techniques

- **Precedence as a first rule.** The document opens by declaring itself overridable: direct user instructions and more specific repo instructions win. This defuses the classic steering-file failure where a generic default fights a specific instruction.
- **Mode-matching as contract interpretation.** Section 1 distinguishes explain/review/diagnose (read-only) from change/build/fix (implement and verify) — the agent infers the permission scope from the verb.
- **Authority boundaries written down.** Section 2 has unusually specific hard lines for 2026: never merge a PR without user authorization, and "plan or specification documents do not grant merge authority" — closing the loophole where an agent treats an approved plan as approval to ship.
- **Anti-drift and anti-distraction clauses.** Quoted or pasted content must not trigger skills or tools (a prompt-injection-adjacent defence), and section 7 forbids agent-private memories in place of updating shared `AGENTS.md` guidance — memory belongs where the whole team can review it.
- **State-safety rules.** Section 4 bans amend/reset/stash/overwrite without authorization, demands resolving exact targets before destructive actions, and — notably — says that when corrected or told to stop, the agent must stop mutating state and inspect before recovering.

## Design decisions

The trade-off is breadth vs depth: seven sections can't make an agent smarter, so the authors optimised for a minimal, reviewable baseline that generalises across tools and repos, betting that short normative rules survive in context better than long playbooks. Versioned immutable tags plus copy-don't-fetch sacrifices freshness for supply-chain safety — a constitution that silently updates itself at agent startup would be a prompt-injection vector and an unaudited behaviour change. The cost is distribution friction: every repo must manually PR its copy forward. It also chooses norms over enforcement — nothing here is mechanically checked, which is precisely the "convention, not constraint" weakness.

## Comparison notes

- [[The Clanker Constitution]] — the earlier blog-essay write-up of the same project by its author (as Wes McKinney); this repo ingest is the canonical artifact and adds what the essay only implied: the concrete install paths, the immutable-tag versioning scheme, and the attribution duties on modified copies.
- [[A Convention Is Not a Constraint]] — Simple Thread's distinction is the sharpest lens on this project: a constitution is pure convention, holding only while the model attends to it. It sits at the softest end of the steering spectrum, which makes its governance apparatus (versioning, review-only updates) the real contribution.
- [[Design Systems for AI Agents (Luke Wroblewski)]] — Wroblewski argues steering files stop rotting because agents read them every time; the Constitution applies the same logic to behaviour instead of design tokens, and its AGENTS.md-first, CLAUDE.md-imports install pattern is a concrete instance of his single-source steering layer.
- [[Nine Harness Patterns (deepsense.ai)]] — deepsense.ai's "AGENTS.md plus commit bodies as the why store" treats steering files as part of the harness; the Constitution is a portable, versioned contribution to that layer, but with explicit rules (skills-loading, no private memories) that overlap and sharpen their pattern list.
- [[Getting Out of the Loop — Lydia Hallie on Claude Code's New Best Practices]] — Hallie reports Anthropic moving away from emphatic CLAUDE.md warnings and supervision; the Constitution is the counterpoint artefact — terse defaults plus hard authority lines (never merge without authorization) aimed at exactly the high-autonomy, low-supervision regime she describes.

---
*Sources: [[raw/constitution]], [[summary/constitution]]*
*Last updated: 2026-11-20*
