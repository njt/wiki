# StoryScope

A 2026 paper that builds a systematic taxonomy of *narrative* (not stylistic) differences between human and AI fiction — and shows these structural choices are a more durable detection signal than surface prose patterns. Across 61,608 stories from 5 LLMs and human authors, narrative features alone achieve 93.2% detection accuracy, survive stylistic editing nearly unscathed, and reveal per-model "fingerprints" (Claude's flat escalation, GPT's gossip obsession, Gemini's bleakness).

---

## The Big Claim

Style-based AI detection is a dead end. New model versions reduce em-dash usage, fine-tuning drops detection rates from 97% to 3%, and surface patterns are trivial to edit away. But *narrative choices* — how a story structures time, whether the narrator explains the theme, how characters resolve conflict — are much harder to "humanize" because changing them requires structural rewrites, not post-hoc word swaps.

StoryScope proves this empirically: 304 discourse-level features across 10 narrative dimensions, extracted automatically via LLM pipeline, classify human vs. AI fiction at 93.2% macro-F1 *without any stylistic cues*. When AI-generated stories are edited to remove surface artifacts (LAMP rewriting), detection drops only 1.6 points. The structural choices are the signal.

---

## Key Quotes

> "AI stories over-explain themes and favor tidy, single-track plots while human stories frame protagonists' choices as more morally ambiguous and have increased temporal complexity."

The core finding in one sentence. AI fiction is over-determined: everything connects, everything means something, and the narrator will tell you what. Human fiction is messier and trusts the reader more.

> "Where a human author might write that a character 'felt afraid,' AI renders fear as a tightening chest, cold sweat, and dimming lamplight."

AI has learned *show-don't-tell* too well. It over-indexes on embodied emotion (81% vs. 38% human) and almost never uses explicit emotion labels (8% vs. 29%). This is the most surprising finding — the cliché about AI writing is that it's vague and abstract, but StoryScope finds the opposite: AI writing is *excessively concrete* about bodies and senses, allergic to simply naming a feeling.

> "AI writes as though no one is watching."

Humans break the fourth wall (67% vs. 39%), address readers directly (28% vs. 7%), and reference specific texts and authors (47% vs. 24%). AI fiction is a closed system — it generates story-worlds that don't acknowledge an audience exists.

> "The five AI models occupy overlapping regions of narrative feature space, well-separated from human stories. Even the closest human-AI centroid pair is farther apart than the most distant AI-AI pair."

AI models have converged on a shared narrative region. They're not just individually different from humans — they're collectively similar to each other in ways that separate them from human storytelling as a category. This is the "artificial hivemind" finding from a narrative-structure angle.

> "Claude produces notably flat event escalation, GPT over-indexes on dream sequences, and Gemini defaults to external character description."

The per-model fingerprints are the paper's most fun finding. Claude is the restrained literary writer, GPT is the gossipy social novelist, Gemini is the bleak external observer. These aren't style tics — they're structural storytelling defaults.

---

## Key Themes

- `#concept` **Narrative features vs. stylistic features** — the central distinction. Style is word choice and sentence rhythm; narrative is plot structure, temporal order, character agency, thematic explicitness. Style is easy to edit; narrative requires rewriting the story.
- `#concept` **AI narrative convergence** — all five LLMs cluster in a shared region of narrative space, distinct from humans. This isn't about individual model bias; it's about a structural tendency in AI-generated fiction regardless of architecture.
- `#concept` **Per-model fingerprints** — each LLM has identifiable narrative defaults (Claude = flat escalation, GPT = gossip, Gemini = bleak external description). These are consistent enough to enable 6-way authorship attribution at 68.4% F1 from narrative features alone.
- `#tool` **StoryScope pipeline** — three-stage LLM pipeline: structured template extraction → cross-source comparison → feature discovery → feature assignment → XGBoost + SHAP classification. Cost ~$4,400 for the full pipeline.
- `#pattern` **Over-determination as the AI signature** — AI stories explain their themes, resolve their plots neatly, and render emotion through bodies rather than naming it. The common thread: AI doesn't trust the reader to infer meaning.
- `#pattern` **Human narrative diversity** — human stories are rarer (mean rarity percentile 0.71 vs. 0.49), more dispersed in feature space, and draw from a broader repertoire of narrative techniques.

---

## Critical Analysis

**The template abstraction is the real innovation.** Most AI detection work treats text as a bag of tokens. StoryScope's move — converting prose to structured JSON templates along NarraBench dimensions before comparing — is what makes the narrative-vs-style separation possible. The templates force downstream analysis to reason about *what happens in the story* rather than *how it's written*. Without this step, the features would be style-heavy (the authors confirm this: only 6 of the top 20 features overlap between template-based and raw-text pipelines).

**But the pipeline is circular in a way the authors don't fully address.** LLMs extract features, LLMs compare templates, LLMs propose features, and then an LLM assigns those features to stories. The classifier is XGBoost, but every input feature is LLM-annotated. The human validation study (κ = 0.84) helps, but it's on 240 feature-items across 12 stories — a tiny fraction of the 61,608-story corpus. We're measuring "do LLMs detect patterns in LLM-generated text that LLMs have identified as potentially distinguishing" — which is not quite the same as "do narrative features distinguish human from AI writing."

**The copyright tension is unresolved.** Human stories come from Books3 (the same dataset at the center of multiple lawsuits). The authors are careful — they don't release the human stories, they use Books3 "strictly for academic purposes" — but the pipeline's entire value proposition is that it can distinguish human from AI fiction, and its human baseline is built on copyrighted material used without author permission. This isn't a criticism of the research so much as a tension the field hasn't figured out.

**The most important finding might be the convergence, not the detection.** The fact that five different LLMs from five different organizations all converge on the same narrative region is striking. It suggests something about the training objective (next-token prediction on internet-scale text) that produces systematic narrative homogenization regardless of architecture. If this holds, better models won't fix it — they'll just cluster tighter.

**"Rarity as proxy for originality" is clever but incomplete.** The paper operationalizes originality as statistical rarity in narrative feature space. This is measurable and legally relevant (US copyright's "minimal degree of originality"), but it conflates *unusual* with *original*. A story can be statistically common in its narrative features and still be original in execution; a story can be rare because it's incoherent. The authors acknowledge this implicitly by using "rarity" rather than "originality" in their metrics, but the framing in §1 invites the stronger claim.

**The real durability question is unanswered.** The paper argues narrative features are more durable than stylistic ones because they require structural rewrites to change. But will future models trained on human-written fiction (or fine-tuned to mimic narrative diversity) close this gap too? The authors show narrative features survive *surface* editing, but they don't test whether a model *instructed to vary its narrative choices* would evade detection. The 6-way attribution results (68.4% F1 vs. 93.2% for binary) already show that AI-AI boundaries are much blurrier than human-AI boundaries — as models improve, the human-AI boundary may blur too.

---

## Related Pages

- [[Why Does AI Write Like That]] — Sam Kriss's taxonomy of AI prose tics. StoryScope is the structural companion: Kriss catalogs the surface symptoms, StoryScope maps the underlying disease.
- [[Various LLM Smells]] — Shiv's field guide to recognizing AI artifacts in your own writing. The user-side experience of the patterns StoryScope quantifies.
- [[Where the Goblins Came From]] — A miniature paperclip maximizer in production: reward models produce distinctive AI behavior, much like training objectives produce narrative convergence.
- [[A Non-Anthropomorphized View of LLMs]] — Halvar Flake: LLMs are functions through ℝⁿ. StoryScope operationalizes this — treating narrative choices as points in feature space that can be measured, compared, and classified.
- [[Creative Firewall]] — Sundar's framework for the boundary between authentic human prompting and AI-optimized output. StoryScope quantifies what's on the AI side of that firewall in fiction.
- [[Performative UI]] — Cataloging tropes as installable components. Parallel to StoryScope's cataloging of narrative defaults as measurable features.
- [[The solution might be cancelling my AI subscription (Wilson)]] — David Wilson's confessional about AI-built projects. StoryScope suggests the problem isn't just volume but homogeneity.
- [[They're Made Out of Weights]] — The philosophical companion: if LLMs are "just weights," what does it mean that their weights converge on the same narrative region?

---
*Sources: [[raw/storyscope]]*
*Last updated: 2026-06-22*
