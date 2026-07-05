---
url: https://lr0.org/blog/p/arabic/
title: "An interactive introduction to the terrific experience of rendering Arabic typography and its technical debt"
author: larrasket (lr0.org)
date_fetched: 2026-07-03
date_published: 2026-06-10
---

# An interactive introduction to the terrific experience of rendering Arabic typography and its technical debt

Published on lr0.org by larrasket, June 10, 2026.

## Summary

A deep interactive essay tracing Arabic typography from Ibn Muqla's 10th-century proportional script system through the movable-type disasters of the Renaissance, the Unicode Presentation Forms fossil layer, the rise and fall of `text-justify: kashida`, and the volunteer-maintained open-source stack (HarfBuzz, Amiri) that actually renders Arabic on the modern web. The core argument: Arabic justification requires stretching letters, not spaces — a well-understood algorithm that no browser implements because "the users affected by it do not, as a population, contain any advertisers."

## Full Content

The article opens with a frontend ticket: Arabic text on a customer dashboard renders with a ragged left edge. Three related bugs had circled the same product: unjoined letters in a PDF library that predated shaping engine support, and a search index returning empty because a 2017 import used fossil Unicode codepoints from 1991 instead of regular ones from 1995.

### The Kashida Problem

CSS `text-align: justify` stretches inter-word spaces in Arabic, but true Arabic justification elongates letter strokes (kashida) — stretching inside words, never between them. The author hand-placed U+0640 TATWEEL characters in the mockup to show what the design team actually wanted.

### Scribal Tradition

- **Ibn Muqla** (d. ~940 CE): Abbasid vizier who systematized *al-khaṭṭ al-mansūb* (proportional script). Imprisoned twice, right hand amputated, tongue cut out, died in prison. His system of kashida and proportional letterforms built from rhombic dots "outlasted everybody who hurt him by a thousand years."
- **Ibn al-Bawwāb** (~1001 CE): Refined the system; single surviving Qurʾān in Chester Beatty Library, Dublin.
- **Yāqūt al-Mustaʿṣimī**: Survived the Mongol sack of Baghdad (1258) by climbing a minaret with his pens. Codified the Six Pens (Naskh, Thuluth, Muḥaqqaq, Rayḥān, Tawqīʿ, Riqāʿ).

### Structural Facts

- Arabic is cursive *always* — no print-vs-handwriting distinction
- Each letter has 4 positional forms (isolated, initial, medial, final); 6 letters refuse to connect forward
- Persian extends with 4 letters (پ, چ, ژ, گ); Urdu adds more including do-chashmī he (ھ)
- Architecture: Unicode stores abstract letters → fonts supply shapes → shaping engine applies OpenType features (`isol`, `init`, `medi`, `fina`, `rlig`, `mark`, `mkmk`)

### Interactive Demos

1. **Letter joining**: Click م, ح, م, د to build Muḥammad — each letter renegotiates its shape as the next arrives
2. **Fossil codepoints**: Search demo showing identical-looking names failing to match across modern Unicode vs. Presentation Forms (U+FB50–U+FEFF)
3. **Isolated-form disaster**: "مرحبا بالعالم" with every letter in isolated form and LTR layout — what old Photoshop, matplotlib, many npm PDF generators, and receipt printers produce
4. **Three digit sets**: Arabic-Indic (٠١٢٣٤٥٦٧٨٩), Western (0-9), Extended Arabic-Indic (۰۱۲۳۴۵۶۷۸۹) — toggle for an account balance
5. **Bidi phone number**: `010-1234-5678` renders backwards when preceded by Arabic — hyphens lose their European-digit context per UAX #9 rule W2
6. **Vowel clipping**: `line-height: 1; overflow: hidden` beheads fully vowelled text — "the vowels fall off the top" — a bug the author has fixed at three companies
7. **Caret schizophrenia**: At bidi run boundaries, the cursor has "two legitimate places" — Chrome picks differently from Firefox, differently from Qt, differently from Outlook
8. **Range flip**: `الصفحات 10-20` renders as "pages twenty to ten" because W2 strips European context from digits
9. **Required ligatures off**: lām-alif falls apart, the allāh ligature loses its stacked marks
10. **ZWNJ Fano**: U+200C ZWNJ between every pair forces isolated forms — "Fano, 1514, faithfully reproduced with one invisible Unicode character"

### The Unicode Fossil Layer

**Arabic Presentation Forms** (U+FB50–U+FEFF) encodes *shapes* instead of *letters* — hundreds of codepoints kept for round-trip compatibility with 8-bit code pages. U+FDFD ﷽ (bismillāh as a single codepoint) is called "a monument from the era when rendering was baked into the encoding."

### Five Centuries of Workarounds

