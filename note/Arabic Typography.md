# Arabic Typography

A deep interactive essay by larrasket tracing Arabic typography from 10th-century calligraphic proportions through the Unicode fossil layer to the modern web, where no browser can justify Arabic text correctly because kashida (letter stretching) and inter-word spacing are fundamentally different operations — and the standoff between shaping engines and font foundries has persisted since 1997.

---

> Every joint a small accident.

— on the first Arabic movable-type book, *Kitāb Ṣalāt al-Sawāʿī* (Fano, 1514)

The first Arabic printed books were disasters. Each letter needed ~4 sorts (isolated/initial/medial/final), multiplied by hundreds of ligatures, so a single fount could run to 500+ pieces of type. The Paganini Qurʾān (Venice, 1537) was so error-riddled it was lost for 450 years — rediscovered in 1987 in a Venetian friary. This wasn't incompetence; it was the wrong technology stack. Moveable type was designed for Latin's discrete letters; Arabic is "cursive *always*" — letters negotiate their shape based on neighbors, and that negotiation happens at render time, not at cast time.

> His system outlasted everybody who hurt him by a thousand years.

— on Ibn Muqla (d. ~940 CE), the Abbasid vizier who systematized proportional Arabic script

Ibn Muqla's biography reads like a George R.R. Martin subplot: imprisoned twice, right hand amputated, tongue cut out, died in prison, buried three times. His *al-khaṭṭ al-mansūb* (proportional script) — letterforms built from rhombic dots of the reed nib, with systematic kashida elongation rules — became the foundation of Arabic typography. Refined by Ibn al-Bawwāb (~1001) and Yāqūt al-Mustaʿṣimī (who survived the 1258 Mongol sack of Baghdad by climbing a minaret with his pens), this system is still what modern shaping engines implement.

> U+FDFD ﷽ is a monument from the era when rendering was baked into the encoding.

The Unicode Arabic Presentation Forms block (U+FB50–U+FEFF) stores *shapes* not *letters* — several hundred codepoints preserved solely for round-trip compatibility with 8-bit code pages. This is the typographic equivalent of fossil fuel: a decision from 1995 that every text stack still pays for. The article demonstrates this with a search demo: identical-looking Arabic names fail to match across encodings. NFKC normalization is the fix, but only if you know to apply it.

> No quarterly report has a line item for "Arabic users can now justify a paragraph."

The structural problem: Latin justification places break points and stretch points at the same location (inter-word spaces). Arabic pulls them apart — breaks between words, stretches *inside* words via kashida. Line capacity to stretch depends on which letters landed on each line. Break points and elongation "have to be chosen together, against a cost function that depends on the actual glyphs." This is a well-understood algorithm. Microsoft Word has shipped it since the late 90s. InDesign Middle East does it. But Chrome, Firefox, and Safari all fall back to inter-word spacing.

The OpenType `jstf` table (1997) was supposed to solve this — fonts declare justification priorities, engines execute them. But "virtually no shaping engine reads it, so virtually no foundry ships it, so no engine acquires a reason to start." IE 5.5 shipped `text-justify: kashida` in 2000 — "the only software vendor on earth that could justify Arabic correctly on a screen" — and the CSSWG later deleted the value from the spec. Both the W3C Arabic Layout Requirements task force and the CSSWG Arabic justification issue remain open in 2026.

> The shaping engine running in your browser at this moment was for years carried by an engineer the US government considered a security risk.

Behdad Esfahbod wrote much of HarfBuzz, the shaping engine that landed in Chrome and Android in 2012. He was detained at the US border in 2017 on suspicion of being Iranian. Khaled Hosny later co-maintained HarfBuzz, created the Amiri font (a one-man reconstruction of the Bulaq Press typeface), and filed dozens of CLDR bugs. If you read well-rendered Arabic on the open web in 2026, you are probably reading it through their unpaid labor.

The article includes ten interactive demos: building the name Muḥammad letter by letter to show positional negotiation, toggling between three Arabic digit sets (Arabic-Indic, Western, Extended Arabic-Indic), watching a phone number render backwards when preceded by Arabic text, seeing vowelled text beheaded by `overflow: hidden`, and experiencing the "caret schizophrenia" where cursor position at bidi boundaries is a browser-specific opinion.

## Key Themes

#typography #unicode #internationalization #open-source #design

## Critical Analysis

This is the best kind of technical writing: a deep domain explained through its failures. The article doesn't just describe how Arabic rendering works — it shows why it's broken, in your browser, right now, with demos you can interact with. The historical through-line (Ibn Muqla → Fano → Bulaq → Simplified Arabic → HarfBuzz → CSSWG deadlock) makes the technical argument feel inevitable rather than esoteric.

**The sharpest insight**: Arabic typography's problems aren't hard because they're unsolved — they're hard because the people who need them solved have no commercial leverage. Kashida justification is a solved algorithm. The `jstf` table exists. Every piece is in place except the will to connect them. The article makes this visible without polemic; the "Won't Fix" entry at the end of the historical timeline does more work than any rant could.

**What's missing**: The article is framed around the web stack, which makes sense given the audience, but it barely touches mobile — where most Arabic users actually read text. Android's text stack has its own kashida story (some OEM skins ship it, AOSP doesn't). Also unmentioned: the politics of Arabic script reform (the Academy of the Arabic Language in Cairo has been debating letterform simplification since 1938) and the role of colonial printing infrastructure in shaping which Arabic typographic traditions survived.

**The Amiri thread**: Khaled Hosny's Amiri font is quietly the hero of this story — a single person reconstructing a historical typeface, shipping curvilinear kashida in four sizes, and releasing it under OFL. It's the typographic equivalent of the HarfBuzz story: critical infrastructure maintained by individuals whose work the entire Arabic-reading internet depends on.

**Why this matters beyond typography**: This is a case study in how technical debt compounds when the affected users lack market power. Every Arab speaker who opens a webpage sees ragged-left justified text. Every Farsi speaker using an npm PDF generator gets isolated-form gibberish. These aren't edge cases — Arabic is the fourth most-spoken language on earth — but the rendering stack treats them as such because the advertising revenue doesn't flow through Arabic-language browsers at the same volume. The article makes this structural argument without ever being boring about it.

The encoding layer this article diagnoses — the Presentation Forms block as a fossil from the 8-bit code page era — is exactly the history Joel Spolsky traces in [[Character Sets and Unicode]]: the OEM free-for-all that made Unicode necessary, and why code points and encodings are separate concepts that too many programmers still conflate.

---

*Sources: [[summary/arabic-typography]]*
*Last updated: 2026-07-03*
