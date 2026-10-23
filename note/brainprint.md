# brainprint — A Decision Brain Mined from Your Agent Session History

brainprint turns the one thing every coding agent already produces — the transcript of you saying yes, no, and "not like that" — into a reusable artifact. It reads your Claude Code, Codex and Pi session files, pairs each agent turn with your reply, scrubs secrets, and has a cheap miner model distill thousands of exchanges into a "decision brain": a Markdown document of how you decide, rendered at three lengths. The one-page advisory is injected into every new session via hooks, and when an agent hits a real choice it appends one line — "🧠 Dana's brain recommends: wait for review, then run it on staging first (confidence: high)". A `/decide` command reads the full 8,000-word brain for a longer prediction. What separates it from every other "agent memory" tool is that it *measures itself*: a held-out brain built from pre-cutoff history is scored against the decisions you actually made in the last three weeks, and you get an accuracy number next to chance before you lean on it.

---

## Architecture

A single `brainprint` Python entry point dispatches `init | score | refresh` into `pipeline/` (~1,300 lines total), with four stages that each cache their output so a failed run resumes where it stopped:

- **Read** (`pipeline/sessions.py`). Per-platform readers over three session stores — `~/.claude/projects`, `~/.codex/sessions`, `~/.pi/agent/sessions` — normalizing different JSONL shapes (Claude/Pi message records vs Codex `response_item` payloads) into `(role, text, ts, cwd)`. Filtering is careful: `isMeta`, `isCompactSummary`, sidechains, tool results and `<system-reminder>` bodies are excluded so only genuinely human replies pair with agent turns; terse replies like "1" or "A" still count. Pairs are deduplicated by `(ts, reply-prefix)`, capped at 20,000, and written atomically to `pairs.jsonl`.
- **Mine** (`pipeline/mine.py`). Pairs are sliced into 100-pair chunks; a miner model writes one-line decision notes per slice ("what they approve, reject, redirect... with a short quote as evidence"). Notes are merge-reduced in groups of 20 until one remains, then synthesized into `full.md` (≤8,000 words), three candidate `page-N.md` files (≤700 words), and a paragraph version. A parallel track mines only the exchanges where an agent *put a choice to you*, producing a "How you answer choices" section. Slices cache under content hashes shared between the held-out and installed runs, so each slice is mined once.
- **Score** (`pipeline/score.py`, `pipeline/decisions.py`). The post-cutoff pairs are screened by regex (`looks_like_decision_yield` — a decision question in the closing sentence plus second-person pronoun — or ≥2 numbered options), and a scorer model extracts options, your pick, and what the agent recommended. The held-out brain predicts each pick; accuracy is reported only above 30 decisions.
- **Install** (`pipeline/render.py`, `install.sh`, `plugins/brainprint/`). A plugin template is filled in with the owner's name and rendered page, written to `<brain_dir>/plugin/`, and registered as a local marketplace for Claude Code and Codex. A SessionStart hook (`advisory_hook.py`) injects the advisory; a TypeScript extension does the same for Pi via `before_agent_start`; `/decide` is a skill that reads `brain/full.md`.

All model calls run through the user's own agent CLIs (`claude -p --tools ""`, `pi -p --no-session`) in a throwaway temp directory with `BRAINPRINT_ADVISORY=0` set, so the tool never needs an API key of its own and can't recurse on itself.

## Key techniques

- **Held-out self-evaluation with a two-half page split.** The miner writes *three* candidate advisory pages; a deterministic hash split assigns half the recent decisions to choose the best page and the other half to report its score, so selection cannot inflate the metric. This is proper ML hygiene applied to a personalization artifact — almost nobody does this.
- **The combined arm.** Scoring compares three predictors: brain alone, agent's own recommendation alone, and *combined* (take the agent's recommendation when it made one, the brain otherwise). The author's comment is candid: "the brain second-guessing a recommendation lost on the author's history, so that is the headline." That's an empirical design decision, not a hunch.
- **Scrub before truncate.** Secrets are regex-redacted across the *full* text before the 1,500/1,000-character clips, "so a key on the boundary cannot leave a tail" — a subtle ordering bug most scrapers get wrong. The secret regex covers OpenAI/Anthropic/AWS/GitHub/Slack/Stripe/JWT/bearer/private-key patterns plus `key=`, `token:`-style assignment forms, with dedicated tests (`tests/test_no_pii.py`).
- **Content-hash caching everywhere.** Note files are named by a SHA-256 hash of their slice text; output directories record their input notes in `manifest.json` and are invalidated only when that set changes. Reruns are idempotent and resume from the exact failed slice.
- **Advisory-stripping in the scorer.** `strip_advisory()` removes any "brain recommends" line plus its "Why:" line from agent turns before they reach the scorer, so an advisory you already run can't leak into the ground truth it's being scored against.
- **Prompt-engineered non-permission.** The choices-section prompt explicitly forbids quoting any reply approving deploys, commits to main, or deletions: "this section is advice and must never read as permission." Both deliverables repeat it.

## Design decisions

The whole system trades freshness for honesty: the score you see is always for a brain 21 days older than the one you use, because only a pre-cutoff brain can be tested on decisions it never saw. The installed brain is chosen on held-out decisions it technically already saw (used only to pick among candidate pages, not to report accuracy) — an honest disclosure of what that gate can and can't prove. Mining targets a cheap model (Pi's default or Haiku) while scoring uses Sonnet, reflecting that distillation is bulk work and judgment is not. Configuration is deliberately tiny — two commands and a scope list — with the model-facing surface pushed into `brainprint.config.json` so users can point mining at any provider. The scope list is enforced as a path boundary (`is_relative_to`), not a prefix, in both the Python hook and the TS extension, avoiding the classic `/proj` matches `/project-x` bug.

Weaknesses worth naming: the decision-yield trigger is English-regex heuristics ("should i", "ok to proceed?"), so non-English sessions and unusual phrasings silently shrink the evaluation set; and the whole thing lives or dies on the premise that session replies are a faithful record of judgment rather than mood.

## Comparison notes

This is the rare [[Agent Memory and Context]] project that answers the question the memory literature usually skips: *measured against what?* Most systems ([[Devin Memory and Dreaming]], [[Giving Claude Agent Memory in 12 Steps]], [[OzBrain]]) are judged by vibes; brainprint refuses to install anything the held-out numbers don't support, and its three-page/halves protocol is a nice pattern the field could steal.

Against [[Guardian Angels]] — Gwern's vision of a continually-learning digital twin — brainprint is a deliberately shrunken, epistemically careful slice: not a personality emulator but a single narrow model (your decisions under agent-offered choices), with self-measurement instead of continual learning, and strictly local storage instead of a hosted twin.

Compared to [[Claude Memory Extractor Research]]'s finding that extraction agents converge with dangerous overconfidence on ambiguous cases, brainprint's response is structural: confidence is *scored* against held-out picks, and `/decide` is instructed to say "the brain doesn't have a clear read here — ask <owner>" rather than manufacture certainty.

And it complements [[Optimizing for Decision Points]]: Simister argues the leverage is in surfacing the human's taste-sensitive moments; brainprint actually captures what happens at those moments and feeds the answer back, turning the decision points into training data for the next session's advisory. It is also the most personal end of the [[Agent Memory and Context]] spectrum — a one-user artifact mined from raw transcripts, closer in spirit to the markdown-as-memory camp than to the graph-based systems like [[Context Graphs]].

#tool #project #agents #memory #personalization

---
*Sources: [[raw/brainprint]], [[summary/brainprint]]*
*Last updated: 2026-10-24*
