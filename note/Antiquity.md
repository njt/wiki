# Antiquity — an Opinionated Workspace for AI Historical Investigation

Antiquity (Jesse Waites) is a Go CLI that scaffolds a workspace for hunting through historical sources with AI agents. It ships no pipeline: it generates folders, an `AGENTS.md` rulebook with a seven-step research loop, model-role key config, and a secret-blocking git hook, then hands the case to your coding agent. It codifies the process from a real investigation that found a lost meteorite in digitised colonial archives — "we built the plane as we flew it", and this is the post-flight rewrite.

---

## Architecture

Tiny tool (~1,700 lines of Go 1.27, Bubble Tea v2 + Bubbles + Lip Gloss), three packages:

- `main.go` — two subcommands: `new` (wizard → scaffold) and `intro` (animated title screen, with a `--snapshot --at MS` headless frame renderer for tests/CI).
- `internal/scaffold` — the substance. An embedded `templates/*.tmpl` FS (`//go:embed`), a `Case` struct, and `Write()` which renders ten documents (AGENTS.md, CASE.md, HANDOFF.md, NARRATIVE.md, lessons.md, keys files, three templates) plus thirteen folders, each seeded with a README.md whose text is a hardcoded doc comment in `Folders` (`scaffold.go`). `initGit` runs `git init -q` and writes a `pre-commit` hook; git is silently skipped if absent.
- `internal/wizard` — a five-step Bubble Tea wizard (name, question, description, API keys, confirm) with ctrl+s skip-everything; `internal/intro` — a hand-rolled 40ms-frame title animation.

The `--yes` non-interactive mode is designed for the agent itself to call the CLI: only the name is required, and the generated AGENTS.md instructs the agent to interview the human for the missing question and scope ("that *is* doc item #1").

## Key techniques

- **The seven-step funnel as prose, not code.** Question + known-answer control → wide retrieval (regex + embeddings) → cheap judge over every candidate → expensive reader over survivors → verify on the original page image → novelty check against catalogues → evidence and write-up. The AGENTS.md template states "a judge pre-sort is ~50× cheaper" and forbids sending the whole candidate list to the reader.
- **Provider-neutral judge questions.** `templates/judge.md.tmpl` requires yes/no, `choice{a,b,c}`, or `score[0-10]` forms "compiled to the judge provider's API at run time" — the model roles in `keys.yml` (judge: claude-haiku, reader: claude-sonnet, embedder: Qwen3-Embedding-0.6B locally, vision) are swappable per role.
- **Secrets defence at scaffold time.** The generated pre-commit hook (`templates/pre-commit.tmpl`) greps staged diffs for `sk-ant-…`, `sk-…`, `ts_…`, `AKIA…` patterns and refuses `keys.yml` outright; keys are written chmod 0600 and gitignored (`scaffold.go` `render("keys.yml.tmpl", …, 0o600)`).
- **Epistemology encoded as folder discipline.** `candidates/` rows carry status new→sorted→read→verified/rejected; instructive rejects get `REJECTED_` prefix evidence images; `controls/` holds known events the pipeline must re-find, with per-run recall — "a negative result without a control is not a result". `lessons.md` is a trap table fed back into `queries/` and `judges/`.
- **Cost as a first-class field.** `keys.yml` carries `budget: per_run_usd: 5, per_day_usd: 25`; `runs/` logs counts at each funnel stage plus cost per model role, and the agent must ask before exceeding budget.

## Design decisions

The central trade-off: **scaffold everything, run nothing.** Instead of building a brittle pipeline framework, Antiquity treats the agent as the runtime and spends its ~1,700 lines on conventions — "prefer a small script you can drop into when a source misbehaves over a framework". The cost is that every guarantee (recall checks, image verification, budget caps) is a rule an agent may violate, not code that enforces it. It also front-loads failure lore: dateline errors, word traps, storms reported as earthquakes, silent truncation from thinking tokens — hard-won specifics most agent templates lack. The human retains ownership of scope, budget, and anything that leaves the machine; agents may never send outreach (`drafts/` is plain text only).

## Comparison notes

- [[AI Against 400 Years of Archives]] — the author's write-up of running this scaffold over 4.35 million archive pages: the seat assignment (cheap triage model, frontier model on the survivors) that this repo encodes as conventions.
- This is the domain-research sibling of [[The Dark Factory is a DOT File]]: both argue the durable artifact is the convention file, not pipeline code — but Antiquity's AGENTS.md is an epistemology (controls, novelty, evidence) rather than a workflow graph.
- Its "cheap judge, expensive reader, human-on-scan-last" funnel is a concrete instance of [[Deterministic When Possible, Probabilistic When Necessary, Human When Cheaper]], with the page-image rule standing in for the deterministic tier.
- Where [[LLM-as-a-Verifier]] treats verification as a model pass, Antiquity goes further: model output is never verification — the original scan is.
- Unlike [[The Archaeologist's Copilot]], whose "archaeologist" reads old *code*, Antiquity's archaeology is literal: same containment-and-conventions instincts applied to archives instead of codebases.

---
*Sources: [[raw/antiquity]], [[summary/antiquity]]*
*Last updated: 2026-10-10*
