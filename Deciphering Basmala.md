# Deciphering Basmala

Mark Dominus unpacks the centuries of typographic and calligraphic tradition behind the basmala — the Islamic phrase "bismillah al-raḥman al-raḥim" that opens nearly every Qur'anic surah — and the Unicode workaround (a single codepoint, U+FDFD) that sidesteps the inability of digital font engines to render it properly.

---

## Key Quotes

> "There is a centuries-long tradition of calligraphic expression of this phrase, in the most perfect possible ways."

Dominus frames the stakes: this isn't just text. The basmala is one of the most revered phrases in Islam, and its rendering has been an artistic and spiritual pursuit for centuries. The gap between what tradition demands and what early digital typography could deliver is the whole story.

> "If you're writing English and your fonts are missing ligatures for 'fi' or 'fl,' it's barely noticeable. In Arabic, the same thing looks grossly wrong."

A compact explanation of why Arabic typography is harder than Latin. Cursive-by-default means ligatures aren't decoration — they're infrastructure. The comparison lands because every English reader has seen "fi" ligatures and never noticed them.

> "It's hard for me to imagine any sound between /h/ and /x/ but Arabic has at least three."

Dominus is honest about his limits as a non-Arabic-speaker, and this aside about the difficulty of Arabic phonetics is both endearing and instructive. He mentions familiarity with the glottal stop from "Hawai'i" — the kind of concrete bridge that makes linguistic description actually work.

---

## Key Themes

- **#concept** Tradition as technical requirement — the basmala isn't just text to render; it's a phrase with centuries of calligraphic perfection behind it. Unicode's single-codepoint solution is a hack, but it's a hack that honors what matters.
- **#concept** The Latin bias in digital typography — early font engines were built for Latin scripts. Arabic's cursive nature, contextual letter forms, and calligraphic traditions were afterthoughts. The basmala codepoint is a monument to that original sin.
- **#pattern** Workaround as preservation — U+FDFD didn't solve Arabic typography. It carved out one special case and gave it a single glyph. That's not engineering; that's triage. But when the alternative was blasphemy-by-bad-rendering, triage was the right call.
- **#concept** Cross-cultural linguistic transmission — Dominus traces "al-" through alcohol, algebra, and alchemy into English, noting it's not in "alligator" (Spanish, not Arabic). The quiet joy of the piece is watching a skilled explainer trace connections across languages he admits he doesn't speak.

---

## Critical Analysis

**The article is a masterclass in "I don't know this, let me show you what I found."** Dominus is upfront about not knowing Arabic, and that honesty is the engine of the piece. He's not performing expertise — he's performing curiosity. The structure mirrors his learning process: here's a problem, here's what I learned about it, here's what I still don't understand. This is rarer than it should be in technical writing, where admitting ignorance is often treated as disqualifying rather than qualifying.

**The Unicode codepoint is a beautiful hack that shouldn't exist.** It's the typographic equivalent of hardcoding a special case because the general solution was too hard. That it works — that a single character can carry centuries of calligraphic tradition — is testament to Unicode's willingness to be messy in service of actual human needs. But it also means every other complex Arabic ligature is still at the mercy of font engines that were designed for Latin.

**The piece is fundamentally about respect.** The basmala matters. Getting it wrong matters. The Unicode Consortium understood this; the font designers Dominus praises understood this; Dominus himself understands it, which is why he wrote the article. Technology that doesn't respect what people care about is bad technology. The article doesn't say this explicitly, but it's the argument humming under every paragraph.

**What's missing:** Dominus gestures at Saleh's article about the "twice-mutilated vizier" and the Beirut newspaperman but doesn't summarize it. The reader who doesn't click through misses the most dramatic part of the story. This is a stylistic choice — Dominus wants you to read Saleh — but it leaves a hole in the narrative for anyone reading on a phone who won't follow the link.

---

*Sources: [[raw/deciphering-basmala]]*
*Last updated: 2026-07-03*
