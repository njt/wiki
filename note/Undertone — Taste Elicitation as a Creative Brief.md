# Undertone — Taste Elicitation as a Creative Brief

Undertone (undertone.guide) is a client-rendered web tool that turns a person's visual instincts into a written creative brief through guided pairwise comparisons ("This one" / "Hard no") over sample designs — layout compositions, typefaces, colour palettes, print finishes — with held-out checks that test whether early preferences predict later choices. What the fetched source really documents, via the tool's own UI copy, is an unusually honest epistemology of taste inference: every explanation is a "provisional explanation, not measured traits or calibrated confidence," and the user's written notes always outrank the model's inferences.

---

## What the source actually is

The site is a JavaScript single-page app; the server returns an empty React shell, so the raw capture consists of the first screen (a mock "Sunday Review" editorial layout under comparison) plus the tool's own interface copy extracted from its bundle. That copy is the substance: it shows a deliberately budgeted interview protocol, not a generic swipe UI.

## Key quotes

> "These are competing provisional explanations, not measured traits or calibrated confidence. Internal weights only order the explanations and select useful comparisons."

The most striking line in the whole copy. Most taste-recommendation products overclaim ("we know your style"); Undertone repeatedly does the opposite, refusing to confuse an ordering heuristic with a measurement. This is a tool that anticipates its own misreadings in the UI itself.

> "A whole reference changes several things at once. Descriptions are approximate, and your reason may fall outside these features. Your written notes take precedence in the brief."

The confounding problem stated plainly: liking a layout means liking its typography, palette, and photo all at once. The escape hatch is human prose — the brief treats the user's own words as the primary source and the inferred dimensions as scaffolding.

> "Repeated means a treatment has won at least three times across two applications, after at least four decisive comparisons. It is evidence to work with, not certainty."

Replication logic made user-visible. The tool publishes its own evidentiary thresholds (three wins, two settings, four decisive comparisons) — a rare move in preference-driven products, where such thresholds are usually hidden.

> "The guided pass ended after you expressed no preference between three pairs; no preference is required, and further exploration is optional."

Five distinct exit conditions — no-preference, double-rejection, want-better, like-both, no-ranking — each with its own framing. The tool treats "I don't know" and "both are fine" as first-class data rather than failures of the algorithm.

> "Your shortlisted preferences held up in fresh comparisons. They are ready to discuss with your designer."

The output is explicitly a handoff artifact for a human designer, not an automated pipeline. The endpoint is a conversation, with the brief as its agenda.

## Key themes

- #tool — **Undertone as a preference-elicitation instrument.** The interaction design (budgeted passes, matched settings that isolate one variable, prediction checks on fresh pairs) is a forced-choice survey methodology wearing a design-tool skin.
- #concept — **Taste as held-out prediction, not profile.** The system is honest that it shortlists reference points ("names describe reference points, not a fixed label for your taste") and validates by predicting choices it hasn't seen — an evaluation loop that mirrors test/ train discipline.
- #pattern — **Notes outrank inferences.** "Your written notes take precedence in the brief" appears in several forms; the durable artifact is co-authored, with the human's explicit boundaries and no-preferences as load-bearing content.

## Critical analysis

This is a thin source — UI copy, not documentation — but the copy is unusually disciplined and rewards reading. Two things make Undertone interesting to this wiki. First, it is a specification tool for the hardest underspecified input of all: what the person actually wants. Every agent-coding failure mode in this wiki traces back to "you asked for the wrong thing," and Undertone attacks exactly that gap, but for visual direction, with the comparison-based elicitation that spec-writing prose never manages. Second, its honesty posture is the opposite of the confident-profile pattern: it publishes uncertainty, thresholds, and exit conditions in the user's face, which is what "calibrated" would look like if preference products meant it.

The limits are real. "Uncalibrated, not a probability" is admitted outright; the finite vocabulary "can miss your reason for choosing"; and the whole-reference confound is only mitigated, not solved, by matched comparisons. It is also unclear how the "LunaRoute" synthesis step works, since no prose describes it. Still, as a demonstration that preference elicitation can be budgeted, checked against held-out pairs, and subordinated to the user's own words, it is a stronger epistemic artifact than most taste tools ship.

## How it relates

- [[Agentation]] — both are "turn human judgment into machine-usable input" tools, but they point in opposite directions: Agentation takes feedback about an existing UI and makes it precise (selectors, paths); Undertone extracts direction before anything exists. Together they bracket the brief-to-feedback loop.
- [[Specifications as the Product]] — Undertone is a working instance of the thesis that the durable artifact is the written direction, not the output: it spends its whole interaction budget producing an editable brief the human keeps revising, and its explicit boundaries and no-preferences are spec content.
- [[Ask for What You Want (Robin Sloan)]] — Sloan's VISION.md asks the person to write their intent loose and exhaustive; Undertone asks them to reveal it through forced choices instead, a complementary answer to the same "capture what you actually want" problem, and it refuses to pretend the elicitation ever finishes.
- [[Make Pages Interactive]] — that tool shows the artifact to the agent and lets the human annotate; Undertone shows artifacts to the human and models the annotating. Both make preference data first-class rather than prose feedback, though Undertone is far more careful about what the data proves.

---
*Sources: [[raw/undertone-guide]], [[summary/undertone-guide]]*
*Last updated: 2026-10-10*
