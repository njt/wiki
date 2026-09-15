---
url: https://github.com/dbmcco/agent-designer
title: "Agent Designer"
author: Braydon (dbmcco)
date_fetched: 2026-09-15
date_published: 2026-08-24
topics:
  - agent-orchestration
  - guardrails-and-feedback-loops
---

# Agent Designer

A Pi skill (v0.3.0, MIT) that designs and deploys **expert ensembles**: casts of LLM-generated domain-expert personas coordinated through a fixed, inherited methodology. It canonizes the craft behind eight hand-built persona panels in the author's `experiments/ai-simulations` — the skill extracts their shared orchestration spine into a versioned kernel and never touches the originals again.

Two protocols share one craft layer. The **panel protocol** builds a standing panel — moderator, 5–8 specialist seats, one permanent adversarial/audit seat, phase overlays — that lives at `panels/<slug>/` in the repo where the work happens, discovered by `find . -name panel.json` rather than any central registry. A calibration gate flips a manifest from `calibrating` to `active` only after the user confirms the goal and roster; nothing is drafted before that. The **runtime protocol** casts 2–5 experts live for one question, executes them through Pi's subagent tool (parallel isolated passes, then an adversarial pass, then synthesis), and leaves transcripts plus a promotable cast on disk — promotion absorbs the files, it never regenerates them.

The persona doctrine's theory is that **a persona is a retrieval key into the training distribution**: a domain-common name, an institutional email signature, a CV in the field's own vocabulary, a specific formative experience, declared proclivities and blind spots, textured communication style, and an expert query vocabulary each land the model in expert-text territory, while generic pointers keep it in generic-assistant space. A two-pass gate enforces this: `validate-persona.sh` checks structure (eight sections, ≥300 words), and a model-mediated voice checklist re-drafts anything that "reads like a job posting" or has "no vocabulary of its own".

The orchestration canon (`methodologies/kernel.md`, 11 rules, semver-versioned, currently 1.2.0) is inherited whole — overlays add phase work but may never amend it. Its distinctive moves: specialists work in isolation before convergence (anti-anchoring), disagreement is signal and every synthesis must carry a labeled `## Dissent` section or a script fails it, quality judgments are model-mediated while deterministic scripts check only structure, and every run closes with a contribution review where causal-lift claims require a one-seat counterfactual — otherwise only traceable contribution may be claimed. A 2026-09-09 self-critique session produced seven foundational challenges to the doctrine; the repo answered with a calibration addendum and an opt-in assignment-decomposition pilot overlay rather than silent edits.
