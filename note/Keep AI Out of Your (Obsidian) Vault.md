# Keep AI Out of Your (Obsidian) Vault

mehdio's contrarian field note: Obsidian's open, local markdown makes AI integration trivially easy — and that is exactly why you shouldn't let AI write into your vault. Generation is a dead end for a personal second brain; the power lies in a *deliberately created* graph of your own notes, and AI's legitimate home is search and retrieval, not composition or connection-making.

---

## Key Quotes

> "A summary of a PDF is noise. An insight I had from reading the PDF is signal." — Kepano, quoted by mehdio

The thesis in one line. A summary is what the model noticed; an insight is what *you* noticed. mehdio draws the practical boundary from it: long AI summaries are "usually just noise," while a 1–2 sentence intro is harmless — provided it's marked as AI-generated so that in five years nobody mistakes it for yours. This is the same taste judgment [[Slop Score]] tries to turn into a number.

> "Your notes will be prompts or libraries for AI tomorrow, but not if you generate them."

The forward-looking argument. Human-curated notes are future feedstock for models precisely because they carry conviction and decisions. A vault full of generated notes collapses toward the average: "no conviction, no decisions made, just all equally similar." mehdio wants human-curated knowledge preserved at scale, and sees it as better training data than synthetic output.

> "The power lies in the deliberately created graph of notes, your very own Second Brain."

The heart of "whose knowledge system is it?" Connections an agent makes aren't *your* connections — you made them only because you had an idea. Automating the links "skips the thinking, and IMO, the insights." Hence: "It's so hard to finish an idea that is not yours."

---

## Key Themes

#concept #obsidian #second-brain #ai-slop #note-taking

**AI in, not on.** The usable AI for Obsidian is retrieval-side: the Obsidian CLI for vibe-coding agents (faster file ops and grep), Smart Connections and Graph Analysis for vector/similarity search, Omnisearch for finding anything in a 26,000-file vault in split seconds. Generation-side AI — summaries, auto-notes, auto-tagging — is what mehdio says to refuse.

**Slop as a search and thinking problem.** The cost of generated text is double. It pollutes search (you fight through noise to reach your own writing), and it erodes provenance (you stop knowing which thoughts were yours). mehdio's remedy is separation: a dedicated PARA "AI" folder, hidden from search, with anything generated clearly quoted and labeled.

**Search is an organizational question.** Zettelkasten plus a good search plugin beats more machinery. "If you need more advanced features, you can just add Smart Connections." The organizational discipline is the product; the AI is an accelerator on top, not a replacement for it.

**Cultivating knowledge for the models themselves.** The piece ends on an unusual economic note: if humans stop keeping their own knowledge bases, models retrain on synthetic sameness, so we need human-curated knowledge at scale — "though we need to find an economic model that also works for the creator."

---

## Critical Analysis

This is a personal preference argued as a universal rule, and it's better read as the former. mehdio is a heavy Obsidian user with a large, mature vault and a Zettelkasten discipline; the advice lands for someone who already *has* a second brain and is protecting it. For a newcomer with no note-taking habit, the stakes are different — the choice isn't "AI slop vs. my thinking" but "AI slop vs. nothing."

The strongest claim is the compounding one: that links you made yourself are where your ideas come from, "sometimes years back," and that automating them forfeits that payoff. That's an argument about *thinking*, not about note-taking, and it's hard to refute. The weakest is the closing economics — that we need human-curated knowledge "at scale" as training data — which is asserted rather than argued, and elides the incentive problem it names in the same breath.

The piece also quietly softens its own headline. "Keep AI out of your vault" turns out to mean "keep AI *generation* out of your vault": retrieval, vector search, local models, and even a hidden AI folder are all permitted. That's a real distinction, not a retreat — it's the difference between using AI as a lens on your knowledge and letting it be the author of it.

---

## How It Lands in This Wiki

- **[[Immaculate Knowledge Graph]]** — mehdio complicates Harper Reed's recipe: Reed pipes an AI-extracted graph of meeting transcripts straight into Obsidian, which is precisely the generated connections mehdio says dilute a second brain.
- **[[LLM Wiki]]** — mehdio is a direct dissent from the Karpathy pattern this wiki implements, at least for *personal* knowledge: retrieval and cross-referencing yes, generation no.
- **[[Introduction to Obsidian]]** — mehdio strengthens Hogan's minimalism by extending "keep it simple" into "keep the AI out," while sharing the file-over-app premise that your files, not the app, are the point.
- **[[Wuphf — Karpathy-Style Agent Wiki]]** — the same "knowledge vs. noise" debate the HN thread staged, but aimed at an individual's second brain rather than a team's agent wiki.

---

*Sources: [[raw/using-obsidian-with-ai]], [[summary/using-obsidian-with-ai]]*
*Last updated: 2026-09-04*
