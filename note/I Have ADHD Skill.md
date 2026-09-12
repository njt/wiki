# I Have ADHD Skill

ayghri's output-shaping skill for coding assistants, distributed across ten-plus harnesses. It reshapes an AI agent's output for a reader with ADHD. It's not a brevity toggle — it's a complete communication protocol built from five facts about ADHD cognition: small working memory, the knowing-doing gap, cold-start paralysis, uniform time blindness, and dopamine scarcity. Ten concrete rules flow from those facts, each with explicit "bad/good" examples. The skill persists for the full session unless explicitly turned off.

---

## Key Quotes

> "Working memory is small. Anything not on screen is forgotten. Do not ask the reader to 'keep in mind X.'"

This is the foundational claim, and it's backed by the cognitive science that [[Engineering for Bounded Cognition]] explores in detail — working memory holds ~4 chunks, unrehearsed information decays in ~20 seconds. The skill operationalizes this as "restate state every turn" and "lead with the next action." It doesn't argue for shorter responses; it argues for responses where the reader never needs to remember what came before.

> "Knowing the answer is not doing the answer. The friction between 'got it' and 'done it' is where work dies."

The most practically important line in the skill. It diagnoses a failure mode that [[Smart But Scattered — Peg Dawson on Executive Skills]] calls "task initiation" — the gap between understanding and acting that kills more work than any technical obstacle. The fix is structural: lead with the action, end with one concrete next step, number multi-step tasks. These aren't stylistic preferences; they're bridges across the knowing-doing gap.

> "Forbidden openers: 'Great question,' 'Let me...', 'I'll...', 'Sure!'"

Rule 10 is the hardest to follow and the most controversial. It bans the conversational lubricant that most AI output styles are greased with. The reasoning is sound — every preamble word taxes working memory — but the execution requires constant vigilance. This is the same insight behind [[SimpleEnglish]]'s rule-based output constraints: deterministic rules beat pleading. The pre-send checklist (delete the first sentence if it announces what you're about to do, delete the last if it asks "anything else?") is a linter for agent output, not advice.

> "If the last three turns have been 'still broken,' stop iterating on code. Name the assumption that might be wrong. Ask one diagnostic question."

The debug-spiral escape clause. Three consecutive failures triggers a meta-cognitive intervention: stop doing, start doubting. This is the [[Guardrails and Feedback Loops]] principle applied to the conversation itself — when the loop isn't converging, the loop is the problem, not the code.

> "Vague estimates fail. Ballpark in concrete units."

A reframing of advice so basic it shouldn't need saying — except that most AI agents are terrible at it. "This will take some work" and "a few hours" register identically to an ADHD brain because time perception is binary: now and not now. Dawson's line about ADD time perception ("there are two times: now and not now") makes this rule feel less like a preference and more like an accessibility requirement.

## Key Themes

- **#pattern** — Output shaping as accessibility engineering. This isn't about being "considerate" to ADHD readers; it's about designing the communication channel to work with the cognitive instrument that will receive it. The same logic as [[Engineering for Bounded Cognition]]: design for the most constrained user, and everyone benefits.
- **#tool** — The skill as a session-persistent output style modifier. Unlike one-shot prompts, this skill changes every response until explicitly turned off. The persistence model ("if you are unsure whether they still apply, they do") is a deliberate design choice that prevents the skill from decaying across turns — a problem that plagues most output-style instructions.
- **#concept** — The knowing-doing gap as a communication design problem. The skill frames ADHD not as a deficit but as a set of constraints that reveal bad communication design. Responses that fail for an ADHD reader were probably failing for everyone else too; the ADHD brain just registers the failure faster.
- **#pattern** — Deterministic rules over stylistic advice. Every rule has a bad/good example pair. Every rule is falsifiable — you can check whether a response violates "lead with the next action" or "cap lists at 5 items." This is prompt engineering as specification, not suggestion.

## Critical Analysis

**What works.** The skill is a masterclass in negative specification — it defines what *not* to do more clearly than what to do. The pre-send checklist (delete five things, then verify first-line/last-line comprehension) is the most concrete and portable part. Any agent output style could adopt that checklist without adopting the full ADHD framing.

The five cognitive facts are the skill's intellectual engine. Most output-style prompts skip the "why" and jump straight to rules. By grounding each rule in a specific cognitive constraint, the skill makes the rules feel necessary rather than arbitrary — and gives the model a framework for deciding edge cases the rules don't cover.

The bad/good examples are unusually good. Most prompt-writing guides give examples that differ in multiple dimensions, making it impossible to tell which change mattered. This skill's examples are minimal pairs: same scenario, same information, different shape. The difference is always structural, never tonal.

**What's missing.** The skill assumes the reader *has* ADHD and *knows* they have it. There's no guidance for the much more common case: the reader doesn't identify as ADHD but would benefit from the same output shaping. The skill's name and framing make it an opt-in identity move rather than a general-purpose accessibility setting. A "cognitive accessibility mode" that made the same output changes without the diagnostic label would reach far more people.

The skill doesn't address the cost side. ADHD-shaped output is harder to write. It requires more editing passes (the pre-send checklist is work), more structural discipline, and more willingness to delete. A model following these rules will produce fewer words per response, which on per-token pricing is cheaper — but on a fixed-effort mental model, it's more demanding. The skill should acknowledge this tension.

Rules 8 (matter-of-fact tone) and 10 (no preamble/closer) are in tension with the conversational norms most AI products are trained to uphold. An agent following rule 10 will sound abrupt, possibly rude, to neurotypical readers. The skill doesn't help the model navigate code-switching between ADHD mode and default mode — or handle the awkward transition when "stop adhd mode" is issued mid-conversation.

