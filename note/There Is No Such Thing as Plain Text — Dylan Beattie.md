# There Is No Such Thing as Plain Text — Dylan Beattie

Dylan Beattie's conference keynote (preserved as a ytx gist digest plus full transcript; the closing applause transcribed as "Kraft" suggests Craft, undated) argues that "plain text" is a 1960s American teleprinter fossil and that every byte is somebody's opinion. Where [[Character Sets and Unicode]] (Spolsky, 2003) made the same claim as a written admonition, Beattie stages it: ASCII's control characters, the code-page free-for-all, Unicode's refusal to answer "are these two strings the same?", database collations as cultural artifacts, a 27-hour incident where encoding failure impersonated a security breach, and a closing shibboleth — "do you know Pike Matchbox?"

---

## The Argument

What people mean by "plain text" is 7-bit ASCII that opens in Windows Notepad. ASCII was designed for mechanical teleprinters — "a computer was a typewriter with a memory" — and its quirks are fossilized constraints: Ctrl+C is literally the end-of-text control character (stop printing, something's gone wrong); the CRLF vs. LF split tracks whether your OS lineage had device drivers (Multics → Unix → Linux/macOS) or ran on cheap hardware without them (CP/M → MS-DOS → Windows); DEL is all-bits-set because you can't unpunch a paper tape.

The eighth bit was a regulatory vacuum: every country invented code pages to reinterpret the top half of ASCII — same bytes, different letters. Data isn't lost in transit; it's reinterpreted. Hence Billy Joel's live album permanently titled "Kohuept" in streaming databases, and a Harry Potter book delivered to a hand-copied, mangled Moscow address.

Unicode won by being free, universal, and opinion-free — but it deliberately refuses to answer the hard questions. It hands you tools (code points, combining characters, normalization forms, collations) and leaves the judgment calls to engineers who mostly don't know they're making them. Alphabetical order is cultural, not mathematical: a live show of hands couldn't agree which ordering of Berlin/Aachen/Zürich/Aarhus/Örebro was correct. The talk's thesis is that this isn't an accident — orthography is political, so text is political, from the 1948 Danish spelling reform to Apple removing the Taiwanese flag for mainland-China iPhones.

## Key Quotes

> "This is telling the database, think like an American."

On the `Latin1_General_CI_AI` collation, which treats Örebro's Ö as "just a letter O with decorations." This is the talk's sharpest move: a SQL collation isn't a technical setting, it's a declaration of whose alphabet the machine inherits. Spolsky's 2003 essay never got here.

> "There is not enough data in the text to get it right."

Why Danish orthography forces a two-column sort pattern — one column to display, a different one to sort. Aarhus sorts last (Å is a separate letter) while Aachen sorts first (AA is just two A's), and nothing in the bytes tells you which rule applies. Deterministic output requires metadata the text doesn't carry.

> "You can't not bring politics into it because politics creates the problems and technology tries to solve them."

The thesis, aimed at the "let's not bring politics into it" crowd. To understand why files sort wrong on Danish Windows you need the 1948 spelling reform and its 2011 reversion — "because Google wasn't finding it and they thought it was bad for tourism."

> "I'm going to sit on my ass and I'm going to shitpost on Twitter, because that's management."

The incident story: a competent security engineer reports "we've been hacked — there's Chinese in the Windows event logs." The team launches a breach investigation; Beattie posts the mystery text publicly and gets an instant diagnosis: "a Unicode mapping error." The real cause was a faulty switch dropping one byte every three minutes, shunting UTF-16LE text sideways into the CJK block. Twenty-seven hours to diagnose. Incident-response-by-broadcast, half-joking but literally what worked.

> "44% of this web page is just null, null, null, null... a tremendous waste of bandwidth."

A Ukrainian webpage sent in UTF-16: the HTML markup is ASCII, so every ASCII byte drags a null byte across the wire. The concrete motivation for UTF-8 — "one of the most brilliant hacks in the history of technology" — whose leading-bit rules make every ASCII document ever written valid UTF-8 unchanged.

> "It's not a word, it's a transcription error. That's the title of a record."

Billy Joel's *Kohuept: Live in Leningrad* — a Cyrillic title typed on an American keyboard that metastasized through record-company databases into streaming services. Data isn't lost in transit; it's reinterpreted, and then the reinterpretation is preserved forever.

> "I don't know if this is Microsoft saying that their official policy on Unicode is that gay pirates are winning, but it's certainly one way to read it."

Windows refuses country-flag emoji entirely — supporting only the pride flag, the skull and crossbones, and the Formula 1 checkered flag. Flags are political, so flag rendering is a diplomatic position, whether or not anyone meant it as one.

## Key Themes

#concept #history #internationalization #politics

## Critical Analysis

**This is Spolsky staged, twenty years later — and the staging earns its keep.** Spolsky's essay ends at "pick one encoding and declare it"; Beattie's talk is about what remains after you do. Equality and sort order are still judgment calls, because normalization and collation are policy, not math. The talk's genuine additions to the TANCST canon are the collation-as-culture material and the political framing made explicit rather than implied.

**The shibboleth is the pedagogical masterstroke.** "Do you know Pike Matchbox?" — the letters shared between Ukrainian Cyrillic and Latin — is a cheap, memorable test that encodes an entire worldview, and it sorts people the way Danish sorts cities. A spec tells you what to do; a shibboleth tells you who has been burned. For interviewers and reviewers, it's more useful than another checklist.

**The security hole is glaring, and the digest sees it.** An hour proving identical-looking text can be different bytes, and not one word on homoglyph attacks, IDN spoofing, or lookalike phishing. The omission section is right that security only ever appears as a false alarm here. The talk proves the weapon exists and never mentions it's been used.

**Folklore presented as fact is the talk's recurring cost.** CP/M did not evolve out of Minix (Minix postdates MS-DOS); emoji's creator is Shigetaka Kurita, not "Morita"; "every letter takes two bytes" in UTF-16 has been false since surrogate pairs — which the talk's own skin-tone examples require. For a talk about how text lies to you, it is carelessly trusting of its own anecdotes.

**The steelman is never engaged.** For pure 7-bit ASCII content, "plain text" genuinely does interoperate — that is precisely why UTF-8's backward compatibility was genius. The YouTube objectors are quoted only for ridicule, never answered. The honest version of the talk's claim is narrower and still interesting: plain text exists only inside an agreement about encoding, collation, and normalization, and the agreement is invisible until it breaks.

**The dedication is the mic drop that also caps the thesis.** "Leave politics out of software" is itself a political position — it asserts that the defaults someone grew up with are neutral. Beattie dedicating the talk to that comment is the whole argument compressed into one gesture.

## Related Pages

- [[Character Sets and Unicode]] — the direct ancestor: Spolsky's 2003 "There Ain't No Such Thing As Plain Text" states the thesis this talk theatricalizes; Beattie extends it with normalization forms, collations, and emoji machinery Spolsky never covered, and makes the politics explicit that Spolsky left implicit ("only the rest of the world has to jump through hoops").
- [[Arabic Typography]] — the same story from the rendering side: larrasket's essay traces scripts the Latin pipeline treats as exceptions, and its "Unicode fossil layer" pairs with Beattie's teleprinter fossils — both argue the technology encodes one culture's assumptions.
- [[Deciphering Basmala]] — two Unicode strategies for absorbing pre-digital writing traditions: Dominus's single codepoint U+FDFD versus Beattie's composition machinery (ZWJ, combining characters, regional indicators); both show the standard offering tools while the hard judgment stays with engineers.
- [[Hostnames and Usernames to Reserve]] — the missing link: that note flags absent IDN-homograph guidance, and Beattie proves lookalike text is trivially constructible without ever drawing the security consequence — the two omissions line up exactly.

---
*Sources: [[raw/there-is-no-such-thing-as-plain-text]], [[summary/there-is-no-such-thing-as-plain-text]]*
*Last updated: 2026-09-13*
