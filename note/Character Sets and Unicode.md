# Character Sets and Unicode

Joel Spolsky's 2003 field manual on the one thing every programmer must understand about text: characters are not bytes, "plain text" is a lie, and you cannot interpret a string without knowing its encoding. The article is the most-referenced plain-English introduction to Unicode ever written, and its core argument — that ignorance of encodings is professional negligence — has only grown sharper as software has become more global.

---

> "It does not make sense to have a string without knowing what encoding it uses. There Ain't No Such Thing As Plain Text."

The article's thesis statement, delivered in bold caps. Every string — in memory, in a file, in an email — carries an implicit or explicit encoding. If you don't know which one, you can't display it, search it, or even figure out where it ends. Nearly every "my website looks like gibberish" bug traces to a programmer who assumed 8 bits = one character = ASCII.

> "In Unicode, the letter A is a platonic ideal. It's just floating in heaven."

Spolsky's most useful conceptual move: separating the *code point* (the abstract letter, the U+ number) from its *encoding* (how you store that number in bytes). This distinction is what makes Unicode comprehensible. A → U+0041 → could be `41` (ASCII), `00 41` (big-endian UTF-16), `41 00` (little-endian UTF-16), or `41` (UTF-8). Same platonic A, four different byte sequences. Until you understand this, Unicode looks like unnecessary complexity rather than a clean abstraction with multiple concrete representations.

> "UTF-8 has the neat side effect that English text looks exactly the same in UTF-8 as it did in ASCII, so Americans don't even notice anything wrong. Only the rest of the world has to jump through hoops."

The political economy of encoding, stated with characteristic bluntness. UTF-8's backward compatibility with ASCII was a brilliant engineering decision that made adoption possible — Americans wouldn't have tolerated doubling their storage for no visible benefit. But it also means the encoding works hardest for non-English speakers, who pay the multi-byte penalty Americans never see. This pattern — English speakers getting the fast path, everyone else paying the tax — recurs throughout software internationalization.

> "Some people are under the misconception that Unicode is simply a 16-bit code where each character takes 16 bits and therefore there are 65,536 possible characters. This is not, actually, correct. It is the single most common myth about Unicode."

Written in 2003, when this misconception was ubiquitous (Java's `char` type and Windows' UCS-2 both baked it into APIs). Unicode had already exceeded 65,536 code points when Spolsky wrote this; the myth was already wrong. He doesn't belabor supplementary planes and surrogate pairs — this is the "absolute minimum," not the full specification — but he makes the key point: Unicode is not limited to 16 bits.

## Historical Narrative

The article's chronological structure is pedagogical genius. Spolsky doesn't start with Unicode — he starts with ASCII, then the OEM code page chaos, then ANSI code pages, then Asian DBCS, and only then arrives at Unicode as the necessary solution. By the time the reader reaches Unicode, they've lived through the problem it solves. This is much more effective than starting with "Unicode is a character set that..."

The OEM story is particularly vivid: code 130 was é on American PCs but the Hebrew letter Gimel on Israeli ones, so American résumés arrived in Israel as `rsums`. The Russian situation was even worse — multiple incompatible interpretations of the upper 128 characters meant Russian documents couldn't be reliably exchanged at all. This is what motivated Unicode: not theoretical elegance, but the practical impossibility of international text interchange.

## The "TANCST" (There Ain't No Such Thing As Plain Text) Rule

This is the article's lasting contribution to programmer vernacular. Spolsky argues that "plain text" is a category error — text without an encoding declaration is incomplete data. He applies this to email headers (`Content-Type: text/plain; charset="UTF-8"`), HTTP responses, and HTML `<meta>` tags. The edge cases are instructive:

- The `<meta charset>` tag must be the *very first thing* in `<head>` because browsers restart parsing when they encounter it — they've been guessing the encoding up to that point.
- If no encoding is declared anywhere, Internet Explorer tries to *guess* based on byte-frequency histograms of different languages. This works often enough to fool naïve developers into thinking they don't need to declare an encoding — until it doesn't.
- Spolsky explicitly rejects Postel's Law ("be conservative in what you emit, liberal in what you accept") for encoding: being liberal means guessing, and guessing means being wrong sometimes.

## Practical Prescription

Spolsky's advice is concrete and contextualized to the Windows ecosystem of 2003: use UCS-2 internally (matching Windows NT/2000/XP's native `wchar_t`), convert to UTF-8 when publishing to the web. The technical details have aged — the industry has largely converged on UTF-8 everywhere, and `wchar_t` is now understood to be a historical mistake — but the principle (pick one internal encoding, publish in UTF-8) holds.

## Key Themes

#concept #encoding #unicode #internationalization #history #fundamentals

## Critical Analysis

**The article's title is doing real work.** The bombastic length and parenthetical "No Excuses!" signal that this isn't an academic exercise — it's a professional obligation. Spolsky is declaring a minimum standard of competence. The onion-peeling-in-a-submarine threat is played for comedy, but the underlying anger is genuine: he had just discovered that PHP, one of the most popular web development tools of the era, had "almost complete ignorance of character encoding issues." This wasn't an obscure corner case; it was the mainstream.

**What aged well:** The conceptual framework — code points vs. encodings, the non-existence of plain text, UTF-8 as the universal answer — has proven durable. The article remains the best "first read" on the topic two decades later. The TANCST rule is arguably more important now than in 2003, as software handles more languages and more data sources.

**What aged less well:** The specific API advice is Windows-centric and pre-C++11. `wchar_t` and UCS-2 are now understood as dead ends — UTF-16 is a worst-of-both-worlds encoding (variable-width like UTF-8 but without ASCII compatibility) that survives mainly in Windows and Java for backward compatibility. The industry has largely settled on UTF-8 everywhere. Spolsky also doesn't cover normalization forms (NFC/NFD), grapheme clusters, or the distinction between code points and user-perceived characters — but he explicitly disclaims completeness, and those topics genuinely are beyond the "absolute minimum."

**The omission that matters most:** Spolsky doesn't discuss the security implications of encoding confusion. Overlong UTF-8 sequences, homograph attacks, and encoding-based filter bypasses are now standard attack vectors. This isn't a flaw in the article — it predates widespread awareness of these issues — but a modern reader needs to know that "getting encoding right" isn't just about avoiding gibberish; it's about not opening security holes.

**The article as a document of its era:** The Windows-centric perspective, the PHP complaint, the ActiveX and COM references — all locate this firmly in 2003. But the fundamentals are timeless. The article is a case study in how to write technical introductions that last: build the narrative chronologically, anchor abstractions in concrete problems, and pick one fact to be the takeaway ("There Ain't No Such Thing As Plain Text").

**Why it matters for the agentic era:** Coding agents that generate text-manipulating code — string operations, file I/O, web responses — need to understand encoding or they will produce broken software at scale. The article's lesson (always know your encoding) is the kind of fundamental that agents steeped in English-language training data are particularly likely to miss. A human developer who reads this learns to ask "what encoding?" whenever they see a string. An agent, unless explicitly trained or prompted to, will blithely assume ASCII.

---
*Sources: [[raw/the-absolute-minimum-every-software-developer-absolutely-positively-must-know-about-unicode-and-character-sets-no-excuses]], [[summary/the-absolute-minimum-every-software-developer-absolutely-positively-must-know-about-unicode-and-character-sets-no-excuses]]*
*Last updated: 2026-08-08*
