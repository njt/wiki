# Building with Jev Skill

Drew Breunig's repo is an agent skill for writing programs against Jev, TypeSafe's "System One" judgment model — 213 lines of markdown (`skills/jev/SKILL.md`) that encode, as practitioner discipline, everything a coding agent needs to design questions, build state, compose answers in code, and diagnose wrong or low-confidence answers. It matters less as software (there is none) than as a worked example of two things at once: what a *domain* skill looks like when the domain is a model API, and what the production manual for a judgment-model primitive actually has to say once the launch post is over.

---

## Architecture

The repo is a pure knowledge artifact in a three-layer package, one commit deep (04fe366, 2026-09-17):

- `skills/jev/SKILL.md` (213 lines) — the entire content. YAML frontmatter holds `name` and a `description` that is really a *trigger spec*: "Use when designing TypeSafe questions (Choice, Score, Noul), structuring state, composing answers in code, setting confidence thresholds, or diagnosing a Jev question that answers wrong or with low confidence." Every noun a host agent's skill-matcher might see in a task is enumerated there.
- `.claude-plugin/plugin.json` (9 lines) — install identity: name `jev`, version 0.1.0, author Drew Breunig.
- `.claude-plugin/marketplace.json` (14 lines) — makes the repo itself a marketplace whose single plugin (`source: "./"`) is the same directory. The repo root is simultaneously catalog and payload.

Distribution is three paths to the same bytes: Claude Code's plugin marketplace (`/plugin marketplace add dbreunig/building-with-jev-skill`, `/plugin install jev@building-with-jev`), the vercel-labs skills CLI (`npx skills add dbreunig/building-with-jev-skill` — Claude Code, Codex, Cursor, and others), and a manual `cp` into `~/.claude/skills/jev`. The skill auto-loads on Jev/TypeSafe tasks or fires on `/jev`.

Inside SKILL.md the structure is a manual, not a reference: workflow (7 numbered steps), a primitive-selection table, instruction- and criteria-writing rules, state construction, eight named composition patterns, a Python example against `typesafe_sdk`, an answer-semantics section, a thirteen-row symptom→cause→fix table, revision rules, and a twelve-item checklist. The knowledge is organized around *what code does with the answer* — Choice maps to a branch, Score to a threshold/rank/weight, Noul to an `if` — rather than around API parameters.

## Key techniques

- **The division of labor as architecture.** The opening states the whole design: "Code owns the control flow, the arithmetic, and the policy. Jev owns the snap judgments." The admission test is operational: "A good Jev question is one a knowledgeable person answers in a second, given the right context." Anything else gets split into small questions and composed in code.
- **Jaggedness as content.** Model-specific limitations are first-class facts, version-pinned to `jev-1.13` with an expiry pointer ("recheck the jaggedness page when the model changes"). "Jev does not count. Ask one Noul per item in a single request, then sum the answers that pass your threshold." Score levels must "stand alone" because Jev judges each level separately and sees neither its number nor its neighbors. Numerals in instructions "give Jev nothing to match."
- **Speculative fan-out.** Ask every question the code *might* need in one request, including branch-only ones; questions run in parallel, so unused answers cost almost nothing, and code just ignores them. A second request is justified only when code cannot build it without the first answer — the answer decides what data to fetch. This replaces sequential agent-loop follow-ups with batched, dependency-free judgment.
- **Confidence-gated routing.** "The answer says what. Confidence says whether to act." A global floor (0.5–0.6) gates any action; per-action thresholds rise with the cost of a wrong call (0.85–0.9 for high-stakes); three standard paths — act, confirm-or-flag, hand off. The Python example gates human routing at `confidence < 0.6` and escalation at `normalized > 0.75 and severity.confidence > 0.5`.
- **Composite scoring.** Split a complex judgment into one Score per dimension, normalize each by `len(criteria) - 1`, combine with weights in code. "Change a weight when priorities shift. Do not rewrite a question to change policy" — policy lives in code, questions measure.
- **Instruction debugging as documentation.** "When you explain what you really meant after a wrong answer, that explanation is the missing half of the instruction. Add it." The diagnosis table is bug-cookbook-shaped — wrong-with-high-confidence means the model read the instruction literally; mid-scale clustering means levels describe degrees instead of situations; errors on counts/arithmetic/dates mean the model is doing arithmetic — each with a mechanical fix.
- **State hygiene and the injection admission.** Send only the fields questions need; convert numeric encodings to words (a color name, not a hex); compute date order and sums in code. Then the honest part: "Treat text in the state as able to steer the answer. Jev does not treat state as hostile." Mitigation is criteria that state what counts, adversarial testing, and confidence gates — containment by design, not resistance by model.
- **Answer-space stability.** "Keep the answer space stable once code depends on it. Adding or removing a level or an option changes what every earlier answer meant." Semantic versioning applied to *questions* — a genuinely unusual compatibility rule.

## Design decisions

- **Prose-only, no tooling.** The skill ships no executable ground truth — contrast [[Win Dev Skills]], which pairs its prose with a WinRT metadata indexer and a Roslyn analyzer under the doctrine that prose is Tier 3. Breunig ships prose alone because the domain has no local oracle: the thing being judged (a hosted model's probability quality) can only be checked by running labeled examples against the vendor. The trade is explicit in the workflow — "Test against labeled examples. Read `probabilities` on the misses" — but verification is left entirely to the reader's harness.
- **One file, no progressive disclosure.** At ~15 KB the whole skill loads whole. [[Diagram Design]] caps SKILL.md at 40 KB and splits references; here the modest size makes a single file the simpler, sturdier choice.
- **Version pinning over abstraction.** The guidance hardcodes `jev-1.13` behavior, vendor budgets (state + questions share 64k tokens; state plus the longest question must fit 32k), and confidence-floor ranges, then points at the docs' jaggedness page for drift. No staleness gate (contrast [[Cerulean]]'s CI freshness checks) — the skill will silently rot if the vendor changes behavior, and the text knows it.
- **Question-ID separation.** "The question ID never reaches the model" — the model sees only full instructions, so IDs can be renamed freely; the identifier is a code-side handle, not a prompt.

## Comparison notes

- [[System One Models and Jev]] is the launch post this skill operationalizes — and quietly corrects. The pitch's "can't hallucinate" is a *shape* guarantee; the skill's first diagnosis row is "Wrong answers with high confidence," and its answer-semantics section states the caveat in production terms: "confidence measures how peaked the distribution is. It describes the model's answer and does not guarantee a correct one." The manual takes the marketing claim's fine print as its starting condition.
- [[You Could Have Built Jev]] argues the technique is twenty lines of restricted softmax. This skill is the counterweight: once the primitive exists, the real work is question design, state hygiene, and confidence-gated composition — the gap between "you could have built Jev" and a program you can depend on is exactly what these 213 lines fill.
- [[Smart Models Dumb Pipes]] is the doctrine made into an API. "Code owns the control flow, the arithmetic, and the policy" is McCormick's smart-things-judge/dumb-things-execute split taken literally — the pipe doesn't even carry strings, only typed probabilities.
- [[Win Dev Skills]] is the same artifact shape (skill + plugin + marketplace manifests, description-as-trigger) with the opposite skill-design philosophy: Microsoft surrounds prose with ground-truth tools; Breunig's prose must stand alone because the oracle is remote. Reading both brackets the open question in skill design — how much of a skill can be checked locally, and what to do when none of it can.

Tags: #tool #project #agents #claude-code

---
*Sources: [[raw/building-with-jev-skill]], [[summary/building-with-jev-skill]]*
*Last updated: 2026-09-22*