**Why it matters.** This skill is important beyond the ADHD use case. It demonstrates that output shaping can be done with the same rigor as any other software specification: grounded facts, falsifiable rules, explicit examples, and a verification checklist. Most output-style prompts are vague vibe statements ("be concise," "be helpful"). This skill treats output formatting as an engineering problem with testable constraints.

The skill also answers a question the agent-design community keeps circling: how do you make an agent *persistently* different without retraining? The answer here is a session-scoped skill with explicit persistence rules and an off switch. It's a pattern that generalizes: any behavioral modification that should survive topic changes can use the same persistence model.

The broader implication: if output shaping for ADHD works this well as a skill, what other cognitive accessibility profiles could be encoded the same way? A skill for dyslexic readers (different formatting, not different content). A skill for readers in crisis (triage framing, not tutorial framing). A skill for non-native speakers (constrained vocabulary, explicit context). The skill-as-accessibility-layer pattern is under-explored, and this is the best reference implementation.

## Architecture — One Ruleset, Ten Harnesses

The repo treats the ruleset as a single source of truth and the harness as a packaging concern. `skills/i-have-adhd/SKILL.md` is the canonical file; everything else is a small manifest that adapts it to a specific agent. `INSTALL.md` documents installation for eleven targets — Claude Code, Codex, Pi, Gemini CLI, Kimi, Qwen, Cursor, GitHub Copilot, Zed, Hermes, and OpenCode — backed by per-harness JSON/TOML in the repo root (`.claude-plugin/marketplace.json`, `.codex-plugin/plugin.json`, `kimi.plugin.json`, `qwen-extension.json`, `gemini-extension.json`, `agents/openai.yaml`, `agents/gemini.toml`).

Two activation postures run through all of them. The **opt-in** posture is the default: in Claude Code, Qwen, and Codex, nothing happens until the skill is explicitly invoked, enforced by `disable-model-invocation: true` in SKILL.md (or `policy.allow_implicit_invocation: false` for Codex). The **always-on** posture is opt-in at session granularity instead: a flag file (`~/.claude/.i-have-adhd-always`) or a pasted instruction block loads the rules from message one.

### The Pi extension — a session-persistent state machine

`extensions/i-have-adhd.ts` is the most sophisticated integration, and it makes explicit the project's real engineering problem: keeping an output style *persistently* applied across a long session without rewriting the system prompt every turn. Four details separate it from a naive plugin:

- **Inject once, not per-request.** `syncContext` asks whether the ruleset is actually in the model's context (`rulesAreInContext` scans `buildContextEntries` for the newest `i-have-adhd-rules` vs. `i-have-adhd-disabled` marker) and injects only when it's missing.
- **Survive compaction.** A `session_compact` handler re-runs `syncContext`, because compaction drops summarized entries and the ruleset must be injected again — the same failure mode [[Claude Code Skills System]] documents for skills, handled explicitly rather than left to "re-invoke it after compaction."
- **Persist state.** On/off is written as a typed session entry (`i-have-adhd-state`), so a forked or resumed session restores the prior choice instead of resetting; a saved choice wins over the always-on flag default.
- **Treat "stop adhd mode" as a protocol message, not a prompt.** The `input` hook intercepts the phrase and, in headless mode, rewrites the turn into a literal "ADHD mode disabled" confirmation so the disable is deterministic.

### The always-on hook

`hooks/always-on.mjs` is a `SessionStart` hook (matcher `startup|resume|clear|compact`) that injects the full ruleset — but only when the opt-in flag file exists. The design constraint is "never block session start": every failure path `exit(0)`s. Three parallel implementations (Node default, POSIX `sh` fallback, PowerShell) all strip SKILL.md's YAML frontmatter with the same two-pass logic and resolve the skill path relative to the script's own location rather than trusting an env var — a small supply-chain-hygiene detail (a hostile `CLAUDE_CONFIG_DIR` cannot redirect the injection).

## Eval Harness — Output Shaping as a Measurable Regression

The most unexpected part of the repo is that a "response style" prompt ships a full paired eval harness. `scripts/run_evals.py` runs baseline vs. candidate conditions over 14 cases (`evals/cases.jsonl`) and scores them blind on a weighted rubric (`evals/rubric.md`): correctness 35%, autonomy 25%, actionability 20%, safety 10%, concision 10%.

Three choices make it unusually rigorous:

- **Paired experimental design.** Conditions are comparable only when judged on identical `(case, trial)` rows; `_check_pairing` rejects any mismatch. Runs are resumable — completed rows are skipped.
- **Runner isolation.** The Claude runner passes `--setting-sources ""` and the Codex runner `--ignore-user-config --ephemeral`, because the repo's *own* always-on flag would otherwise inject the ruleset into the baseline and "measure the skill against itself." The pinned model is recorded as part of the result.
- **A release gate, not just a score.** A candidate ships only if it has no blocker findings, correctness and safety are each within 0.1 points of baseline, and the weighted score beats baseline — with the rule that any public competitor claim must use the same cases, models, trials, and rubric. That is the same discipline [[SimpleEnglish]] brings with its regex linter, but with an LLM judge and a statistical gate instead of a deterministic one.

The case catalog also encodes the skill's *negative space*: alongside "lead with the action" cases sit `medical-boundary` ("does using this style prove I have ADHD? — no") and `destructive-action` (confirm before `rm`-style commands), testing that the style doesn't over-tighten into harm.

---

*Sources: [[raw/i-have-adhd]], [[summary/i-have-adhd]], [[raw/i-have-adhd-skill]]*
*Last updated: 2026-08-14*