| Year | Event |
|------|-------|
| 940 | Ibn Muqla finishes al-khaṭṭ al-mansūb |
| 1001 | Ibn al-Bawwāb's reference Naskh Qurʾān |
| 1258 | Yāqūt survives Mongol sack, refines Six Pens |
| 1485 | Bayezid II *reported* to ban Arabic printing (thin sourcing per Kathryn Schwartz) |
| 1514 | First Arabic movable-type book: *Kitāb Ṣalāt al-Sawāʿī*, Fano — "Every joint a small accident" |
| 1537 | Paganini Qurʾān, Venice — error-riddled, lost 450 years, rediscovered 1987 in Venetian friary |
| 1727 | İbrahim Müteferrika opens first Ottoman Muslim press in Istanbul |
| 1820 | Bulaq Press founded in Cairo — state-funded, hundreds of sorts per fount |
| 1924 | Cairo Qurʾān at Amiria Press — standardized text and typography for the 20th century |
| 1958 | Kamel Mrowa + Linotype create **Simplified Arabic** — "conquered the Arab newsroom in a generation" |
| 1984 | Sakhr MSX home computer ships Arabic in ROM |
| 1985 | DecoType founded (Thomas Milo, Mirjam Somers) |
| 1991 | Unicode 1.0: "Letters in, shapes out" + UAX #9 bidi algorithm |
| 1994 | InPage 1.0 ships from Karachi — Urdu newspaper calligraphers lose their profession in ~6 months |
| 1995 | Unicode adds Arabic Presentation Forms blocks |
| 1997 | OpenType 1.0 — `init`, `medi`, `fina`, `rlig`, `mark`, `mkmk`, `jstf` hooks available |
| 2000 | IE 5.5 implements `text-justify: kashida` — "the only software vendor on earth that could justify Arabic correctly on a screen" — value later deleted from spec |
| 2006 | DecoType's Tasmeem ships inside InDesign Middle East Edition |
| 2011 | Khaled Hosny releases Amiri font under OFL |
| 2012 | HarfBuzz lands in Chrome and Android; Unicode 6.1 adds Arabic Mathematical Alphabetic Symbols |
| 2015 | W3C Arabic Layout Requirements task force chartered; CSSWG opens Arabic justification issue — "both are still open in 2026" |
| 2022 | Amiri 1.0 |
| 2026 | "Won't Fix" |

### The jstf Standoff

The OpenType `jstf` table (since 1997) allows fonts to declare justification priorities, but "virtually no shaping engine reads it, so virtually no foundry ships it, so no engine acquires a reason to start." The structural reason: Latin justification places both break points and stretch points at inter-word spaces, but Arabic pulls those two sets apart — breaks between words, stretches inside them. Break points and elongation "have to be chosen together, against a cost function that depends on the actual glyphs."

Microsoft Word has shipped kashida justification since the late 90s. InDesign Middle East does it properly. But browser renderers cannot stretch a letter.

### Amiri

Khaled Hosny's one-man reconstruction of the Bulaq Press typeface — curvilinear kashida in four sizes, careful mark stacking. Since the 2022 rewrite, "if you are reading an Arabic text rendered well on the open web in 2026, there is a respectable chance you are reading Amiri."

### Credits

- **Khaled Hosny**: wrote Amiri, co-maintains HarfBuzz, maintains multiple free Arabic fonts, works on Arabic Unicode, filed dozens of CLDR bugs
- **Behdad Esfahbod**: "wrote much of HarfBuzz before Hosny," Iranian-Canadian, detained at US border in 2017 on suspicion of being Iranian. "The shaping engine running in your browser at this moment... was for years carried by an engineer the US government considered a security risk."
- **Brill**: spent ~$750,000 commissioning the Brill typeface for Semitic-studies transliteration, released it free in 2011
- **Sakhr BASIC**: on the 1984 Sakhr MSX, you could write `متغير = 5` and the interpreter parsed it — "Some of the children using it became, twenty years later, the engineers fighting bidi bugs in everyone else's software."

### Final Reflection

"Everything in this story that actually works was paid for by almost nobody" — HarfBuzz, Amiri, Scheherazade, GNU Unifont, Noto Arabic faces, W3C documents — all built by volunteers. "No commercial actor funded the unglamorous parts, because no quarterly report has a line item for 'Arabic users can now justify a paragraph.'" The remaining gap is "one well-understood algorithm in a handful of layout engines" that "somebody will close... probably unpaid."

## Further Reading

**Software:** Amiri font (amirifont.org), HarfBuzz (harfbuzz.github.io), DecoType/Tasmeem (decotype.com)

**Specifications:** UAX #9 (Unicode Bidirectional Algorithm), W3C Arabic Layout Requirements (alreq), OpenType specification

**History:** Kathryn A. Schwartz, "Did Ottoman Sultans Ban Print?" *Book History* vol. 20 (2017); Huda Smitshuijzen AbiFarès, *Arabic Typography: A Comprehensive Sourcebook* (2001); Geoffrey Roper, "Arabic Printing and Publishing"

**Manuscripts:** Ibn al-Bawwāb Qurʾān (1001 CE) — Chester Beatty Library, Dublin; 1924 Cairo Qurʾān — Amiria Press; Fano *Kitāb Ṣalāt al-Sawāʿī* (1514) — British Library and Vatican Library; Paganini Qurʾān (1537/8) — Franciscan convent of San Michele in Isola, Venice
