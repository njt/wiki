# Various LLM Smells

Shiv catalogs the patterns that become visible after months of using LLMs to polish prose — the tells you can't un-see once you've seen them. Short, actionable, and refreshingly free of the moral panic that usually accompanies this genre. Not a takedown; a field guide.

---

## Key Quotes

> "ai-smell seems like an artifact that emerges across various AI assisted tasks."

The thesis in one sentence. Shiv frames these not as bugs in the model but as emergent properties of the human-AI workflow itself — what happens when a particular kind of optimization meets a particular kind of taste. Same phenomenon as [[Where the Goblins Came From]], but from the user side rather than the training side.

> "I'm not against LLM/AI usage for creative tasks. This is just me noticing things."

The disclaimer is the point. Shiv isn't making a case against using LLMs — they started a math blog *using* LLMs. This is pattern recognition from the inside, not critique from the outside. Compare with [[Automatic Programming]], where antirez draws a bright line between "assisted" and "abdication" — Shiv is operating firmly on the assisted side and still finding tells.

> "Way too many punchlines. Short, declarative, quasi-profound one-liners."

Shiv identifies four specific writing tells:
- **Punchline density** — every paragraph ends with a mic drop
- **Consecutive short sentences** — the staccato rhythm of emphasis
- **"X is the Y of Z"** constructions — the rhetorical formula that sounds deeper than it is
- **"Not just X, it's Y"** frames — the aesthetic-satisfaction upgrade pattern

These are more granular than Sam Kriss's catalog in [[Why Does AI Write Like That]] — Kriss traces the *words* (delve, em dashes, "an X with Y and Z"), while Shiv traces the *structures*. Together they're a complete diagnostic manual.

---

## Key Themes

### #pattern #concept — The Recognizability Threshold

Shiv's core insight: there's a threshold after which AI artifacts become obvious. You don't notice them at first — the output reads as "better." Then around month three, the patterns crystallize. This maps to what [[Vibe Coding and the Maker Movement]] calls "evaluative anesthesia" — the initial dopamine of "this is so good" masks the tells until you've built enough exposure to see them.

### #concept — User-Side vs. Reader-Side Detection

Most AI-writing criticism comes from the reader's perspective ([[Why Does AI Write Like That]], [[Tech Writers and AI (HN Discussion)]]). Shiv's contribution is the **user's self-diagnostic**: how to catch yourself over-relying on the LLM before anyone else does. This is the prose equivalent of [[Write Only Code]]'s "slop radius" — how far does the AI fingerprint extend beyond what you intended?

### #tool — Visual Tells as a Separate Channel

Shiv's second section on website tells (JetBrains Mono, the blinking-dot badge, specific card patterns) isn't about writing at all — it's about **visual design convergence**. When every AI-generated landing page looks the same, the tells become ambient. This connects to the broader #pattern of AI monoculture: [[Where the Goblins Came From]] showed it in model behavior, [[Why Does AI Write Like That]] showed it in prose, Shiv shows it in UI.

---

## Critical Analysis

**The useful part is the method, not the list.** Any specific tell will age out — future models won't use JetBrains Mono by default, won't lean on "not just X, it's Y." What persists is the practice: use the tool heavily for months, then step back and catalog what you can now see that you couldn't before. That's the transferable skill.

**The deleted math blog is the ghost at the feast.** Shiv wrote a math blog, used LLMs to polish it, noticed the tells, and then... deleted the blog? The piece doesn't say whether the tells were the reason. But the sequence is suggestive: you notice the AI fingerprints, and suddenly the work doesn't feel like yours anymore. This is the existential problem beneath the technical one — not "does AI write badly" but "does AI-assisted writing cease to be mine." See [[Radical Accountability]] and [[Creative Firewall]].

**The gap between Shiv and Kriss is instructive.** Kriss ([[Why Does AI Write Like That]]) catalogs AI prose tics as a literary critic examining a phenomenon from outside. Shiv catalogs them as a practitioner examining their own output from inside. Both are valuable, but Shiv's is more actionable — you can use these tells as a checklist against your own drafts. Kriss tells you what to see; Shiv tells you what to fix.

**The website tells section is underdeveloped and therefore more interesting.** The writing tells are specific and named. The visual tells are just screenshots. This asymmetry suggests the visual tells are harder to articulate — they're gestalt, not lexical. A blinking-dot badge component "smells" like AI, but you can't reduce it to a formula the way you can with "X is the Y of Z." The visual vocabulary for AI detection doesn't exist yet.

---

*Sources: [[raw/various-llm-smells]]*
*Last updated: 2026-05-31*
